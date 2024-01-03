---
title: Real time scheduling introduction
date: 2018-6-20
tags:
- Real time
- scheduling
- Priority

categories:
- concepts
description: "There are many computer controlled applications where delays in critical processing can have undesirable, or even disastrous consequences. A real-time system is one whose correctness depends on timing as well as functionality."
---
# Real time scheduling introduction

## Requirements
For the traditional scheduling Algorithms, we looked at *turn-around time (or throughput)*, *fairness*, *mean response time*.
But real-time systems have very different requirements, characterized by different metrics.
- timeliness ... how closely does it meet its timing requirements (e.g. ms/day of accumulated tardiness)
- predictability ... how much deviation is there in delivered timeliness

And in the real-time system, here are some other concepts:
- feasibility ... whether or not it is possible to meet the requirements for a particular task set
hard real-time ... there are strong requirements that specified tasks be run a specified intervals (or within a
- specified response time). Failure to meet this requirement (perhaps by as little as a fraction of a micro-second) may result in system failure.
- soft real-time ... we may want to provide very good (e.g. microseconds) response time, the only consequences of missing a deadline are degraded performance or recoverable failures.

It sounds like real-time scheduling is more critical and difficult than traditional time-sharing, and in many ways it is. But real-time systems may have a few characteristics that make scheduling easier:

We may actually know how long each task will take to run. This enables much more intelligent scheduling.
- Starvation (of low priority tasks) may be acceptable. The space shuttle absolutely must sense attitude and acceleration and adjust spolier positions once per millisecond. But it probably doesn't matter if we update the navigational display once per millisecond or once every ten seconds. Telemetry transmission is probably somewhere in-between. Understanding the relative criticality of each task gives us the freedom to intelligently shed less critical work in times of high demand.
- The work-load may be relatively fixed. Normally high utilization implies long queuing delays, as bursty traffic creates long lines. But if the incoming traffic rate is relatively constant, it is possible to simultaneously achieve high utilization and good response time.

## Real-Time Scheduling Algorithms

In the simplest real-time systems, where the tasks and their execution times are all known, there might not even be a scheduler. One task might simply call (or yield to) the next. This model makes a great deal of sense in a system where the tasks form a producer/consumer pipeline.

But for many real-time systems, the work-load changes from moment to moment, based on external events. These require dynamic scheduling. For dynamic scheduling algorithms, there are two key questions:

1. how they choose the next (ready) task to run ?
    shortest job first
    static priority ... highest priority ready task
    soonest start-time deadline first (ASAP)
    soonest completion-time deadline first (slack time)
2. how they handle overload (infeasible requirements) ?
    best effort
    periodicity adjustments ... run lower priority tasks less often.
    work shedding ... stop running lower priority tasks entirely.

*Preemption* may also be a different issue in real-time systems. In ordinary time-sharing, preemption is a means of improving mean response time by breaking up the execution of long-running, compute-intensive tasks. A second advantage of preemptive scheduling, particularly important in a general purpose timesharing system, is that it prevents a buggy (infinite loop) program from taking over the CPU. *The trade-off, between improved response time and increased overhead (for the added context switches), almost always favors preemptive scheduling.*

The truth is... (disadvantages)
- preempting a running task will almost surely cause it to miss its completion deadline.
- since we so often know what the expected execution time for a task will be, we can schedule accordingly and should have little need for preemption.
- embedded and real-time systems run fewer and simpler tasks than general purpose time systems, and the code is often much better tested ... so infinite loop bugs are extremely rare.

For the least demanding real time tasks, a sufficiently lightly loaded system might be reasonably successful in meeting its deadlines. However, this is achieved simply because the frequency at which the task is run happens to be high enough to meet its real time requirements, not because the scheduler is aware of such requirements. A lightly loaded machine running a traditional scheduler can often display a video to a user's satisfaction, not because the scheduler "knows" that a frame must be rendered by a certain deadline, but simply because the machine has enough cycles and a low enough work load to render the frame before the deadline has arrived.
