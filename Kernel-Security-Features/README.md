Most of the time, nothing has to be setup here as it comes enabled by default. It is a good practice to check if the features are set and if not, set them.

**KASLR (Kernel Address Space Layout Randomization)**

KASLR randomizes the location of the kernel in memory to make it harder for attackers to predict addresses. It's enabled by default on most modern kernels. To check if it's enabled:

```
cat /proc/cmdline | grep kaslr
```

If you don't see "nokaslr", it's likely enabled. To explicitly enable it, add "kaslr" to your kernel command line in your bootloader config.

**SMAP (Supervisor Mode Access Prevention):**
SMAP prevents the kernel from accessing user-space memory directly, reducing the attack surface. To check if it's supported:

```
grep smap /proc/cpuinfo
```

If you see `smap` in the flags, your CPU supports it. It's usually enabled by default if supported.

**SMEP (Supervisor Mode Execution Prevention):**
SMEP prevents the kernel from executing code in user-space memory. To check if it's supported:

```
grep smep /proc/cpuinfo
```

If you see `smep` in the flags, your CPU supports it. It's usually enabled by default if supported.T o ensure SMAP and SMEP are enabled:

1. Make sure you're using a recent Linux kernel version.
2. Check that they're not disabled in your kernel command line (look for "nosmap" or "nosmep" in /proc/cmdline).
3. If using a custom kernel, ensure the CONFIG_X86_SMAP and CONFIG_X86_SMEP options are enabled in the kernel configuration.

### Harden the Kernel

You can enable kernel security features:

Edit `/etc/default/grub` :

```bash
nvim /etc/default/grub
```

Add the following to `GRUB_CMDLINE_LINUX_DEFAULT`:

```bash
slab_nomerge init_on_alloc=1 init_on_free=1  page_alloc.shuffle=1 pti=on vsyscall=none debugfs=off oops=panic  module.sig_enforce=1 lockdown=confidentiality mce=0 quiet loglevel=0
```

`slab_nomerge`: Disables merging of similar kernel memory allocations, reducing the risk of certain types of attacks.

`init_on_alloc=1`: Initializes memory to zero on allocation, preventing information leaks from reused memory.

`init_on_free=1`: Zeroes memory on free, further preventing information leaks.

`page_alloc.shuffle=1`: Randomizes page allocator freelists, making certain attacks harder.

`pti=on`: Enables Page Table Isolation, mitigating Meltdown-type attacks.

`vsyscall=none`: Disables vsyscall, an obsolete method that can be used in certain exploits.

`debugfs=off`: Disables debugfs, reducing attack surface by removing a potential information leak source.

`oops=panic`: Causes the kernel to panic on oops, preventing potential exploits from kernel crashes.

`module.sig_enforce=1`: Enforces module signature verification, preventing loading of unsigned kernel modules.

`lockdown=confidentiality`: Enables kernel lockdown mode, restricting certain operations that could compromise kernel integrity.

`mce=0`: Disables machine check exception reporting, which can potentially leak sensitive information.

`quiet loglevel=0`: Reduces kernel log output, potentially hiding sensitive information from attackers.
