---
layout: post
title:  "perf - the official Linux profiler"
date:   2021-02-19
last_modified_at: 2021-02-19
categories: [Performance]
tags: [Tracing]
---

## Introduction

perf, aka perf_events, is the official Linux profiler and included in the Linux kernel source under tools/perf. It can instrument CPU performance counters, tracepoints, kprobes, and uprobes. It is capable of system profiling. 

## What can perf do?

## Performance Monitoring Counter(PMC)

**Show system wide PMC statistics**

The following example shows PMC statistics for the entire system, for 5 seconds 
```bash
$ perf stat -a sleep 5

 Performance counter stats for 'system wide':

        399,489.78 msec cpu-clock                 #   79.290 CPUs utilized
            21,795      context-switches          #    0.055 K/sec
               208      cpu-migrations            #    0.001 K/sec
            10,071      page-faults               #    0.025 K/sec
    15,739,163,983      cycles                    #    0.039 GHz
    11,493,495,432      instructions              #    0.73  insn per cycle
     2,091,049,131      branches                  #    5.234 M/sec
        23,987,055      branch-misses             #    1.15% of all branches

       5.038337951 seconds time elapsed
```

**Show PMC statistics for the specified process**

The following example shows PMC statistics for the dd process

```bash
$ perf stat dd if=/dev/zero of=/dev/null count=10000000
10000000+0 records in
10000000+0 records out
5120000000 bytes (5.1 GB) copied, 24.273 s, 211 MB/s

 Performance counter stats for 'dd if=/dev/zero of=/dev/null count=10000000':

         24,215.88 msec task-clock                #    0.998 CPUs utilized
                29      context-switches          #    0.001 K/sec
                 9      cpu-migrations            #    0.000 K/sec
               216      page-faults               #    0.009 K/sec
    84,457,629,678      cycles                    #    3.488 GHz
    57,721,722,068      instructions              #    0.68  insn per cycle
    11,116,384,346      branches                  #  459.053 M/sec
       123,121,154      branch-misses             #    1.11% of all branches

      24.274369589 seconds time elapsed

       6.860780000 seconds user
      17.355968000 seconds sys
```

## perf events

perf events can be listed using *perf list*. I just include a few for examples of each kind of events. It includes hardware event, hardware cache event, software event, PMU event tracepoint event and SDT event.

```bash
$ perf list
List of pre-defined events (to be used in -e):

  cache-misses                                       [Hardware event]
  cache-references                                   [Hardware event]
  context-switches OR cs                             [Software event]
  page-faults OR faults                              [Software event]
  L1-dcache-load-misses                              [Hardware cache event]
  L1-dcache-loads                                    [Hardware cache event] 
  cache-misses OR cpu/cache-misses/                  [Kernel PMU event]
  cache-references OR cpu/cache-references/          [Kernel PMU event]
  cpu-cycles OR cpu/cpu-cycles/                      [Kernel PMU event]  
  block:block_plug                                   [Tracepoint event]
  block:block_rq_abort                               [Tracepoint event]
  block:block_rq_complete                            [Tracepoint event]
  block:block_rq_insert                              [Tracepoint event]
  block:block_rq_issue                               [Tracepoint event]
  irq:softirq_entry                                  [Tracepoint event]
  kmem:kfree                                         [Tracepoint event]
  kmem:kmalloc                                       [Tracepoint event]
  kmem:kmalloc_node                                  [Tracepoint event]
  kmem:kmem_cache_alloc                              [Tracepoint event]
  kmem:kmem_cache_alloc_node                         [Tracepoint event]
  kmem:kmem_cache_free                               [Tracepoint event]
  kmem:mm_page_alloc                                 [Tracepoint event]
  kmem:mm_page_alloc_extfrag                         [Tracepoint event]
  kmem:mm_page_alloc_zone_locked                     [Tracepoint event]
  kmem:mm_page_free                                  [Tracepoint event]
  kmem:mm_page_free_batched                          [Tracepoint event]
  kmem:mm_page_pcpu_drain                            [Tracepoint event]
  syscalls:sys_enter_write                           [Tracepoint event]
  syscalls:sys_enter_read                            [Tracepoint event]
  sdt_libc:memory_heap_free                          [SDT event]
  sdt_libc:memory_heap_less                          [SDT event]
  sdt_libc:memory_heap_more                          [SDT event]
  sdt_libc:memory_heap_new                           [SDT event]  
```

## PMC sample frequency

A default sample frequency is used when using perf record with PMCs. Thus, not all the event are recorded. This is desirable because some PMCs events can occur billions of time per second which causes significant overhead of recording.

The following two examples record the hardware cycle events and software page-faults events. The output shows the frequency sampling is enabled(freq=1) and the sample frequency is 4000.

```bash
perf record -vve cycles -a sleep 1
Using CPUID GenuineIntel-6-55-4
intel_pt default config: tsc,mtc,mtc_period=3,psb_period=3,pt,branch
------------------------------------------------------------
perf_event_attr:
  size                             112
  { sample_period, sample_freq }   4000
  sample_type                      IP|TID|TIME|CPU|PERIOD
  read_format                      ID
  disabled                         1
  inherit                          1
  mmap                             1
  comm                             1
  freq                             1
  task                             1
  sample_id_all                    1
  exclude_guest                    1
  mmap2                            1
  comm_exec                        1
------------------------------------------------------------
[...]
[ perf record: Captured and wrote 2.057 MB perf.data (14549 samples) ]

$ perf record -vve page-faults -a sleep 1
Using CPUID GenuineIntel-6-55-4
intel_pt default config: tsc,mtc,mtc_period=3,psb_period=3,pt,branch
------------------------------------------------------------
perf_event_attr:
  type                             1
  size                             112
  config                           0x2
  { sample_period, sample_freq }   4000
  sample_type                      IP|TID|TIME|CPU|PERIOD
  read_format                      ID
  disabled                         1
  inherit                          1
  mmap                             1
  comm                             1
  freq                             1
  task                             1
  sample_id_all                    1
  exclude_guest                    1
  mmap2                            1
  comm_exec                        1
------------------------------------------------------------
[...]
[ perf record: Captured and wrote 1.434 MB perf.data (615 samples) ]
```

The sample frequency can be modified using the -F option.

```bash
perf record -F 99 -vve cycles -a sleep 1
Using CPUID GenuineIntel-6-55-4
intel_pt default config: tsc,mtc,mtc_period=3,psb_period=3,pt,branch
------------------------------------------------------------
perf_event_attr:
  size                             112
  { sample_period, sample_freq }   99
  sample_type                      IP|TID|TIME|CPU|PERIOD
  read_format                      ID
  disabled                         1
  inherit                          1
  mmap                             1
  comm                             1
  freq                             1
  task                             1
  sample_id_all                    1
  exclude_guest                    1
  mmap2                            1
  comm_exec                        1
------------------------------------------------------------
```

In Linux, there is a limit for frequency rate and cpu utilization for perf. These limits can be changed with *sysctl* if needed.

```bash
$ sysctl -a | grep kernel.perf_event_max_sample_rate
kernel.perf_event_max_sample_rate = 16000

$ sysctl -a | grep kernel.perf_cpu_time_max_percent
kernel.perf_cpu_time_max_percent = 25
```

## CPU profiling

perf is often used for CPU profiling. 

In the following example, we have a fio workload running to create 10000 files and 100KB each. We can profile the workload while it's running.

We increase the open files parameter value from the default 1024 to 10240 so that we can create 10000 files.

```bash
$ ulimit -a | grep "open files"
open files                      (-n) 1024

$ ulimit -n 10240

$ ulimit -a | grep "open files"
open files                      (-n) 10240
```

We use the following fio job file to run the workload.

```bash
$ cat fio_job_file.ini
[job1]
ioengine=libaio
iodepth=8
rw=write
direct=1
nrfiles=10000
filesize=102400
blocksize=4096
dedupe_percentage=30
buffer_compress_percentage=50
buffer_pattern="0123456789"
numjobs=1
directory=/testdir

$ fio fio_job_file.ini
```

We start the perf profiling in a different terminal while the above fio workload is running.

```bash
$ perf record -F 99 -p `pidof fio` -a -g -- sleep 5
```

Once the profiling is done, a *perf.data* result file is generated for later analysis. We use *perf report --stdio* command to summarize the *perf.data* and generate a text report.

From the text report, we can have a better idea on how the fio works by understanding the system call function graph. This is extremly useful when we need identify the high latency of functions.

```bash
$ ls -la | grep perf.data
-rw-------.  1 root root   143460 Feb 25 02:25 perf.data

$ perf report --stdio > perf.report.stdio.out
# To display the perf.data header info, please use --header/--header-only options.
#
#
# Total Lost Samples: 0
#
# Samples: 335  of event 'cycles:ppp'
# Event count (approx.): 3215628508
#
# Children      Self  Command  Shared Object       Symbol
# ........  ........  .......  ..................  ............................................
#
    41.73%     0.00%  :198904  [kernel.kallsyms]   [k] system_call_fastpath
            |
            ---system_call_fastpath
               |
               |--30.99%--sys_io_submit
               |          |
               |          |--29.96%--do_io_submit
               |          |          |
               |          |          |--15.62%--vx_naio_write_v2
               |          |          |          vx_naio_write
               |          |          |          |
               |          |          |          |--9.73%--vx_naio_handoff
               |          |          |          |          |
               |          |          |          |           --8.57%--__wake_up
               |          |          |          |                     __wake_up_common_lock
               |          |          |          |                     __wake_up_common
               |          |          |          |                     vx_wq_wakeup_function
               |          |          |          |                     default_wake_function
               |          |          |          |                     try_to_wake_up
               |          |          |          |                     |
               |          |          |          |                      --0.87%--_raw_spin_lock_irqsave
               |          |          |          |
               |          |          |           --5.89%--vx_naio_checks.isra.5.constprop.6
               |          |          |                     |
               |          |          |                      --5.16%--vx_fel_io_allowed
               |          |          |                                |
               |          |          |                                |--2.86%--vx_irwunlock
               |          |          |                                |          vx_recsmp_rangeunlock
               |          |          |                                |          vx_rwsleep_rec_unlock
               |          |          |                                |
               |          |          |                                 --1.92%--vx_irwlock
               |          |          |                                           vx_irwlock2
               |          |          |                                           vx_recsmp_rangelock
               |          |          |                                           vx_rwsleep_rec_lock
               |          |          |                                           _raw_spin_lock_irqsave
               |          |          |
               |          |          |--7.94%--kmem_cache_alloc
               |          |          |          __slab_alloc
               |          |          |          ___slab_alloc
               |          |          |
               |          |          |--3.83%--rw_verify_area
               |          |          |          security_file_permission
               |          |          |          |
               |          |          |           --3.48%--selinux_file_permission
               |          |          |                     __inode_security_revalidate
               |          |          |
               |          |          |--1.04%--lookup_ioctx
               |          |          |
               |          |           --0.83%--fget
               |          |
               |           --0.86%--kmem_cache_alloc
               |
               |--5.52%--sys_io_getevents
               |          read_events
               |          |
               |          |--3.06%--schedule
               |          |          __schedule
               |          |          |
               |          |           --2.76%--deactivate_task
               |          |                     update_rq_clock.part.76
               |          |
               |          |--1.77%--aio_read_events
               |          |          |
               |          |           --0.58%--_cond_resched
               |          |
               |           --0.64%--prepare_to_wait
               |                     _raw_spin_lock_irqsave
               |
                --5.04%--sys_open
                          do_sys_open
                          |
                          |--4.12%--do_filp_open
                          |          path_openat
                          |          |
                          |          |--2.38%--link_path_walk
                          |          |          |
                          |          |           --0.66%--lookup_fast
                          |          |                     vx_drevalidate
                          |          |                     vx_nmspc_resolve
                          |          |                     _raw_spin_lock_irqsave
                          |          |
                          |          |--0.90%--get_empty_filp
                          |          |          |
                          |          |           --0.72%--kmem_cache_alloc
                          |          |
                          |           --0.81%--do_last
                          |                     |
                          |                      --0.56%--__audit_inode
                          |                                audit_copy_inode
                          |                                get_vfs_caps_from_disk
                          |                                vx_linux_getxattr
                          |                                vx_get_eatype
                          |
                           --0.92%--getname
                                     getname_flags
                                     kmem_cache_alloc
[...]
```

## Tracepoint events

In the previous profile, we saw a function kmem_cache_alloc. It's a pre-defined tracepoint. So, we can trace it directly to understand the call graph related.

```bash
$ perf list | grep kmem_cache_alloc
  kmem:kmem_cache_alloc                              [Tracepoint event]
```

```bash
$ perf record -F 99 -e kmem:kmem_cache_alloc -p `pidof fio` -a -g -- sleep 5 
```

We can check the target tracepoint call graph as below.

```
$ perf report --stdio
    14.61%    14.61%  (mmap_region+0x38c) call_site=ffffffff877fa39c ptr=0xffff9193252c5f38 bytes_req=216 bytes_alloc=216 gfp_flags=GFP_KERNEL|GFP_ZERO
            |
            ---__mmap64
               system_call_fastpath
               sys_mmap
               sys_mmap_pgoff
               vm_mmap_pgoff
               do_mmap
               mmap_region
               kmem_cache_alloc
[...]               
```
*perf script* can also be used to get a summary. It's useful to spot patterns overtime which might be lost in a huge report summary.

```bash
$ perf script

fio 264249 [020] 3116264.020748: kmem:kmem_cache_alloc: (getname_flags+0x4f) call_site=ffffffff8785c74f ptr=0xffff9234810d5000 bytes_req=4096 bytes_alloc=4096 gfp_flags=GFP
_KERNEL
	ffffffff8782660d kmem_cache_alloc+0xfd ([kernel.kallsyms])
	ffffffff8785c74f getname_flags+0x4f ([kernel.kallsyms])
	ffffffff8785d8c5 user_path_at_empty+0x45 ([kernel.kallsyms])
	ffffffff8785d951 user_path_at+0x11 ([kernel.kallsyms])
	ffffffff87850633 vfs_fstatat+0x63 ([kernel.kallsyms])
	ffffffff87850a51 SYSC_newlstat+0x31 ([kernel.kallsyms])
	ffffffff87850ebe sys_newlstat+0xe ([kernel.kallsyms])
	ffffffff87d8fede system_call_fastpath+0x25 ([kernel.kallsyms])
	    7fd8de3ea365 __lxstat64+0x15 (/usr/lib64/libc-2.17.so)
	            8000 [unknown] ([unknown])
[...]
```

The following are the reports without using "-g" option during profiling. It summarized the profile without the detail function graph.

```bash
$ perf script
    fio 276212 [047] 3116580.119986: kmem:kmem_cache_alloc: (get_empty_filp+0x5c) call_site=ffffffff8784d10c ptr=0xffff922b52f2ee00 bytes_req=256 bytes_alloc=256 g
    fio 276212 [047] 3116580.120138: kmem:kmem_cache_alloc: (sys_io_setup+0xaa) call_site=ffffffff878a182a ptr=0xffff918f24972f40 bytes_req=576 bytes_alloc=576 gfp
    fio 276212 [008] 3116580.120419: kmem:kmem_cache_alloc: (do_io_submit+0x194) call_site=ffffffff878a1da4 ptr=0xffff91224f606f00 bytes_req=192 bytes_alloc=192 gf
    fio 276212 [008] 3116580.120420: kmem:kmem_cache_alloc: (do_io_submit+0x194) call_site=ffffffff878a1da4 ptr=0xffff91224f606f00 bytes_req=192 bytes_alloc=192 gf
    fio 276212 [008] 3116580.120420: kmem:kmem_cache_alloc: (do_io_submit+0x194) call_site=ffffffff878a1da4 ptr=0xffff91224f606f00 bytes_req=192 bytes_alloc=192 gf
    fio 276212 [008] 3116580.120425: kmem:kmem_cache_alloc: (vx_alloc+0x152) call_site=ffffffffc13ec032 ptr=0xffff91b661b0bd10 bytes_req=88 bytes_alloc=88 gfp_flag
    fio 276212 [008] 3116580.120440: kmem:kmem_cache_alloc: (getname_flags+0x4f) call_site=ffffffff8785c74f ptr=0xffff919470bd5000 bytes_req=4096 bytes_alloc=4096
```

```bash
$ perf report --stdio
    12.93%  (vx_alloc+0x152) call_site=ffffffffc13ec032 ptr=0xffff90e61cd6c840 bytes_req=88 bytes_alloc=88 gfp_flags=GFP_NOFS|GFP_NOWARN|GFP_NORETRY
    12.50%  (do_io_submit+0x194) call_site=ffffffff878a1da4 ptr=0xffff91cbb8190300 bytes_req=192 bytes_alloc=192 gfp_flags=GFP_KERNEL|GFP_ZERO
    12.42%  (do_io_submit+0x194) call_site=ffffffff878a1da4 ptr=0xffff91e20e737980 bytes_req=192 bytes_alloc=192 gfp_flags=GFP_KERNEL|GFP_ZERO
    12.02%  (do_io_submit+0x194) call_site=ffffffff878a1da4 ptr=0xffff919379aea240 bytes_req=192 bytes_alloc=192 gfp_flags=GFP_KERNEL|GFP_ZERO
    11.69%  (vx_alloc+0x152) call_site=ffffffffc13ec032 ptr=0xffff922ef33f91b8 bytes_req=88 bytes_alloc=88 gfp_flags=GFP_NOFS|GFP_NOWARN|GFP_NORETRY
    10.60%  (do_io_submit+0x194) call_site=ffffffff878a1da4 ptr=0xffff9153dad8be00 bytes_req=192 bytes_alloc=192 gfp_flags=GFP_KERNEL|GFP_ZERO
     5.79%  (getname_flags+0x4f) call_site=ffffffff8785c74f ptr=0xffff90edba1c3000 bytes_req=4096 bytes_alloc=4096 gfp_flags=GFP_KERNEL
     5.74%  (vx_alloc+0x152) call_site=ffffffffc13ec032 ptr=0xffff922efbb0b580 bytes_req=88 bytes_alloc=88 gfp_flags=GFP_NOFS|GFP_NOWARN|GFP_NORETRY
     4.88%  (do_io_submit+0x194) call_site=ffffffff878a1da4 ptr=0xffff91224f606a80 bytes_req=192 bytes_alloc=192 gfp_flags=GFP_KERNEL|GFP_ZERO
```

## perf stat

*perf stat* subcommand can be used to count the target event during the specified seconds.

The following example shows the *kmem:kmem_cache_alloc* tracepoints fired 146,375 times during 5 seconds.

```bash
$ perf stat -e kmem:kmem_cache_alloc -p `pidof fio` -a -- sleep 5

 Performance counter stats for process id '9639':

           146,375      kmem:kmem_cache_alloc

       5.003282238 seconds time elapsed
```
## Dynamic tracing with probe events

Except for tracing the predefined perf events which are present in *perf list* command, *perf* is capable of creating more events dynamically.

**kprobes**

**uprobes**

## perf stat

## Resource

* [perf source code](https://elixir.bootlin.com/linux/latest/source/tools/perf)