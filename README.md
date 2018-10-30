#os-team20

## How to Build
$ ./build

to build the test code, type
$ arm-linux-gnueabi-gcc -I /include test.c -o test
$ sdb root on
$ sdb push test /root/test

in debug console
$ direct_set_debug.sh --sdb-set
$ /root/test

## High-Level Design & implementation
  1. register system call sched_getweight, sched_setweight
  2. init_task.h, kernel/kthread.c 을 통해 기본 스케줄려를 SCHED_WRR로 바꿈.
  3. wrr.c을 통해 scheduler를 구현하고 makefile에 등록함.
  4. use a test investigate implement.

## Investigate

## Improve the WRR scheduler

WRR scheduler에서는 worst case latency가 queue의 갯수에 따라 증가하게 된다. 
u_i = fraction wieght of each que such that sum of weight of all task.
u_i는 connection의 갯수가 증가할수록 작아지게 된다.by w_i = u_i*M.    w_i 는 각 큐의 무게합. WRR에서는 M이라는 Constant을 곱함으로 que가 증가할수록 u_i는 감소하고 w_i도 감소하게 된다. 
$ LATENCY_i = W(1-u_i)L_i/r 
이러한 단점을 LOW LATENCY WRR SCHEDULER로 극복할 수 있다. 이는 M과 같이 constant을 곱해서 weight을 구하는것이 아니라, connection의 갯수가 증가함에 따라 감소하는 변수를 두어서 이러한 문제를 해결한다. connection의 갯수가 증가함으로써 total latency을 증가시키는데 이는 N이 증가할수록 r이라는 변수가 더 급격하게 줄어들길떄문제 전체 weight의 합은 감소됨을 이용하는 것이다. r= r_0*(1-k(N/N_max)^2)^(1/2)와 같은 변수를 u_i에 곱해서 weight을 산출한다.

## Lessons Learned
* custom scheduler를 구현하는 방법을 알게되었다.
* test파일을 작성하면서 prime factorization naive tiral division method에 대해 알게되었다.


## How To Register System Call
### 1. Increment the number of System calls
in file: "arch/arm/include/asm/unistd.h"
``` c
#define __NR_syscalls  (N)
```
to
```c
#define __NR_syscalls  (N+4)
```
Total number of system calls must be a multiplication of 4.

### 2. assign system call number
in file: "arch/arm/include/uapi/asm/unistd.h"
add
```c
#define __NR_myfunc      (__NR_SYSCALL_BASE+ #) 
```

### 3. make asmlinkage function
in file: "include/linux/syscalls.h"
```c
asmlinkage int my_func()  // if no parameter then write 'void' 
```

### 4. add to system call table
in file: "arch/arm/kernel/calls.S"
```
call(sys_myfunc)
```

### 5. Revise Makefile
in file: "kernel/Makefile"
```
obj -y = ...  ptree.o
```

### etc.
in file: "kernel/myfunc.c"  
the name of function must be sys_myfunc()
