---
layout: post
title: "数据结构cpuinfo_mips分析"
date: 2012-12-20 14:10:01
comments: true
tags: [mips,cpu]
---
有关代码中的cpuinfo_mips数据结构和对该数据结构填充赋值的代码。

kernel3.5.4
<!--more-->

## cpuinfo_mips分析
## 该数据结构的定义
arch/mips/include/asm/cpu-info.h:41行
	
	struct cpuinfo_mips {
		unsigned int		udelay_val;
		unsigned int		asid_cache;
	
		/*
	 	* Capability and feature descriptor structure for MIPS CPU
	 	*/
		unsigned long		options;
		unsigned long		ases;
		unsigned int		processor_id;
		unsigned int		fpu_id;
		unsigned int		cputype;
		int			isa_level;
		int			tlbsize;
		struct cache_desc	icache;	/* Primary I-cache */
		struct cache_desc	dcache;	/* Primary D or combined I/D cache */
		struct cache_desc	scache;	/* Secondary cache */
		struct cache_desc	tcache;	/* Tertiary/split secondary cache */
		int			srsets;	/* Shadow register sets */
		int			core;	/* physical core number */
	#ifdef CONFIG_64BIT
		int			vmbits;	/* Virtual memory size in bits */
	#endif
	#if defined(CONFIG_MIPS_MT_SMP) || defined(CONFIG_MIPS_MT_SMTC)
		/*
	 	 * In the MIPS MT "SMTC" model, each TC is considered
	 	 * to be a "CPU" for the purposes of scheduling, but
	 	 * exception resources, ASID spaces, etc, are common
	 	 * to all TCs within the same VPE.
	 	 */
		int			vpe_id;  /* Virtual Processor number */
	#endif
	#ifdef CONFIG_MIPS_MT_SMTC
		int			tc_id;   /* Thread Context number */
	#endif
		void 			*data;	/* Additional data */
		unsigned int		watch_reg_count;   /* Number that exist */
		unsigned int		watch_reg_use_cnt; /* Usable by ptrace */
	#define NUM_WATCH_REGS 4
		u16			watch_reg_masks[NUM_WATCH_REGS];
		unsigned int		kscratch_mask; /* Usable KScratch mask. */
	} __attribute__((aligned(SMP_CACHE_BYTES)));



## 对于该数据结构的填充
arch/mips/kernel/setup.c:36，定义cpu_data
	
	struct cpuinfo_mips cpu_data[NR_CPUS] __read_mostly;
	EXPORT_SYMBOL(cpu_data);
	
填充这个数据结构的地点：

 * arch/mips/kernel/smp.c:121行，函数start_secondary(void)
 
函数注释：First C code run on the secondary CPUs after being started up by the master.
	
	cpu_data[cpu].udelay_val = loops_per_jiffy;

 * arch/mips/kernel/smtc.c:267行，函数smtc_configure_tlb(void)

函数注释：Configure shared TLB - VPC configuration bit must be set by caller
		
	/* MIPS32 limits TLB indices to 64 */
	if (tlbsiz > 64)
		tlbsiz = 64;
	cpu_data[0].tlbsize = current_cpu_data.tlbsize = tlbsiz;
	smtc_status |= SMTC_TLB_SHARED;
	local_flush_tlb_all();
 * 349行，函数smtc_tc_setup(int vpe, int tc, int cpu)

函数注释：Common setup before any secondaries are started，Make sure all CPUs are in a sensible state before we boot any of the secondaries.For MIPS MT "SMTC" operation, we set up all TCs, spread as evenly as possible across the available VPEs.
	
	/* In general, all TCs should have the same cpu_data indications. */
	memcpy(&cpu_data[cpu], &cpu_data[0], sizeof(struct cpuinfo_mips));
	/* For 34Kf, start with TC/CPU 0 as sole owner of single FPU context */
	if (cpu_data[0].cputype == CPU_34K || cpu_data[0].cputype == CPU_1004K)
		cpu_data[cpu].options &= ~MIPS_CPU_FPU;
	cpu_data[cpu].vpe_id = vpe;
	cpu_data[cpu].tc_id = tc;
	/* Multi-core SMTC hasn't been tested, but be prepared */
	cpu_data[cpu].core = (read_vpe_c0_ebase() >> 1) & 0xff;
		
 * 394行，函数smtc_prepare_cpus(int cpus)

		/* cpu_data index starts at zero */
		cpu = 0;
		cpu_data[cpu].vpe_id = 0;
		cpu_data[cpu].tc_id = 0;
		cpu_data[cpu].core = (read_c0_ebase() >> 1) & 0xff;
		cpu++;
