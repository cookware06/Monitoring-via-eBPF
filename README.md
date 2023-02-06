# Monitoring via eBPF

# Monitoring via eBPF (How modern EDR Solutions monitor every process execution)

Historically, Linux logging was entirely up to developers as what needs to logged and the formats of these logs. Logging was primarily done by using the **auditd** component.

Recently, the late kernal version 3.15 was came up with eBPF. Let's dig why it was happened.

Every Organizations needed to have mandatory logging in OS to see what's going in their OS via system call. Basically a kernel monitoring system, which monitor every system call such as user login, privilege escalation, scheduled tasks, system reboots, etc. Significant perfomance impact was noticed on strong monitoring strategy. Anything further than that such as File Access, Modification and Execution needed to mentioned using auditd.

Later it became possible to monitor every single system call by a process execution using auditd. But the output format was complex as below.,

![image](https://user-images.githubusercontent.com/123907103/216847609-1b478074-6cce-4a20-aa9b-302210b5438c.png)

Looks busy, isn't it! It was harder to find meaning info from these log format.

# How it's started

In the beginning tcpdump was available across linux systems which uses a very simple VM in kernal space, that could watch the packets coming through the network stack and filter! User space was much empty as tcpdump was using space in kernel which shown a great results in terms of perfomance.

Later they've updated it to modern times as in the form of BPF (Berkeley Packet Filter - Developer(s): Steven McCanne, Van Jacobson). Initial idea was to replace network filter with BPF which provides a raw interface to data link layers, permitting raw link-layer packets to be sent and received., but they've generatized it in a broader sense and further lead to **eBPF (extended Berkeley Packet Filter)**.

# What's eBFP

eBPF is a revolutionary technology with origins in the Linux kernel that can run sandboxed programs in an operating system kernel. It is used to safely and efficiently extend the capabilities of the kernel without requiring to change kernel source code or load kernel modules.

![image](https://user-images.githubusercontent.com/123907103/216848514-49066b4f-241e-4ce3-b9a3-3fa2ddd7738d.png)

Reference: https://ebpf.io/

As Hal pomeranz would say, it's a generic VM which is capable of filtering pretty much any system calls.

![image](https://user-images.githubusercontent.com/123907103/216849000-f3353270-5d98-4dd6-b62d-587ae2526475.png)

As it's more general, filtering language became so much more complex. It goes through multiple steps as shown above. Endleswis possibilities, and the innovation that eBPF is unlocked has only just begun.

**Let's use eBPF with BCC program**

Why BCC?

BCC is a toolkit for creating efficient kernel tracing and manipulation programs, and includes several useful tools and examples. It makes use of extended BPF (Berkeley Packet Filters), formally known as eBPF, a new feature that was first added to Linux 3.15.

![image](https://user-images.githubusercontent.com/123907103/216852382-2bb8cb32-5247-4c01-8bdb-a48d2409c599.png)

Most ways to do BCC is to wrap up in a python program as above shown. We can see that bcc compiler was set with multiple hooks like process execution with name and whether it succeeded or failed. Such a transparent way of process execution !

Let's go through some real world tools which makes use of BPF., (Thanks to brendan gregg for these repo's)

**1. execsnoop** - these can monitor each process execution which is written in python.
     Refer: https://github.com/brendangregg/perf-tools/blob/master/execsnoop
     
**2. opensnoop** - these looks for file opens.
     Refer: https://www.brendangregg.com/blog/2014-07-25/opensnoop-for-linux.html

**3. tcpaccept** - these monitor for incoming connections
     Refer: https://github.com/iovisor/bcc/blob/master/tools/tcpaccept.py

**4. tcpconnect** - these looks for outbound connections
     Refer: https://github.com/iovisor/bcc/blob/master/tools/tcpconnect.py

From a baseline monitoring perspective, these programs literally can monitoring every single process execution and much more.

Let's go with a user-friendly command-line tool  called "**bcftrace**" (Refer: https://github.com/iovisor/bpftrace) it's a high level tracing language for Linux eBPF.

**bcftrace -e 'BEGIN { printf("PID\tEXE\n"); } tracepoint: syscalls:sys_enter_execve { printf(%d\t%\n", pid, str (argv->filename));}**

Interpret as.,

1. printf("PID\tEXE\n"); - Output the process execution with PID and EXE Name.
2. tracepoint: syscalls:sys_enter_execve ( printf(%d\t%\n", pid, str (argv->filename));} - Hook on to exec call (sys_enter_execve) and output the PID and executable name.  Executable name is a pointer and we are converting to a string.

Let's execute a **ls** prgrogram and watch what the bcftrace program is doing,

![image](https://user-images.githubusercontent.com/123907103/216854300-6fa574b5-062b-41d5-a3d5-7d1cc2e5054d.png)

![image](https://user-images.githubusercontent.com/123907103/216854437-eb4fcbcd-d6a1-4e96-8501-50a3c4c45c8a.png)

We can see that the bcftrace is capturing the syscall with the hook.

Here we have used syscalls as "sys_enter_execve", but you can also try with other hooks under the syscall folder 
"/sys/kernel/debug/tracing/events/syscalls/" as below.,

![image](https://user-images.githubusercontent.com/123907103/216854706-90376833-4b0e-497c-81b4-453cbaef938c.png)

Also, the file under the /sys/kernel/debug/tracing/events/syscalls/[syscalls]/ has named "format" where you can see the available field names which can be used to define further while tracing a call.

Let's commercialize the this into modern EDR solution. Today's modern EDR solution are relying these basic monitoring procedure which makes them to watch each system call from a process execution.

# References: 

1. https://www.youtube.com/watch?v=8z9GEgnVFKg
2. https://ebpf.io/
3. https://github.com/brendangregg

**Big Thanks to Hal Pomeranz for great explanation with Antisyphon and Spyderbat**












