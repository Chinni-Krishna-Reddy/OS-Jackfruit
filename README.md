OS-Jackfruit — Lightweight Multi-Container Runtime
1. Team Information

Name: Rachit S Mane
SRN: PES1UG24AM214

Name: P.Chinni Krishna Reddy 
SRN: PES1UG24AM209

2. Build, Load, and Run Instructions
Build Project
make
Load Kernel Module
sudo insmod monitor.ko

Verify:

lsmod | grep monitor
ls -l /dev/container_monitor
Start Supervisor
sudo ./engine supervisor ./rootfs-base
Create Container Root Filesystems
cp -a rootfs-base rootfs-alpha
cp -a rootfs-base rootfs-beta
Start Containers
sudo ./engine start alpha ./rootfs-alpha "/bin/sh"

sudo ./engine start beta ./rootfs-beta "/bin/sh"
View Running Containers
sudo ./engine ps
View Container Logs
sudo ./engine logs logger1
Stop Containers
sudo ./engine stop alpha
sudo ./engine stop beta
Unload Module
sudo rmmod monitor
3. Demo with Screenshots
1. Multi-Container Supervision

Screenshot: screenshots/1-multi-container.png

Shows:

One supervisor process active
Containers alpha and beta running concurrently
Both tracked under same supervisor
2. Metadata Tracking

Screenshot: screenshots/2-metadata.png

Shows:

ID       PID   STATE   SOFT(MB) HARD(MB)

alpha    4990 exited   40       64
beta     5000 exited   40       64
hogtest  4249 killed   10       20

Demonstrates metadata maintained by supervisor.

3. Bounded-Buffer Logging

Screenshot: screenshots/3-logging.png

Shows:

line1
line2
line3

Container output captured via producer-consumer bounded logging pipeline.

4. CLI and IPC

Screenshot: screenshots/4-cli-ipc.png

Shows:

sudo ./engine start alpha ./rootfs-alpha "/bin/sh"

Supervisor receives CLI request through control IPC channel.

5. Soft-Limit Warning

Screenshot: screenshots/5-soft-limit.png

Shows:

[container_monitor] SOFT LIMIT container=hogtest

Kernel monitor warning generated.

6. Hard-Limit Enforcement

Screenshot: screenshots/6-hard-limit.png

Shows:

[container_monitor] HARD LIMIT container=hogtest

Container killed after hard limit exceeded.

7. Scheduling Experiment

Screenshot: screenshots/7-scheduling.png

Shows:

cpu_hi (nice -5)
cpu_lo (nice 15)

Observed:

cpu_hi received larger CPU share
cpu_lo received lower scheduling priority
8. Clean Teardown

Screenshot: screenshots/8-clean-teardown.png

Shows:

ps -ef | grep sleep

No zombie processes remain.

4. Engineering Analysis
Isolation Mechanisms

Process isolation achieved using:

PID namespace
UTS namespace
Mount namespace
chroot rootfs isolation

Containers share host kernel but maintain isolated process/filesystem views.

Supervisor and Process Lifecycle

Long-running supervisor:

manages all containers
tracks metadata
handles signals
reaps child processes
prevents zombies
IPC, Threads and Synchronization

Two IPC mechanisms used:

Pipes → container logging
CLI control channel → supervisor commands

Synchronization:

mutex for shared metadata
producer-consumer bounded buffer
condition variables for buffer full/empty coordination
Memory Management and Enforcement

Kernel module tracks RSS.

Soft limit:

warning only

Hard limit:

process killed

Kernel-space enforcement ensures reliable policy application.

Scheduling Behavior

Different nice values caused:

higher CPU allocation for cpu_hi
reduced CPU share for cpu_lo

Demonstrates Linux fairness and priority scheduling.

5. Design Decisions and Tradeoffs
Subsystem	Design Choice	Tradeoff	Justification
Isolation	chroot	simpler than pivot_root	easier implementation
Supervisor	single parent process	central bottleneck	simpler management
Logging	bounded buffer	synchronization complexity	prevents lost logs
Kernel monitor	LKM enforcement	added kernel complexity	proper memory control
Scheduling	nice-based priorities	limited experiment scope	demonstrates scheduler behavior
6. Scheduler Experiment Results
Container	Nice	CPU Share
cpu_hi	-5	Higher
cpu_lo	15	Lower

Result:

Higher priority container received more CPU time, validating Linux scheduler behavior.