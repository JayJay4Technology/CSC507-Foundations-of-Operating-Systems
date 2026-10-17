# Foundations of Operating Systems

## Overview

An **operating system (OS)** is the software layer that manages a computer's
hardware and provides useful, consistent abstractions to applications and
people. It coordinates processors, memory, storage, and devices; controls how
programs execute; and enforces rules about sharing and protection. Examples
include Linux, Windows, macOS, Android, and the operating systems embedded in
routers, vehicles, and appliances.

The OS is more than a collection of utilities or a graphical interface. Its
central responsibilities are to:

1. **Abstract hardware.** Replace device-specific details with interfaces such
   as processes, files, virtual memory, and network sockets.
2. **Allocate resources.** Decide which program gets processor time, memory,
   storage, and access to shared devices.
3. **Protect and isolate.** Prevent programs from corrupting one another or
   the operating system, and control access to data and devices.
4. **Coordinate activity.** Manage concurrent work, communication, and
   synchronization across programs and processors.
5. **Provide common services.** Offer system calls and supporting tools so
   applications can perform useful work without directly controlling hardware.

These responsibilities are interconnected. For example, running two programs
at once requires processor scheduling, memory isolation, and safe mechanisms
for communicating and sharing files.

## A layered view of a computer system

A useful way to understand a computer is as a set of layers:

```text
People and user interfaces
          |
Applications and system utilities
          |
System-call interface and libraries
          |
Operating-system kernel
          |
Processor, memory, storage, and devices
```

The **kernel** is the privileged core of the operating system. It handles
operations that require controlled access to hardware, including scheduling,
address-space management, device coordination, and enforcing protection.
Operating systems also include system programs and services outside the kernel:
for example, shells, graphical environments, background services, libraries,
and administration tools.

The boundary between applications and the kernel is important. An application
normally runs in **user mode**, where hardware access is restricted. When it
needs a protected service, it requests one through a **system call**. The
processor switches to **kernel mode**, the kernel validates and carries out the
request, and control eventually returns to user mode. This controlled boundary
helps keep ordinary application errors from becoming unrestricted hardware
operations.

System calls are one kind of controlled transition into the kernel. A
**hardware interrupt** is an asynchronous notification, such as a device
completion, while an **exception** is raised by the current instruction, for
example because of an invalid operation or a page fault. Processors transfer
control to kernel-provided handlers for these events. The kernel handles the
event and either resumes the interrupted work, schedules other work, or reports
an error to the program. Exact terminology and mechanisms vary by processor
architecture.

## What happens when a computer starts?

At power-on, the processor begins executing firmware, commonly UEFI or BIOS.
The firmware initializes enough hardware to locate and start a bootloader. The
bootloader loads the operating-system kernel and may provide configuration or
startup choices. The kernel then initializes memory management, processors,
device support, and essential system services. Finally, it starts a first
user-space process and other services, after which a login screen, shell, or
application environment becomes available.

This sequence illustrates a recurring OS design principle: each stage
establishes the conditions needed by the next stage, moving from minimal
hardware-level initialization toward a fully functioning system.

## Processes and programs

A **program** is a passive set of instructions stored as a file. A **process**
is a running instance of a program, together with its execution state and the
resources the operating system associates with it. A process typically has:

- A **program counter**, identifying the next instruction to execute.
- **CPU register values** and a stack used for function calls and temporary
  data.
- A **virtual address space** containing code, data, heap, and stack regions.
- An identity and security credentials, such as a user ID and permissions.
- References to resources such as open files, sockets, and other processes.
- A **process control block (PCB)** or equivalent kernel data that records
  information needed to manage and resume it.

Processes commonly move among states such as **new**, **ready**, **running**,
**waiting/blocked**, and **terminated**. A process is ready when it could run
but is waiting for a processor. It is blocked when it cannot continue until
some event occurs, such as a disk read completing or a message arriving.

The operating system creates processes, schedules them, provides ways for them
to communicate, and reclaims their resources when they finish. Processes offer
stronger isolation than simply running multiple tasks in one shared program
space.

### Threads

A **thread** is an execution path within a process. Threads in the same process
share its address space and many of its resources, but each thread has its own
program counter, registers, and stack. Threads can make it convenient to
perform independent activities concurrently, such as handling user input
while doing background work.

Sharing an address space also creates risks: two threads may access the same
data at the same time. Correct multithreaded programs use synchronization to
coordinate such access. A process can have one thread or many, and operating
systems may schedule threads directly or schedule processes whose threads are
then managed by a runtime.

## CPU scheduling and concurrency

On a single processor core with one execution context, only one instruction
stream executes at a time, but the OS can switch rapidly among runnable tasks.
This creates the **illusion of parallel activity** and lets the system remain
responsive while some tasks wait for I/O. Some processors expose multiple
hardware execution contexts per core; on multicore processors, multiple
instruction streams can truly execute simultaneously.

The **scheduler** selects which ready task should run. Scheduling goals may
include:

- **Fairness:** give competing tasks a reasonable share of processor time.
- **Responsiveness:** keep interactive applications quick to react.
- **Throughput:** complete as much useful work as possible.
- **Low waiting and turnaround time:** reduce time spent queued and time from
  submission to completion.
- **Meeting deadlines:** satisfy timing constraints in real-time workloads.

No single policy is best for every goal. A simple first-come, first-served
policy is easy to understand but may make short tasks wait behind long ones.
Round-robin scheduling gives runnable tasks turns, often using a time slice.
Priority-based scheduling favors important work, but a low-priority task can
be delayed indefinitely unless the system addresses **starvation**. Real-time
systems may use deadline-oriented policies. Actual operating systems often
combine policies and adapt them to workload and platform.

A **context switch** occurs when the processor stops executing one thread and
starts or resumes another. The OS saves the outgoing execution state and
restores the incoming state. Context switches make sharing possible but have
overhead: time spent switching is time not spent executing application work.

### Race conditions and synchronization

When concurrent tasks access shared state, the outcome can depend on the
precise timing of their operations. This is a **race condition**. For example,
if two threads both read a shared counter, add one, and write the result, both
may read the same old value and one increment may be lost.

A **critical section** is code that accesses shared state and must be protected
from conflicting access. Common synchronization mechanisms include:

- **Mutexes (locks):** allow one thread at a time to enter a protected region.
- **Semaphores:** maintain a counter used to control access or signal events.
- **Condition variables:** let threads sleep until a condition becomes true.
- **Atomic operations:** perform a small read-modify-write operation
  indivisibly.
- **Read-write locks:** allow concurrent readers while excluding conflicting
  writers.

Synchronization must be designed carefully. A **deadlock** occurs when tasks
wait forever for resources held by one another. A **livelock** occurs when
tasks keep reacting to one another but make no useful progress. **Starvation**
occurs when a task is continually denied the resources or time it needs.
Consistent lock ordering, short critical sections, and appropriate scheduling
policies help reduce these problems.

Deadlock is possible when four conditions hold together: resources are
mutually exclusive, a task holds resources while waiting for others, resources
cannot be forcibly taken away, and a circular chain of tasks waits for the
next task's resource. An OS or application can **prevent** deadlock by
breaking at least one condition, **avoid** unsafe allocations when it has
enough information to do so, or **detect** deadlocks and recover, for example
by terminating a task or reclaiming a resource. Each approach has different
costs and assumptions; many general-purpose systems leave some deadlock
avoidance to application design.

## Memory management and virtual memory

Programs need memory for instructions, data, stacks, and dynamically allocated
objects. The OS must keep track of memory use and provide each process with
safe access to its own memory.

**Virtual memory** gives a process an address space that appears private and
continuous, even though its data may occupy separated locations in physical
memory. The processor's memory-management hardware, typically using page
tables, translates virtual addresses into physical addresses and checks access
permissions. This translation provides:

- **Isolation:** one process normally cannot read or overwrite another
  process's memory.
- **Relocation:** programs need not know their physical placement in RAM.
- **Protection:** pages can be marked readable, writable, executable, or
  inaccessible.
- **Sharing:** selected pages, such as shared libraries, can be mapped into
  multiple processes.
- **A larger logical address space:** inactive data can be backed by storage
  when physical memory is constrained.

Memory is commonly managed in fixed-size units called **pages** (virtual
memory) and **frames** (physical memory). If a process accesses a page that is
not currently resident in RAM, a **page fault** transfers control to the OS.
The OS determines whether the access is valid and, if appropriate, loads the
page from its backing store. A page fault is not necessarily an error; it is
also a normal mechanism for demand loading.

Address translation is on a program's memory-access path, so processors cache
recent page-table translations in a **translation lookaside buffer (TLB)**.
Under memory pressure, the OS may need to select a page to evict.
**Page-replacement policies** choose which page to remove from memory; their
goal is generally to avoid evicting pages that will soon be needed again.
Evicting a modified page may require writing it to backing storage. These
policies affect performance, and no policy can eliminate the cost of working
sets that collectively exceed available memory.

Moving pages to and from storage is much slower than accessing RAM. If the
system spends too much time paging instead of running programs, it may
**thrash**. Memory pressure can be reduced by using memory efficiently,
limiting unnecessary concurrent workloads, and choosing sound page-replacement
and resource-management policies.

## Storage, files, and file systems

Storage devices retain data beyond a program's execution. The OS provides a
**file system**: a way to name, organize, store, retrieve, and protect data.
A file is a named collection of data with associated metadata, which may
include its size, owner, timestamps, and permissions. Directories organize
files into a hierarchy.

The file-system interface hides many details of the underlying device. A
program can request operations such as opening, reading, writing, seeking, and
closing a file without knowing how the storage medium lays out its data.
Operating systems may also provide links, quotas, file locking, caching, and
journaling or other mechanisms to improve reliability after a crash.

File systems must balance performance, space efficiency, consistency, and
recoverability. For example, writes may be cached before they reach persistent
storage. A sudden power failure can therefore leave recent updates unwritten
or only partly applied. Applications that need stronger durability use
appropriate synchronization or commit operations, while file systems use
recovery techniques to restore a consistent state.

Below the file-system interface, storage devices are commonly accessed through
logical blocks. The OS coordinates requests, buffering, caching, and device
queues. On rotating disks, request order can affect seek time, so storage
schedulers may reorder work while balancing throughput and fairness. Solid
state drives have different access characteristics, but queueing, resource
contention, and durability still matter. Applications should not assume that
a successful write is durable unless they use the appropriate OS and
file-system synchronization interfaces.

## Input/output and device management

Keyboards, displays, disks, network adapters, sensors, and other devices differ
greatly. The OS provides common I/O interfaces while device-specific **drivers**
translate those requests into commands the hardware understands.

Devices can signal work completion or other events through **interrupts**.
Rather than continuously checking every device, the processor can do other
work and respond when an interrupt arrives. Some devices also support
**direct memory access (DMA)**, allowing data to move between a device and
memory with less processor involvement.

I/O can be **blocking**, where the requesting task waits until the operation
finishes, or **nonblocking/asynchronous**, where it can continue working and
learn about completion later. Buffering and caching can improve performance
and accommodate different speeds between producers and consumers. The OS must
coordinate these mechanisms while ensuring that device access remains
controlled and reliable.

## Protection and security

**Protection** concerns controlling how processes and users access system
resources. **Security** is the broader effort to defend the system and its
information against misuse, mistakes, and attacks. Operating systems use
multiple layers of defense, including:

- User and group identities, authentication, and permission checks.
- Privilege separation between applications and the kernel.
- Memory isolation and controlled inter-process communication.
- File and device access controls.
- Auditing, system updates, and secure configuration.
- Sandboxes, resource limits, and other restrictions on application behavior.

The **principle of least privilege** says that a component should receive only
the authority it needs to perform its task. This limits damage if the
component is buggy or compromised. Security is not a single feature: it
depends on correct kernel behavior, careful application design, secure
configuration, and timely maintenance.

## Inter-process communication

Processes often need to exchange data or coordinate their activity. Common
**inter-process communication (IPC)** mechanisms include:

- **Pipes:** byte streams often used between related processes or command-line
  programs.
- **Message queues:** structured messages sent and received through a managed
  queue.
- **Shared memory:** multiple processes map the same memory region, usually
  with synchronization to prevent races.
- **Signals or events:** notifications that a particular condition occurred.
- **Sockets:** communication endpoints that can connect processes on the same
  machine or across a network.

IPC choices involve tradeoffs. Shared memory can be fast for large amounts of
local data but requires careful synchronization. Message passing can make
ownership and boundaries clearer, though it may involve copying and
serialization.

## Networking

Modern operating systems include networking support that lets applications
communicate using standard protocols. The OS manages network interfaces,
routes packets, and provides abstractions such as sockets. Applications can
use these interfaces without directly controlling the network hardware.

Network communication introduces delays, failures, and security concerns that
do not arise in the same way with local memory. A remote service may be slow or
unavailable; messages can be delayed, reordered, or lost; and data may cross
untrusted networks. Applications should therefore handle errors, use
appropriate timeouts, validate received data, and protect sensitive
communication.

## Virtualization and containers

**Virtualization** allows one physical computer to host multiple isolated
virtual machines. A **hypervisor** allocates hardware resources to guest
operating systems, each of which behaves as if it had its own machine.
Virtualization is useful for server consolidation, testing, isolation, and
running software designed for another environment.

**Containers** isolate applications while generally sharing the host kernel.
They package an application with its dependencies and use OS mechanisms to
limit and separate processes, networking, and resource use. Compared with a
virtual machine, a container usually has less overhead, but it does not
provide a separate guest kernel in the same way. Both techniques depend on
careful resource allocation and isolation.

## Types of operating systems

Operating-system designs reflect the needs of their environments:

- **Batch systems** run groups of jobs with limited interactive input.
- **Time-sharing systems** let many users or tasks interact with a computer
  through rapid sharing of resources.
- **Desktop and mobile systems** emphasize interactive use, broad device
  support, and application isolation.
- **Server systems** emphasize throughput, availability, remote management,
  and support for many concurrent clients.
- **Embedded systems** operate within dedicated devices and often have
  restricted memory, power, or hardware.
- **Real-time systems** must respond within specified timing constraints.
  In a hard real-time system, missing a deadline can be a system failure;
  soft real-time systems tolerate occasional deadline misses with degraded
  quality.
- **Distributed operating systems and distributed environments** coordinate
  resources or services across networked computers. They must account for
  partial failures and communication delays. Not every distributed system is
  itself a single operating system; this category describes an important
  setting in which OS concepts are applied.

These categories overlap. A server can run virtual machines, a mobile device
can contain real-time components, and an embedded device can provide network
services.

## Kernel architectures

Operating systems organize kernel functionality in different ways:

- A **monolithic kernel** places many services, such as drivers and file
  systems, in kernel space. Direct communication can be efficient, but a bug
  in a privileged component can affect the whole system.
- A **microkernel** keeps a smaller set of mechanisms in kernel space and
  moves more services into separate processes. This can improve modularity and
  fault isolation, although communication between components may add overhead.
- A **hybrid kernel** combines ideas from both approaches, retaining some
  services in privileged space while structuring other parts as separate
  components.

These terms describe broad design choices, not a guarantee of performance,
security, or reliability. Implementation quality and the system's workload
matter as much as the architectural label.

## Important design tradeoffs

Operating-system design repeatedly balances competing goals:

- **Performance versus isolation:** tighter boundaries may add overhead but
  reduce the impact of failures.
- **Fairness versus priority:** equal treatment is not always appropriate for
  urgent or latency-sensitive work.
- **Caching versus freshness:** caching can reduce delays but introduces
  consistency questions.
- **Generality versus specialization:** common abstractions simplify software,
  while specialized interfaces can better serve particular hardware or
  workloads.
- **Availability versus maintenance:** updates improve security and
  correctness but may require restarts or service interruptions.
- **Simplicity versus features:** more features can be useful but increase the
  amount of code and behavior that must be understood and secured.

Good OS design makes these tradeoffs explicit and provides predictable,
well-documented behavior to applications and administrators.

## A practical way to study operating systems

When analyzing an OS feature, ask:

1. **What problem is being solved?** Identify the hardware limitation or
   application need.
2. **What abstraction is exposed?** Consider whether the application sees a
   process, file, thread, socket, or another interface.
3. **What policy is used?** Separate the mechanism that makes an action
   possible from the policy that decides when or for whom it happens.
4. **What state must be tracked?** Look for data structures such as process
   tables, page tables, queues, and file metadata.
5. **What are the failure cases?** Consider invalid requests, resource
   exhaustion, races, device failures, and crashes.
6. **What are the costs and tradeoffs?** Examine latency, throughput, memory
   use, fairness, complexity, and security.

This method connects theoretical topics to real systems. For example, to
understand file I/O, trace an application request through a library and system
call, into kernel file-system code, through a device driver, and finally to
storage hardware. Then consider caching, permissions, concurrent access, and
what happens if the operation fails.

## Core vocabulary

| Term | Meaning |
| --- | --- |
| Kernel | The privileged core responsible for key system-management tasks. |
| System call | A controlled request from a program for an OS service. |
| Process | A running program and the state and resources managed for it. |
| Thread | An execution path with its own CPU state within a process. |
| Context switch | Saving one task's execution state and restoring another's. |
| Scheduler | The OS component that selects runnable work for a processor. |
| Race condition | A timing-dependent result caused by concurrent access to state. |
| Deadlock | A state in which tasks wait indefinitely for resources held by one another. |
| Virtual memory | An address-translation and protection system for process memory. |
| Page fault | An event raised when a memory access needs OS handling. |
| File system | The OS organization and interface for persistent files and directories. |
| Device driver | Software that controls or communicates with a hardware device. |
| Interrupt | A signal that requests processor attention for an event. |
| IPC | Mechanisms for communication and coordination among processes. |
| Hypervisor | Software that creates and manages virtual machines. |

## Summary

Operating systems make computer hardware usable by presenting abstractions,
allocating scarce resources, coordinating concurrent work, and enforcing
protection. Processes and threads explain how work executes; scheduling
explains how processor time is shared; virtual memory explains how memory is
organized and isolated; file systems and device drivers connect applications
to persistent storage and peripherals; and security, networking, IPC, and
virtualization extend these foundations to complex, connected environments.

The unifying idea is **controlled sharing**: an OS lets many programs and
users make productive use of the same machine while striving to keep their
work efficient, predictable, and isolated.