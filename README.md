# LearnRTThread
这是一个学习RTThread的项目

RT-Thread学习   
1、动态内存分配   
配置动态内存区域：rt_system_help_init()   
分配内存函数：rt_malloc，分配动态内存。   
建议清零操作rt_memset()   
释放：rt_free()   
2、线程创建   
函数静态线程创建：rt_err_t rt_thread_init()   
动态线程创建：rt_thread_t rt_thread_create()   
两者的区别，如果都在片内内存中，没有区别。动态创建可以使用外部内存，此时效率要低一些。   
3、系统滴答时钟频率通常在STM32中为100Hz   
GPIO驱动框架   
void rt_pin_mode设置引脚参数   
void rt_pin_write()设置输出状态   
void rt_pin_read()读取引脚状态。   
#include "rtdevice.h"static void led_entry(void *param){ rt_pin_mode(14,PIN_MODE_OUTPUT); while(1) { rt_pin_write(14,PIN_LOW);    rt_thread_delay(50);//rt_thread_mdelay(500);rt_thread_sleep(50); rt_pin_write(14,PIN_HIGH); rt_thread_delay(50); } }   
void led_test(void){ rt_thread_t tid; tid = rt_thread_create("led",led_entry,RT_NULL,512,10,10); if(tid != RT_NULL) { rt_thread_startup(tid); }}   
list_thread 可以列出所有线程，通常将栈最大使用量占总容量的70%比较合理。   
4,、rt_thread支持256级线程优先级，数字越小，优先级越高，0表示最高优先级，最低优先级给空闲线程。   
用户可以通过rt_config.h中的RT_THREAD_PRIORITY_MAX宏来修改最大支持的优先级。   
针对STM32通常设置为32个优先级。时间片只有在相同优先级的就绪态线程中起作用。   
5、空闲线程有钩子函数。任务调度有钩子函数   
6、临界区，每次只允许一个线程进入临界区，可以通过使用关闭系统调度(包括任务调度和中断)，还可以通过互斥特性保护临界区(信号量，互斥量)   
调度器上锁rt_enter_critical()   
调度器解锁rt_exit_critical();   
因为所有线程调度都是建立在中断的基础上的，所以关闭中断也将使得系统不在进行调度   
关闭中断rt_hw_interrupt_disable()   
打开中断rt_hw_interrupt_enable();   
7、信号量   
线程间通讯，包括同步，互斥，交换数据等，称之为IPC。   
RT-Thread的IPC机制包括信号量，互斥量，时间，邮箱，消息队列。   
信号量可以通过获取和释放达到同步或者互斥的目的。   
定义静态信号量：struct rt_semaphore static_sem   
  初始化和脱离rt_err_t rt_sem_init()   
                       rt_err_t rt_sem_detach()   
定义动态信号量：rt_sem_t dynamic_sem   
  创建和删除rt_sem_t rt_sem_create()   
                    rt_err_t rt_sem_delete()   
获取信号量 rt_err_t rt_sem_take()   
                   rt_err_t rt_sem_trytake()   
释放信号量 rt_err_t rt_sem_release()   
8、互斥量，相当于二值信号量   
定义静态互斥量：struct rt_mutex static_mutex   
  初始化和脱离rt_err_t rt_mutex_init()   
                       rt_err_t rt_mutex_detach()   
定义动态互斥量：rt_mutex_t dynamic_mutex   
  创建和删除rt_sem_t rt_mutex_create()   
                    rt_err_t rt_mutex_delete()   
获取互斥量 rt_err_t rt_mutex_take()   
释放互斥量 rt_err_t rt_mutex_release()   
9、事件集   
信号量用于一对一的线程同步，而一对多，多对一，多对多就需要事件集来处理了   
事件集是一个32bit无符号整形变量，每一个bit代表一个事件，通过逻辑与 ，逻辑或建立一个事件组合   
定义静态事件集：struct rt_enent static_evt   
  初始化和脱离rt_err_t rt_enent_init()   
                       rt_err_t rt_enent_detach()   
定义动态事件集：rt_enent_t dynamic_evt   
  创建和删除rt_enent_t rt_enent_create()   
                    rt_err_t rt_enent_delete()   
发送事件集 rt_err_t rt_enent_send()   
接收事件集 rt_err_t rt_enent_recv()   
10、邮箱   
用于线程间通讯。   
定义静态邮箱：struct rt_mailbox static_mb   
  初始化和脱离rt_err_t rt_mb_init()   
                       rt_err_t rt_mb_detach()   
定义动态邮箱：rt_mailbox_t dynnamic_mb   
  创建和删除rt_mailbox_t rt_mb_create()   
                    rt_err_t rt_mb_delete()    
发送邮箱 rt_err_t rt_mb_send()   
                rt_err_t rt_mb_send_wait()   
接收邮箱 rt_err_t rt_mb_recv()   
11、消息队列   
线程间通讯方式 之一，是对邮箱的扩展。消息队列能够发出不固定长度的消息，相比邮箱的固定四字节要灵活 的多。    
定义静态消息队列：struct rt_messagequeue static_mq   
  初始化和脱离rt_err_t rt_mq_init()   
                       rt_err_t rt_mq_detach()   
定义动态消息队列：rt_mq_t dynnamic_mq   
  创建和删除rt_mq_t rt_mq_create()   
                     rt_err_t rt_mq_delete()   
发送消息队列 rt_err_t rt_mq_send()链表尾部   
发送紧急消息 rt_err_t rt_mq_urgent()链表头部   
接收消息队列 rt_err_t rt_mq_recv()   
12、软件定时器   
基于滴答定时器，只能实现滴答定时器整数倍的定时。   
定义静态软件定时器 struct rt_timer static_timer   
  初始化和脱离void rt_timer_init()   
                       rt_err_t rt_timer_detach()   
定时动态软件定时器 rt_timer_t dynamic_timer   
  创建和删除rt_timer_t rt_timer_create()   
                     rt_err_t rt_timer_delete()   
启动定时器 rt_err_t rt_timer_start()   
停止定时器 rt_err_t rt_timer_stop()   
13、内存池   
内存池是一种内存分配方式，用于分配大量大小相同的小内存块，可以极大的加快内存分配与释放速度，尽可能避免内存碎片化，支持线程挂起。   
定义静态内存池 struct rt_mempool static_mp   
  初始化和脱离rt_err_t rt_mp_init()   
                       rt_err_t rt_mp_detach()   
定时动态内存池 rt_mp_t dynamic_mp   
  创建和删除rt_mp_t rt_mp_create()   
                     rt_err_t rt_mp_delete()   
申请内存池 void* rt_mp_alloc()   
释放内存池 void rt_mp_free()   
