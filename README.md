# linux-kernel-reverse

Steps to reverse engineer a custom Linux kernel (for dynamic and static analysis).
Mainly notes to myself.

## Getting the kernel version

The kernel version should appear in several strings (in [`linux_banner`](https://elixir.bootlin.com/linux/latest/A/ident/linux_banner) and [`vermagic`](https://elixir.bootlin.com/linux/latest/A/ident/vermagic)).

Searching for "Linux version" should do it.

## Extracting kallsyms

If the kernel is compiled with CONFIG_KALLSYMS, non-static symbols are contained in the Linux image.
Note that the symbol names are compressed (see [kallsyms.c](https://elixir.bootlin.com/linux/latest/source/kernel/kallsyms.c) ).

We can extract them in several ways:
  * read `/proc/kallsyms` on a live system;
  * parse the Linux image statically (we need to locate `kallsyms_num_syms`, `kallsyms_addresses`, `kallsyms_token_table`, `kallsyms_names`);
  * or use Unicorn/angr to emulate the function `kallsyms_on_each_symbol`).

Once we do that, we should get a text file with the address, type and name of each symbol.
