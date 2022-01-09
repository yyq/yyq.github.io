---
layout: post
title: "SMP结构在MIPS中的体现"
date: 2012-12-20 14:05:01
comments: true
tags: [cpu,mips,smp]
---
囧了哎，前面分析完x86的一堆代码，这会儿又得找到在内核源码中，有关mips架构和smp有关的代码

恩，一句话评论：mips的smp结构就是x86的smp的一种精简

依旧是kernel 3.5.4 ，主要在目录/arch/mips下
<!--more-->

# 和MIPS有关的SMP数据结构和函数

## xchg指令，CPU锁总线
**spinlock_t**数据结构

include/linux/spinlock_types.h, line 76
	
	struct {
		u8 __padding[LOCK_PADSIZE];
		struct lockdep_map dep_map;
	};
**spin_trylock()**和**spin_unlock()**之间是需要保证互斥，同一时间中，只有一个CPU在里面执行的临界区。

## 内存路障Memory Barrier
高速缓存的使用可能会使实际的内存操作改变次序。从访问内存的角度看，CPU可能会有一些路基操作已经完成，但是物理上尚未实现。“内存路障”就是使得某些操作之前，把之前的物理操作完成，才能开始新的。函数**mb(),rmb(),wmb()**
	arch/mips/include/asm/barrier.h, line 132
		
	#define wmb()           fast_wmb()
	#define rmb()           fast_rmb()
	#define mb()            fast_mb()


## 知道当前进程在哪个cpu上运行

**smp_processor_id()**,include/linux/smp.h, line 218
	
	# define smp_processor_id() debug_smp_processor_id()

**smp_processor_id()**,include/linux/smp.h, line 220

	# define smp_processor_id() raw_smp_processor_id()

**hard_smp_processor_id()**,arch/mips/include/asm/netlogic/mips-extns.h, line 71

	static inline int hard_smp_processor_id(void)
	{
        return __read_32bit_c0_register($15, 1) & 0x3ff;
	}
		

## 一CPU应系统中另一CPU的请求而进行一次进程调度

当一个CPU需要另一个CPU进行一次进程调度时，就可以通过调用一个函数smp_send_reschedule()向目标CPU发出一个RESCHEDULE_VECTOR中断请求。

**smp_send_reschedule()**,arch/mips/include/asm/smp.h, line 53

	static inline void smp_send_reschedule(int cpu)
	{
        extern struct plat_smp_ops *mp_ops;     /* private */ 
        mp_ops->send_ipi_single(cpu, SMP_RESCHEDULE_YOURSELF);
	}
	
## 处理器间某种中断向量：CALL_FUNCTION_VECTOR
还有一个处理器间中断向量是CALL_FUNCTION_VECTOR，用来请求目标CPU执行一个指定的函数。
(x86才找到这个，mips没找到)发送者先设置好一个全局的call_data_struct数据结构，然后向目标发出请求。

该向量的中断服务程序是**smp_call_function_interrupt()**

arch/mips/kernel/smp.c, line 146

	void __irq_entry smp_call_function_interrupt(void)
	{
        irq_enter();
        generic_smp_call_function_single_interrupt();
        generic_smp_call_function_interrupt();
        irq_exit();
	}

## SMP结构中的进程调度
cpu会通过**schedule()**从系统的就绪队列中挑选了一个进程作为运行的下一个进程。

kernel/sched/core.c, line 3462
	
	asmlinkage void __sched schedule(void)
	{
         struct task_struct *tsk = current; 
         sched_submit_work(tsk);
         __schedule();
	}
	EXPORT_SYMBOL(schedule);

__schedule()中，区分了smp和单CPU。



## SMP系统的引导
系统刚刚加电时或者reset后，系统中暂时只有一个处理器运行：“引导处理器”BP，其余暂停状态的是“应用处理器”AP。

BP完成自身初始化，进入保护模式，开启页式存储管理机制，完成内存的初始化，然后**start_kernel()**会调用**smp_init()**SMP结构初始化

init/main.c, line 322

	static void __init smp_init(void)
	{
		APIC_init_uniprocessor();	//其中内容很多
	}
	//2.6代码如下：
	smp_boot_cpus();
	smp_threads_read=1
	smp_commence()
在某处注释发现：**smp_boot_cpus()/smp_commence() is replaced by smp_prepare_cpus()/__cpu_up()/smp_cpus_done()**

依次启动系统中的各个CPU。各自初始化后都停下来等待统一的“起跑”命令。smp_commence（）

**smp_prepare_boot_cpu**，arch/mips/kernel/smp.c, line 189

	/* preload SMP state for boot cpu */
	void __devinit smp_prepare_boot_cpu(void)
	{
        set_cpu_possible(0, true);
        set_cpu_online(0, true);
        cpu_set(0, cpu_callin_map);
	}

**smp_prepare_cpus**，arch/mips/kernel/smp.c, line 178

	void __init smp_prepare_cpus(unsigned int max_cpus)
	{
        init_new_context(current, &init_mm);
        current_thread_info()->cpu = 0;
        mp_ops->prepare_cpus(max_cpus);
        set_cpu_sibling_map(0);
	#ifndef CONFIG_HOTPLUG_CPU
        init_cpu_present(cpu_possible_mask);
	#endif
	}
	
**__cpu_up**，arch/mips/kernel/smp.c, line 197

	int __cpuinit __cpu_up(unsigned int cpu, struct task_struct *tidle)
	{
        mp_ops->boot_secondary(cpu, tidle); 
        /*
         * Trust is futile.  We should really have timeouts ...
         */
        while (!cpu_isset(cpu, cpu_callin_map))
                udelay(100);

        return 0;
	}

**smp_cpus_done**，arch/mips/kernel/smp.c, line 171

	void __init smp_cpus_done(unsigned int max_cpus)
	{
        mp_ops->cpus_done();
        synchronise_count_master();
	}

内核中的全局变量，**max_cpus**，表示系统中有多少个CPU。也可以指定。

arch/mips/kernel/smp-bmips.c, line 39

	static int __maybe_unused max_cpus = 1;



















