# linux-kernel-reverse

Steps to reverse engineer a custom Linux kernel (for dynamic and static analysis, aka QEMU and Ghidra).
Mainly notes to myself.

## Putting the kernel in an ELF

The first step we may want to do is to convert the raw vmlinux to an ELF. 
Delimiting the sections with the permissions (.text, .data, .bss mainly) will allow us to load it directly into Ghidra.
Il will also serve as a container for the debug infos that we will add in the rest of the document.

Note that if the raw image is a bzImage, we must extract the vmlinux and wrap this file.

## Getting the kernel version

The kernel version should appear in several strings (in [`linux_banner`](https://elixir.bootlin.com/linux/latest/A/ident/linux_banner) and [`vermagic`](https://elixir.bootlin.com/linux/latest/A/ident/vermagic)).

Searching for "Linux version" should do it.

Once you have the exact version, you can retrieve it either:
  * by looking for the tarball of the official release on [kernel.org](https://mirrors.edge.kernel.org/pub/linux/kernel/). 
  * or by cloning the [linux-stable](git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git) repository.

## Extracting kallsyms

If the kernel is compiled with CONFIG_KALLSYMS, non-static symbols are contained in the Linux image.
Note that the symbol names are compressed (see [kallsyms.c](https://elixir.bootlin.com/linux/latest/source/kernel/kallsyms.c) ).

We can extract them in several ways:
  * read `/proc/kallsyms` on a live system;
  * parse the Linux image statically (we need to locate `kallsyms_num_syms`, `kallsyms_addresses`, `kallsyms_token_table`, `kallsyms_names`);
  * or use Unicorn/angr to emulate the function `kallsyms_on_each_symbol`).

Once we do that, we should get a text file with the address, type and name of each symbol.

## Adding symbols

We can then use a tool (?) to add the symbols to the ELF file.
This will be recognized by Ghidra and GDB.

## Compiling the kernel

For dynamic and static analysis, we need to know the structures used by the kernel.
We can get them from the vanilla kernel.
For this, we will compile the vanilla kernel, and use the DWARF information to recover the types.

However, the structures are highliy dependent on the kernel configuration.
So we must guess the kernel configuration as close as possible.
We can do this by trial and error.

We compile the vanilla kernel with an given configuration, and get a vmlinux.
To make sure we have a similar configuration, we can compare the kallsyms of both vmlinux.

Once we have a valid vmlinux, we can use Ghidra to extract the DWARF information, and add them to our proprietary vmlinux.
We can make sure that the offset of the structure members makes sense.

We can then add the DWARF information to our ELF file.

## Unwind info

TODO
