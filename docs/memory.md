# 内存

## 分段

虽然段内存基本已经被抛弃，不再使用了，但是 Linux 仍然使用了段机制来访问内存。
每一个CPU都有一个GDT（全局描述符表），存放在 `cpu_gdt_table` 数组中，如下：

```s
// arch/i386/kernel/head.S
/*
 * The Global Descriptor Table contains 28 quadwords, per-CPU.
 */
	.align PAGE_SIZE_asm
ENTRY(cpu_gdt_table)
	.quad 0x0000000000000000	/* NULL descriptor */
	.quad 0x0000000000000000	/* 0x0b reserved */
	.quad 0x0000000000000000	/* 0x13 reserved */
	.quad 0x0000000000000000	/* 0x1b reserved */
	.quad 0x0000000000000000	/* 0x20 unused */
	.quad 0x0000000000000000	/* 0x28 unused */
	.quad 0x0000000000000000	/* 0x33 TLS entry 1 */
	.quad 0x0000000000000000	/* 0x3b TLS entry 2 */
	.quad 0x0000000000000000	/* 0x43 TLS entry 3 */
	.quad 0x0000000000000000	/* 0x4b reserved */
	.quad 0x0000000000000000	/* 0x53 reserved */
	.quad 0x0000000000000000	/* 0x5b reserved */
```

`head.S`文件中有大量关于段的定义、访问、使用的代码，但很明显，它已经逐渐被遗弃了，在最新的RISC-V体系架构中，
压根就不会使用到分段，而是直接使用分页。

## 分页

在 `x86` 架构中，每个CPU都有一个独有的 `cr3` 寄存器用于保存页表地址，页表逐级指向下一层，并最终找到真正的物理内存：

```
cr3 => directory => table => page

31.   21    12    0
-------------------
|     |     |     |
-------------------
```

在一些情况下，可以省略中间一层，比如去掉 `table`，那么一页就是`4M`了，用于大内存页的场景。

在 64 位操作系统中，分页由3层变成4层，另外由于页表使用非常频繁，因此CPU为页表单独设立了缓存区，称为TLB。

Linux 中页表的初始化、操作、定义等均在 `arch/i386/mm` 文件夹下，另外 Linux 中的地址空间映射定义在
`System.map` 文件中，但源代码中是无法搜索到该文件的，因为它是编译生成而来的。比如页表初始化：

```c
// arch/i386/mm/init.c
static void __init pagetable_init (void)
{
	unsigned long vaddr;
	pgd_t *pgd_base = swapper_pg_dir;

#ifdef CONFIG_X86_PAE
	int i;
	/* Init entries of the first-level page table to the zero page */
	for (i = 0; i < PTRS_PER_PGD; i++)
		set_pgd(pgd_base + i, __pgd(__pa(empty_zero_page) | _PAGE_PRESENT));
#endif

	/* Enable PSE if available */
	if (cpu_has_pse) {
		set_in_cr4(X86_CR4_PSE);
	}

	/* Enable PGE if available */
	if (cpu_has_pge) {
		set_in_cr4(X86_CR4_PGE);
		__PAGE_KERNEL |= _PAGE_GLOBAL;
		__PAGE_KERNEL_EXEC |= _PAGE_GLOBAL;
	}
	// ....
}	
```

