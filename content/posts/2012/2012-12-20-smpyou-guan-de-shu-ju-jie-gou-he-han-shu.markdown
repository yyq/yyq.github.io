---
layout: post
title: "SMP有关的数据结构和函数"
date: 2012-12-20 13:54:01
comments: true
tags: [cpu,smp,数据结构]
---
### 这些代码和函数都是在kernel3.5.4版本里面找到的，使用工具[understand](http://www.scitools.com/)和[lxr](http://lxr.linux.no/)

实验室的项目要求分析来着，上网找了一通资料，看到了这些有用的信息
<!--more-->
## 问题1：处理器之间的同步与互斥
解决SMP结构中对临界资源操作的原子性。有的指令，既要读又要写。“读-改-写”操作。汇编时，加入前缀LOCK，机器代码就使CPU在执行时，把阴险LOCK电位拉低，锁住总线。其他CPU就暂时不能通过总线访问内存了。

不用LOCK前缀也会锁住总线的特例：include/asm/spinlock.h中，spin_trylock()函数，就是使用schg指令（将一个内存单元中的内容与一个寄存器的内容对换）的实例。
该函数查看某个变量为1或0，如果spin_trylock()返回0，表示加锁失败，如果返回1，加锁成功。

### xchg指令时，CPU自动锁住总线
spinlock_t的结构定义在：/include/linux/spinlock_types.h
	
	struct {
		u8 __padding[LOCK_PADSIZE];
		struct lockdep_map dep_map;
	};
spin_trylock()和spin_unlock()之间是需要保证互斥，同一时间中，只有一个CPU在里面执行的临界区。

在竞争资源的时候，只有所有CPU在对同一个内存单元执行xchg指令才有意义，若分别对自己的高速缓存副本执行，而执行的结果不能立即被其他CPU看见，则spin_trylock()就毫无作用了。

### 信号量的实现
函数down()的代码（/arch/x86/include/asm/rwsem.h）中，指令前面就有前缀LOCK
### 内存路障
高速缓存的使用可能会使实际的内存操作改变次序。从访问内存的角度看，CPU可能会有一些路基操作已经完成，但是物理上尚未实现。“内存路障Memory Barrier”就是使得某些操作之前，把之前的物理操作完成，才能开始新的。函数mb(),rmb(),wmb()
### 知道当前进程在哪个cpu上运行
	smp_processor_id()
	hard_smp_processor_id();
定义地点：include/linux/smp.h, line 218：

	#define smp_processor_id() raw_smp_processor_id()
raw_smp_processor_id的定义地点：arch/x86/include/asm/smp.h, line 194：
	
	#define raw_smp_processor_id() (this_cpu_read(cpu_number))
this_cpu_read的定义地点：include/linux/percpu.h, line 291

	#define this_cpu_read(pcp) __pcpu_size_call_return(this_cpu_read_, (pcp))
__pcpu_size_call_return的定义地点：include/linux/percpu.h, line 175

	#define __pcpu_size_call_return(stem, variable)                         \
	 ({      typeof(variable) pscr_ret__;                                    \
	        __verify_pcpu_ptr(&(variable));                                 \
	         switch(sizeof(variable)) {                                      \	         case 1: pscr_ret__ = stem##1(variable);break;                   \
	         case 2: pscr_ret__ = stem##2(variable);break;                   \
	         case 4: pscr_ret__ = stem##4(variable);break;                   \
	         case 8: pscr_ret__ = stem##8(variable);break;                   \
	         default:                                                        \
	                 __bad_size_call_parameter();break;                      \
	         }                                                               \
	         pscr_ret__;                                                     \
	 })

在单cpu系统中，这两个函数都固定返回0。
每当一个CPU调度一个进行运行时，都把自己的逻辑序号设置在该进程的task_struct结构中的processor字段。这个序号，来自于原来在这个CPU上运行的进程。第一个CPU上运行的进程，这个序号是0，次CPU序号取决于主CPU启动它们运行的先后，从本地APIC中读取物理序号。



## 问题2：高速缓存与内存之间（内容的）一致性
各个CPU私有的，局部的高速缓存与公共的、全局的内存如何同步的问题

对于高速缓存的第一部分，只有数据才存在一致性问题，因为指令都是只读并且不会动态改变。每个CPU内部都有一部分专门的硬件，监视系统总线对内存的操作。如果有别人的写，自己也存着，则自动把相应的缓冲废弃。

高速缓存的TLB部分，是通过IPI即“处理器间中断”来解决。i386SMP结构中，采用“高级可编程中断控制器”APIC。
	
	send_IPI_mask()			//当一个CPU需要向其他CPU发出终端请求时，用这个函数。
	flush_tlb_others()		//当一个CPU要求其他CPU废弃各自TLB中的内容，调用
在/arch/x86/mm/tlb.c中的三个函数中，都有用到flush_tlb_others();
	
	void flush_tlb_current_task(void)
	{......
		flush_tlb_others(mm_cpumask(mm), mm, TLB_FLUSH_ALL);
	}
	void flush_tlb_mm(struct mm_struct *mm)
	{......
		flush_tlb_others(mm_cpumask(mm), mm, TLB_FLUSH_ALL);
	}
	void flush_tlb_page(struct vm_area_struct *vma, unsigned long va)
	{......
		flush_tlb_others(mm_cpumask(mm), mm, va);
	}
将flush_tlb_mm()与flush_tlb_page()作比较，基本相同，只有作为调用时的参数不一样。一个是FLASH_ALL,而不是具体的地址。	
	
（还没找到）在mm_struct数据结构中有个字段cpu_vm_mask，表示哪一些CPU正在使用这个空间。系统的每一个CPU都对应着这个位图中的一位。当一个cpu在进程调度中从老进程切换到新进程的时候，就要修改mm_struct这个数据结构中的位图。

全局变量cpu_online_map,也是个位图，记录着系统中所有的CPU，参数cpumask的内容只能是cpu_online_map的子集。flush_mm,flush_va,flush_cpumask都是全局变量，系统中的所有CPU都能看到这些变量，同时，对这些全局变量的改变必须互斥。通过send_IPI_mask()向有关CPU发送INVALIDATE_TLB_VECTOR中断请求。与该中断请求相对应的中断响应程序是
	
	smp_invalidate_interrupt()		//arch/x86/mm/tlb.c 中 132行
	{
		unsigned int cpu;
		unsigned int sender;
		union smp_flush_state *f;
		cpu = smp_processor_id();
		sender = ~regs->orig_ax - INVALIDATE_TLB_VECTOR_START;
		f = &flush_state[sender];
		if (!cpumask_test_cpu(cpu, to_cpumask(f->flush_cpumask)))
			goto out;
		if (f->flush_mm == this_cpu_read(cpu_tlbstate.active_mm)) {
			if (this_cpu_read(cpu_tlbstate.state) == TLBSTATE_OK) {
				if (f->flush_va == TLB_FLUSH_ALL)
					local_flush_tlb();
				else
					__flush_tlb_one(f->flush_va);
			} else
				leave_mm(cpu);
		}
	out:
		ack_APIC_irq();
		smp_mb__before_clear_bit();
		cpumask_clear_cpu(cpu, to_cpumask(f->flush_cpumask));
		smp_mb__after_clear_bit();
		inc_irq_stat(irq_tlb_count);
	}
内核中有个全局的tlb_state数据结构数组cpu_tlbstate[]，定义于arch/x86/mm/tlb.c
	
	DEFINE_PER_CPU_SHARED_ALIGNED(struct tlb_state, cpu_tlbstate)
		= { &init_mm, 0, };
tlb_state的定义在：/arch/x86/include/asm/tlbflush.h 147行
	
	struct tlb_state {
		struct mm_struct *active_mm;
		int state;
	};
数组cpu_tlbstate中所有的元素初值都是{&init_mm,0}。每个CPU在这个数组中都有一个tlb_state的tlb_state结构。在switch_mm()中，每当一个CPU切换到一个进程的虚拟空间时，就把这个结构中的指针active_mm设置成指向新的mm_struct结构，表示这个CPU正在使用这个虚存空间。



## 问题3：SMP结构中的中断机制
intel为pentium设计了APIC（Advanced Programable Interrupt Controllor）高级可编程中断控制器。在CPU芯片内部集成了一个本地APIC，SMP结构中还需要一个外部的全局的APIC，如图：
![pics](./jietu.png)
有几个为SMP结构专用的内核中断响应程序是 通过一些宏操作利用gcc预处理字符串替换和拼接功能自动生成的。宏操作定义在arch/x86/include/asm/hw-irq.h
举例：

* reschedule_interrupt()的实际的处理中断响应的函数为smp_reschedule_interrupt().
* invalidate_interrupt()的实际的处理中断响应的函数为smp_invalidate_interrupt()
* call_function_interrupt()的实际的处理中断响应的函数为smp_call_function_interrupt()

### 一CPU应系统中另一CPU的请求而进行一次进程调度

arch/x86/kernel/smp.c   249行
 
	void smp_reschedule_interrupt(struct pt_regs *regs)
	{
		ack_APIC_irq();
		inc_irq_stat(irq_resched_count);
		scheduler_ipi();
	/*
	 * KVM uses this interrupt to force a cpu out of guest mode
	 */
	} 
而ack_APIC_irq()定义在arch/x86/include/asm/apic.h  479行
	
	static inline void ack_APIC_irq(void)
	{
	/*
	 * ack_APIC_irq() actually gets compiled as a single instruction
	 * ... yummie.
	 */
		apic_eoi();
	}
	//*****442行内容如下*****
	static inline void apic_eoi(void)
	{
		apic->eoi_write(APIC_EOI, APIC_EOI_ACK);		// APIC_EOI_ACK是0x0
	}
smp_reschedule_interrupt()首先做的就是向CPU中的apic发出对中断请求的确认。

往本地寄存器中写一个0，表示收到了中断请求。可是，实际上对这种中断请求的服务隐藏在内核中对中断处理的公共部分。不管是什么中断请求，内核在针对特定中断请求的服务完成以后都要检查本CPU是否应该进行一次进程调度，而这正是smp_reschedule_interrupt()所要达到的目的。当一个CPU需要另一个CPU进行一次进程调度时，就可以通过调用一个函数smp_send_reschedule()向目标CPU发出一个RESCHEDULE_VECTOR中断请求。
定义在：/arch/x86/include/asm/smp.h   138行

	static inline void smp_send_reschedule(int cpu)
	{
		smp_ops.smp_send_reschedule(cpu);
	}

### CPU通过内部APIC向其他CPU发中断请求
除外部APIC可以把来自外部设备的中断请求提交系统中的各个CPU外，每个CPU也都可以通过其内部APIC向其他CPU发出中断请求。当一个CPU要引起其他CPU的INVALIDATE_TLB_VECTOR或者RESCHEDULE——VECTOR中断时，可以通过调用send_IPI_mask()来达到目的。
定义地点arch/parisc/kernel/smp.c  221行
	
	static void send_IPI_mask(const struct cpumask *mask, enum ipi_message_type op)
	{
		int cpu;
		for_each_cpu(cpu, mask)
			ipi_send(cpu, op);
	}

### CPU内部APIC的控制寄存器
CPU内部APIC有一些控制寄存器，APIC_ICR和APIC_ICR2是其中的两个。要向系统中的一个CPU发出中断请求是，首先要通过apic_wait_icr_idle(),确认或等待APIC_ICR处于空闲状态，然后通过__prepare_ICR()和__prepare_ICR2()，准备好要写入这两个寄存器的value

apci_wait_icr_idle()的定义：arch/x86/include/asm/apic.h   457行
	
	static inline void apic_wait_icr_idle(void)
	{
		apic->wait_icr_idle();
	}

 __prepare_ICR和__prepare_ICR2的定义：  arch/x86/include/asm/ipi.h  33行
	
	static inline unsigned int __prepare_ICR(unsigned int shortcut, int vector,unsigned int dest)
	{
		unsigned int icr = shortcut | dest;
		switch (vector) {
		default:
			icr |= APIC_DM_FIXED | vector;
			break;
		case NMI_VECTOR:
			icr |= APIC_DM_NMI;
			break;
		}
		return icr;
	}

	static inline int __prepare_ICR2(unsigned int mask)
	{
		return SET_APIC_DEST_FIELD(mask);
	}
寄存器APIC_ICR2主要用来说明发送中断请求的目标，然后把含有中断向量的数值写入APIC_ICR，就完成了中断请求的发送操作。

### 处理器间某种中断向量：CALL_FUNCTION_VECTOR
还有一个处理器间中断向量是CALL_FUNCTION_VECTOR，用来请求目标CPU执行一个指定的函数。发送者先设置好一个全局的call_data_struct数据结构，然后向目标发出请求。

call_data_struct的定义：arch/cris/arch-v32/kernel/smp.c  45行

	struct call_data_struct {
		void (*func) (void *info);
		void *info;
		int wait;
	};
数据结构中的函数指针func就是要求对方执行的函数，另一个指针info为参数。

该向量的中断服务程序是smp_call_function_interrupt(),定义在 arch/x86/kernel/smp.c 262行

	void smp_call_function_interrupt(struct pt_regs *regs)
	{
		ack_APIC_irq();
		irq_enter();
		generic_smp_call_function_interrupt();
		inc_irq_stat(irq_call_count);
		irq_exit();
	}
### 中断请求需要发送给除当前CPU外的所有CPU
通过send_IPI_allbutself(),广播式的中断请求。 定义在  arch/parisc/kernel/smp.c  229行

	static inline void send_IPI_allbutself(enum ipi_message_type op)
	{
		int i;	
		for_each_online_cpu(i) {
			if (i != smp_processor_id())
				send_IPI_single(i, op);
		}
	}

### 设置各个CPU的APIC中的时钟中断源
在系统的初始化阶段，主CPU在启动次CPU运行后，通过setup_APIC_clocks()设置APIC的时钟中断源。在3.5.4的源码中，这个函数没有找到.
start_kernel()>smp_init()>smp_boot_cpus()>setup_APIC_clocks()

所有CPUde APIC都通过APIC总线连接在一起，它们都有相同的时钟脉冲周期，是由setup_APIC_timer完成的。每个CPU只能设置自己的APIC，而不能直接设置其他CPU的APIC，所以通过smp_call_function向所有的次CPU都发出一个处理器间中断请求，让各个次CPU来执行这个函数。定义在arch/x86/kernel/apic/apic.c 523行

	static void __cpuinit setup_APIC_timer(void)
	{
		struct clock_event_device *levt = &__get_cpu_var(lapic_events);

		if (this_cpu_has(X86_FEATURE_ARAT)) {
			lapic_clockevent.features &= ~CLOCK_EVT_FEAT_C3STOP;
			/* Make LAPIC timer preferrable over percpu HPET */
			lapic_clockevent.rating = 150;
		}
		memcpy(levt, &lapic_clockevent, sizeof(*levt));
		levt->cpumask = cpumask_of(smp_processor_id());
		clockevents_register_device(levt);
	}
Setup the local APIC timer for this CPU. Copy the initialized values of the boot CPU and register the clock event in the framework.



## 问题4：SMP结构中的进程调度
cpu会通过schedule()从系统的就绪队列中挑选了一个进程作为运行的下一个进程。
schedule()的定义：kernel/sched/core.c:3462
	
	asmlinkage void __sched schedule(void)
	{
         struct task_struct *tsk = current; 
         sched_submit_work(tsk);
         __schedule();
	}
	EXPORT_SYMBOL(schedule);
__schedule()中，原有设计应该是区分了smp和单CPU，但是目前没发现区别。



## 问题5，SMP系统的引导
系统刚刚加电时或者reset后，系统中暂时只有一个处理器运行：“引导处理器”BP，其余暂停状态的是“应用处理器”AP。

BP完成自身初始化，进入保护模式，开启页式存储管理机制，完成内存的初始化，然后start_kernel()会调用smp_init()SMP结构初始化，代码定义在：init/main.c:322
### smp_init
	static void __init smp_init(void)
	{
		APIC_init_uniprocessor();	//其中内容很多
	}
	2.6代码如下：
	smp_boot_cpus();
	smp_threads_read=1
	smp_commence()
在某处注释发现：smp_boot_cpus()/smp_commence() is replaced by smp_prepare_cpus()/__cpu_up()/smp_cpus_done().

依次启动系统中的各个CPU。各自初始化后都停下来等待统一的“起跑”命令。smp_commence（）

### smp_prepare_cpus
arch/x86/kernel/smpboot.c:996

	void __init native_smp_prepare_cpus(unsigned int max_cpus)
	{
		unsigned int i;

		preempt_disable();
		smp_cpu_index_default();

		/*
	 	* Setup boot CPU information
	 	*/
		smp_store_cpu_info(0); /* Final full version of the data */
		cpumask_copy(cpu_callin_mask, cpumask_of(0));
		mb();

		current_thread_info()->cpu = 0;  /* needed? */
		for_each_possible_cpu(i) {
			zalloc_cpumask_var(&per_cpu(cpu_sibling_map, i), GFP_KERNEL);
			zalloc_cpumask_var(&per_cpu(cpu_core_map, i), GFP_KERNEL);
			zalloc_cpumask_var(&per_cpu(cpu_llc_shared_map, i), GFP_KERNEL);
		}
		set_cpu_sibling_map(0);

		if (smp_sanity_check(max_cpus) < 0) {
			printk(KERN_INFO "SMP disabled\n");
			disable_smp();
			goto out;
		}

		default_setup_apic_routing();

		preempt_disable();
		if (read_apic_id() != boot_cpu_physical_apicid) {
			panic("Boot APIC ID in local APIC unexpected (%d vs %d)",
		     	read_apic_id(), boot_cpu_physical_apicid);
			/* Or can we switch back to PIC here? */
		}
		preempt_enable();

		connect_bsp_APIC();

		/*
	 	* Switch from PIC to APIC mode.
	 	*/
		setup_local_APIC();

		/*
	 	* Enable IO APIC before setting up error vector
	 	*/
		if (!skip_ioapic_setup && nr_ioapics)
			enable_IO_APIC();

		bsp_end_local_APIC_setup();

		if (apic->setup_portio_remap)
			apic->setup_portio_remap();

		smpboot_setup_io_apic();
		/*
	 	* Set up local APIC timer on boot CPU.
	 	*/

		printk(KERN_INFO "CPU%d: ", 0);
		print_cpu_info(&cpu_data(0));
		x86_init.timers.setup_percpu_clockev();

		if (is_uv_system())
			uv_system_init();

		set_mtrr_aps_delayed_init();
	out:
		preempt_enable();
	}
	
内核中的全局变量，max_cpus，表示系统中有多少个CPU。也可以指定。
### do_boot_cpu
arch/x86/kernel/smpboot.c:667


### fork_by_hand
每个CPU在运动中必须有自己的上下文。forkbyhand，为目标CPU创建起第一个内核线程

































