# kfd

# kfd fork
This kfd fork merges the kern-info branch of the original kfd project together with white4ever's fork of kfd which implements vnodebypass [here](https://github.com/wh1te4ever/kfd). Do note that offsets for this in dynamic_info.h are only for iOS 16.6 beta 1, iPhone 14 Pro.

Issues: getProcByName panics the device (presumably due to an offset error)

kfd, short for kernel file descriptor, is a project to read and write kernel memory on Apple
devices. It leverages various vulnerabilities that can be exploited to obtain dangling PTEs, which
will be referred to as a PUAF primitive, short for "physical use-after-free". Then, it reallocates
certain kernel objects inside those physical pages and manipulates them directly from user space
through the dangling PTEs in order to achieve a KRKW primitive, short for "kernel read/write". The
exploit code is fully contained in a library, [libkfd](kfd/libkfd.h), but the project also contains
simple executable wrappers for [iOS](kfd/ContentView.swift) and [macOS](macos_kfd.c). The public API
of libkfd is quite small and intuitive:

```c
enum puaf_method {
    puaf_physpuppet,
    puaf_smith,
};

enum kread_method {
    kread_kqueue_workloop_ctl,
    kread_sem_open,
};

enum kwrite_method {
    kwrite_dup,
    kwrite_sem_open,
};

u64 kopen(u64 puaf_pages, u64 puaf_method, u64 kread_method, u64 kwrite_method);
void kread(u64 kfd, u64 kaddr, void* uaddr, u64 size);
void kwrite(u64 kfd, void* uaddr, u64 kaddr, u64 size);
void kclose(u64 kfd);
```

`kopen()` conceptually opens a "kernel file descriptor". It takes the following 4 arguments:

- `puaf_pages`: The target number of physical pages with dangling PTEs.
- `puaf_method`: The method used to obtain the PUAF primitive, with the following options:
    - `puaf_physpuppet`:
        - This method exploits [CVE-2023-23536][1].
        - Fixed in iOS 16.4 and macOS 13.3.
        - Reachable from the App Sandbox but not the WebContent sandbox.
    - `puaf_smith`:
        - This method exploits [CVE-2023-32434][2].
        - Fixed in iOS 16.5.1 and macOS 13.4.1.
        - Reachable from the WebContent sandbox and might have been actively exploited.
- `kread_method`: The method used to obtain the initial `kread()` primitive.
- `kwrite_method`: The method used to obtain the initial `kwrite()` primitive.


---
#kfd fork fork
## What are the supported OS versions and devices?

@Only 14P ios 16.6 beta1. I'll add more later.

---

##  How to build and run kfd on an iPhone?

In Xcode, open the root folder of the project and connect your iOS device.

- To build the project, select Product > Build (⌘B).
- To run the project, select Product > Run (⌘R), then click on the "kopen" button in the app.

---

## Where to find detailed write-ups for the exploits?

This README presented a high-level overview of the kfd project. Once a PUAF primitive has been
achieved, the rest of the exploit is generic. Therefore, I have hoisted the common part of the
exploits in a dedicated write-up:

- [Exploiting PUAFs](writeups/exploiting-puafs.md)

In addition, I have split the vulnerability-specific part of the exploits used to achieve the PUAF
primitive into distinct write-ups, listed below in chronological order of discovery:

- [PhysPuppet](writeups/physpuppet.md)
- [Smith](writeups/smith.md)

However, please note that these write-ups have been written for an audience that is already familiar
with the XNU virtual memory system.
