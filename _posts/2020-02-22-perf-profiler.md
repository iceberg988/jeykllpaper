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

## Tracepoint events

In terminal one, we kick off the dd workload to read from /dev/zero and write to a file with 5GB data.

```bash
$  dd if=/dev/zero of=testfile2 count=10000000
10000000+0 records in
10000000+0 records out
5120000000 bytes (5.1 GB) copied, 39.4875 s, 130 MB/s
```

In terminal two, while the dd workload is running, we start recording the events of all syscalls which match the pattern sys_enter_*.

```bash
$ perf record -e syscalls:sys_enter_* -p `pidof dd` -a sleep 5
Warning:
PID/TID switch overriding SYSTEM
[ perf record: Woken up 571 times to write data ]
Warning:
Processed 2162353 events and lost 12 chunks!

Check IO/CPU overload!

[ perf record: Captured and wrote 203.492 MB perf.data (2039794 samples) ]
```

After the perf recording, we can use *perf script* command to print the events.

```bash
$ perf script | more
dd  9851 [022] 3196252.710031:                  syscalls:sys_enter_read: fd: 0x00000000, buf: 0x01588000, count: 0x00000200
dd  9851 [022] 3196252.710033:                  syscalls:sys_enter_write: fd: 0x00000001, buf: 0x0158a000, count: 0x00000200
[...]
```

## Dynamic tracing with probe events

Except for tracing the predefined perf events which are present in *perf list* command, *perf* is capable of creating more events dynamically.

**kprobes**



**uprobes**



## Resource

* [perf source code](https://elixir.bootlin.com/linux/latest/source/tools/perf)