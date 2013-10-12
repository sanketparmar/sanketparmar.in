---
layout: post
title: Locking in Linux Kernel
last_changed: 2013-09-08
published: true
tags: kernel locking sip semaphore mutes
---

## Some definitions

### Critical Region
---
In parallel computing, A region of code that read or write shared writable data.
TODO: Add image

### Race Condition
---
Race conditions arise in software when an application depends on the sequence or
timing of processes or threads for it to operate properly. Due to race condition,
Final output may vary and can't predict. 


<div>
<div style="width:315px; float:left;">
{% capture m %}
#### *Table 1: _What we want_* 
---
&nbsp;Seq. No&nbsp; |&nbsp;&nbsp;&nbsp;&nbsp;Thread 1&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;Thread 2&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;Value of `a`&nbsp;&nbsp;|
:----:|:----------------:|:--------------------:|:---:|
1)  | read a     |            | 0 |
2)  | inc a by 1 |            | 0 |
3)  | write a    |            | 1 |
4)  |            | read a     | 1 |
5)  |            | inc a by 1 | 1 |
6)  |            | write a    | 2 |
{% endcapture %}
{{m | markdownify}}
</div>

<div style="width:315px; float:right;">
{% capture m %}

#### *Table2: _What we get_*
---
&nbsp;Seq. No&nbsp; |&nbsp;&nbsp;&nbsp;&nbsp;Thread 1&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;Thread 2&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;Value of `a`&nbsp;&nbsp;|
:----:|:----------------:|:--------------------:|:---:|
1)  | read a     |            | 0 |
2)  |            | read a     | 0 |
3)  |            | inc a by 1 | 0 |
4)  |            | write a    | 1 |
5)  | inc a by 1 |            | 0 |
6)  | write a    |            | 1 |
{% endcapture %}
{{m | markdownify}}
</div>
</div>

<br/>
In this example, two threads want to increment a global variable `a`. In 
Table 1. expected result is shown.


### Mutual Exclusion
---
If more  than one process can access data at   the same  time, as  is  the case
in preemptive multitasking systems and SMP systems, mutual exclusion must be 
introduced to protect this shared data.

### Synchronization
---
Synchronization is about making sure event happen at the same time (as in a 
common clock in communications systems) as opposed to asynchronous which means 
not happen at the same time.

Unfortunately, the term synchronization is often misused in the context of 
mutual exclusion. Synchronization is, by definition “To occur at the same time; 
be simultaneous”. Synchronization between tasks is where, typically, one task 
waits to be notified by another task before it can continue execution 
(unilateral rendezvous). A variant of this is either task may wait, called the 
bidirectional rendezvous. This is quite different to mutual exclusion, which is
a protection mechanism. However, this misuse has arisen as the counting semaphore
can be used for unidirectional synchronization.

**Mutual exclusion** is about making sure thing don't happen at the same time, 
whereas **synchronization** is about making sure they do.


<br/>
## Basic Linux Kernel Locking 

### Spinlock
---
The most common locking primitive in the kernel is the spinlock, defined in 
include/asm/spinlock.h and include/linux/spinlock.h. The spinlock is a very 
simple single-holder lock. If a process attempts to acquire a spinlock and it 
is unavailable, the process will keep trying (spinning) until it can acquire 
the lock. This simplicity creates a small and fast lock. 

#### _*Usage*_
{% highlight c++ %}
spinlock_t mr_lock = SPIN_LOCK_UNLOCKED;
unsigned long flags;
spin_lock_irqsave(&mr_lock, flags);
/* critical section ... */
spin_unlock_irqrestore(&mr_lock, flags);
{% endhighlight%}

For kernels compiled without `CONFIG_SMP`, and without `CONFIG_PREEMPT` spinlocks
do not exist at all. This is an excellent design decision: when no-one else can
run at the same time, there is no reason to have a lock.

If the kernel is compiled without `CONFIG_SMP`, but `CONFIG_PREEMPT` is set, then 
spinlocks simply disable preemption, which is sufficient to prevent any races.
For most purposes, we can think of preemption as equivalent to SMP, and not 
worry about it separately.[(source)](https://www.kernel.org/pub/linux/kernel/people/rusty/kernel-locking/x109.html)

#### _*Some important points about spinlock*_

* Single holder lock
* If you can't get the spinlock, you keep trying in loop (busy-wait)
* Very fast and can be used any way (in interrupt handler also)
* If the lock time is very high then don't use it (1-5ms)
* Can't Sleep
* It disables the preemption
* Avoid holding spinlock for more than 5 lines of code and across any function call (except accessors like readb).
* If the variable can also be modified by an interrupt routine, you must disable (at least that) interrupt while the spinlock is being
held (use spin_lock_irqsave and spin_unlock_irqrestore)

<br/>
### Semaphore
---
What spinlocks do is busy waiting(retry if flag is busy). It wastes CPU time.
In case of long locking time, It is costly operation. As a solution Linux kernel
provides semaphores.

If you need a lock that is safe to hold for longer periods of time, safe to 
sleep with or capable of allowing concurrency to do more than one process at a 
time then use semaphore.

Semaphores are manipulated via two methods: down (historically P) and up 
(historically V). The former attempts to acquire the semaphore and blocks if it
fails. The later releases the semaphore, waking up any tasks blocked along 
the way.

#### _*Usage:*_

{% highlight c++ %}
struct semaphore mr_sem;
sema_init(&mr_sem, 1);      /* usage count is 1 */
if (down_interruptible(&mr_sem))
  /* semaphore not acquired; received a signal ... */
  /* critical region (semaphore acquired) ... */
  up(&mr_sem);
{% endhighlight%}

####  _*Kernel implementation overview:*_

{% highlight c++ %}
down(mr_sem)
{
	spin_lock(mr_sem.lock)
	if (mr_sem.count > 0) 
		mr_sem.count--
	else {
		semaphore_waiter waiter
		waiter.task = current task
		waiter.up = false
		add waiter into mr_sem.wait_list
		while(true) {
			change current task status to TASK_UNINTERRUPTIBLE
			spin_unlock(mr_sem.lock)
			sleep using schedule_timeout(MAX_SCHEDULE_TIMEOUT)
			/* It deschedule the current process and put it appropriate queue */
			spin_lock(mr_sem.lock)
			if (waiter.up)
				break
		}
	}
	spin_unlock(mr_sem.lock)
}
{% endhighlight%}

{% highlight c++ %}
up(mr_sem)
{
	spin_lock(mr_sem.lock)
	if (mr_sem.wait_list is empty) 
		mr_sem.count++
	else {
		waiter = list_first_entry(mr_sem.wait_list)
		waiter.up = true
		change current status of waiter.task to TASK_NORMAL
		/* Puts the entry task into runnable queue */
	}
	spin_unlock(mr_sem.lock)
}
{% endhighlight%}


<br/>
#### _*Some important points about semaphore*_

* Sleeping lock
* The correct use of a semaphore is for signaling from one task to another
* Almost all practical applications use "count = 1"
* Example of counting semaphore is to implement "throttle"

{% highlight c++ %}
#define MAX_CONCURRENT_USER 20
sema_init(&mr_sem, MAX_CONCURRENT_USER);
/* Now 20 concurrent users can access the resource */
{% endhighlight %}

#### _*Problem with semaphore*_
"Accidental release" is most important drawback of semaphore. This problem 
arises mainly due to a bug fix, product enhancement or cut-and-paste mistake.
In this case, through a simple programming mistake, a semaphore isn’t correctly
acquired but is then released.

<div>
<div style="width:310px; float:left;">
{% capture m %}
{% highlight c++%}
/* Apps thread code */
 ...
 down(S);
 if (count > 0) --count;
 /* read data from buffer */
 up(S);
 ...


 {% endhighlight %}
{% endcapture %}
{{m | markdownify}}
</div>

<div style="width:310px; float:right;">
{% capture m %}
{% highlight c++%}
/* Another thread code */
 ...
 /* Opps forgot
 	down(S);
  */
 ++count;
 /* write data to buffer */
 up(S);
 ...
 {% endhighlight %}
{% endcapture %}
{{m | markdownify}}
</div>
</div>

When the counting semaphore is being used as a binary semaphore (initial count
of 1 – the most common case) this then allows two tasks into the critical
region. Each time the buggy code is executed the count is increment and yet
another task can enter. This is an inherent weakness of using the counting
semaphore as a binary semaphore.[(source)](http://blog.feabhas.com/2009/09/mutex-vs-semaphores-%E2%80%93-part-1-semaphores/)


### Mutex
---
The mutex is similar to the principles of the binary semaphore with one
significant difference: the principle of ownership. Ownership is the simple
concept that when a task locks (acquires) a mutex only it can unlock (release)
it. If a task tries to unlock a mutex it hasn’t locked (thus doesn’t own) then
an error condition is encountered and, most importantly, the mutex is not
unlocked. If the mutual exclusion object doesn’t have ownership then, 
irrelevant of what it is called, it is not a mutex.

They first appear in kernel 2.6.17. It has low memory footprint and fast as 
compared to semaphore.


#### _*Usage:*_ 
{% highlight c++ %}
DEFINE_MUTEX(mr_mutex)

/* Dynamically init */
mutex_init(&mr_mutex)
mutex_lock(&mr_mutex)
/* critical section ... */
mutex_unlock(&mr_mutex)
{% endhighlight%}


#### _*Kernel implementation overview:*_

<br/>
Implementation of mutex is architecture specific. Here is the overview.

{% highlight c++ %}
mutex_lock(mr_mutex)
{
	might_sleep();
	/* might_sleep will print a stack trace if it is executed in an atomic 
	   context. It is helpful in debugging */

	/* Kernel tries to follow the fast path(From mutex = 1 -> mutex = 0) */
	 __mutex_fastpath_lock(&mr_mutex->count, ...);
	 
	/* If fast path failed to aquire the lock if will follow slow path */
	__mutex_lock_slowpath(&mr_mutex->count, ...);

	/* in slow path */ 
	disable_preemption();
	/* add task into waiter.list 
	   wait for signal
	   lock_acquired
	   remove task from waiter.list
	   set owner for debugging purpose */
	preemption_enable();
}
{% endhighlight%}

{% highlight c++ %}
mutex_unlock(mr_mutex)
{
	/* Kernel tries to follow the fast path(From mutex = 0 -> mutex = 1) */
	 __mutex_fastpath_unlock(&mr_mutex->count, ...);
	
	/* If fast path failed to release the lock if will follow slow path */
	__mutex_unlock_slowpath(&mr_mutex->count, ...);

	/* in slow path */
	waiter = list_first_entry(mr_sem.wait_list);
	wake_up_process(waiter->task);
	/* change current status of waiter.task to TASK_NORMAL
	   puts the entry task into runnable queue */
}
{% endhighlight%}
<br/>
### Strict use case of mutex
- only one task can hold the mutex at a time
- only the owner can unlock the mutex
- multiple unlocks are not permitted
- recursive locking is not permitted
- a mutex object must be initialized via the API
- a mutex object must not be initialized via memset or copying
- task may not exit with mutex held
- memory areas where held locks reside must not be freed
- held mutexes must not be reinitialized
- mutexes may not be used in hardware or software interrupt contexts such as tasklets and timers
[(source)](https://www.kernel.org/doc/Documentation/mutex-design.txt)

To enforce the above rules, set `CONFIG_DEBUG_MUTEXES`. It will print stack trace if any of 
the above rules violates. 

<br/>
## Some important points
#### *_What to use when ?_*
 Requirement                         | Lock                   |
------------------------------------|------------------------|
Low overhead locking                | Spin lock is preferred |
Short lock hold time                | Spin lock is preferred |
Long lock hold time                 | Mutex is preferred     |
Need to lock from interrupt context | Spin lock is required  |
Need to sleep while holding lock    | Mutex is required      |


