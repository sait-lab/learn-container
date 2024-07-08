## Container Internals

<img src="./container-internals.assets/container-a-box-for-one-app.webp" alt="container-a-box-for-one-app" style="zoom:50%;" />  



---



### Namespaces

Excerpt from https://blog.kubesimplify.com/understanding-how-containers-work-behind-the-scenes

>When you initialize a container, docker generates a set of namespaces for the container and every container has its own unique set of namespaces.
>
>**There are various types of namespaces with different properties:**
>
>- User/UIDs namespaces - a container running within the user namespace is isolated from the User IDs and Group IDs of other containers, making them unaware of each other's existence.
>- UTS namespaces - isolate hostname and domain name information.
>- IPC namespaces - isolate IPC method.
>- Net namespaces - isolate network interfaces.
>- PID namespaces - isolate process IDs.
>- Mount namespaces - isolate mount points.

`unshare` runs program in new namespaces. Run `man unshare` to learn about `unshare` on Linux.

```
DESCRIPTION
       The unshare command creates new namespaces (as specified by the command-line options described below) and then
       executes the specified program. If program is not given, then "${SHELL}" is run (default: /bin/sh).

       By default, a new namespace persists only as long as it has member processes. A new namespace can be made
       persistent even when it has no member processes by bind mounting /proc/pid/ns/type files to a filesystem path. A
       namespace that has been made persistent in this way can subsequently be entered with nsenter(1) even after the
       program terminates (except PID namespaces where a permanently running init process is required). Once a persistent
       namespace is no longer needed, it can be unpersisted by using umount(8) to remove the bind mount. See the EXAMPLES
       section for more details.
```



#### Process ID (PID) namespace on Linux

Excerpt from https://man7.org/linux/man-pages/man7/pid_namespaces.7.html

> PID namespaces isolate the process ID number space, meaning that processes in different PID namespaces can have the same PID.
>
> PIDs in a new PID namespace start at 1, somewhat like a standalone system, and calls to fork(2), vfork(2), or clone(2) will produce processes with PIDs that are unique within the namespace.

![diag-pid-ns](./container-internals.assets/diag-pid-ns.webp) 

Credit: Nginx blogs

Run `lsns` on the Linux host to list information about all the currently accessible namespaces.

```shell
sudo lsns -t pid
```

If you don't have containers running, this command should list one namespace only.

```
        NS TYPE NPROCS PID USER COMMAND
4026531836 pid     251   1 root /sbin/init
```

Use `unshare` to create a new `PID` namespace:

```shell
sudo unshare --pid --fork --mount-proc /bin/bash
```

Check running process in the new namespace:

```shell
# This command runs in the new namespace
ps aux
```

Start a new SSH session and connect to the same host, run `lsns` on the Linux host to list information about all the currently accessible namespaces.

```shell
sudo lsns -t pid
```

If you don't have containers running, this command should list two namespaces.

```
4026531836 pid     254     1 root /sbin/init
4026532654 pid       1 28775 root /bin/bash
```

![namespace-1](./container-internals.assets/namespace-1.webp) 

Create two processes in the new namespace and observer process list:

```
# These commands run in the new namespace
sleep 3600 &
sleep 3601 &
ps aux
```

On the host, show processes with namespace id.

```shell
# These commands run outside of the created namespace
sudo ps -e -o pidns,pid,ppid,args | grep sleep
sudo lsns -t pid
```

![pidns-pid](./container-internals.assets/pidns-pid.webp) 



#### Observe Container and Namespaces

Start a container on the host and start an interactive terminal (enter the shell of the container):

```shell
# This command runs on the host
docker container run -it --rm alpine
```

Create two long running processes and show the processes inside the container.

```shell
# These commands run inside the container
sleep 3600 &
sleep 3600 &
ps aux
```

Find out the namespace id of the container on the host. Show the processes on the host.

```shell
# These commands run on the host
sudo lsns -t pid
sudo ps -e -o pidns,pid,ppid,args | grep NAMESPACE_ID
```

![container-pidns-pid](./container-internals.assets/container-pidns-pid.webp) 



#### Net namespaces on Linux

🚧



---



### Control groups (cgroups)

Excerpt from https://www.redhat.com/sysadmin/cgroups-part-one

>Cgroups are, therefore, a facility built into the kernel that allow the administrator to set resource utilization limits on any process on the system. In general, cgroups control:
>
>- The number of CPU shares per process.
>- The limits on memory per process.
>- Block Device I/O per process.
>- Which network packets are identified as the same type so that another application can enforce network traffic rules.

>cgroups are a mechanism for controlling certain subsystems in the kernel. These subsystems, such as devices, CPU, RAM, network access, and so on, are called *controllers* in the cgroup terminology.
>
>Each type of controller (`cpu`, `blkio`, `memory`, etc.) is subdivided into a tree-like structure. Each branch or leaf has its own weights or limits. A control group has multiple processes associated with it, making resource utilization granular and easy to fine-tune.
>
>***NOTE**: Each child inherits and is restricted by the limits set on the parent cgroup.*

![cgroup-diag](./container-internals.assets/cgroup-diag.webp) 

Credit: https://www.redhat.com/sysadmin/cgroups-part-one

More on `cgroups`: https://www.redhat.com/sysadmin/cgroups-part-two

With the help of `cgroups`, `runc` is able to share available hardware resources with the container and puts a limit on how much resources the container can use.


![diag-cgroup](./container-internals.assets/diag-cgroup.webp) 

Credit: Nginx blogs

####  `cpu` subsystem of  `cgroups` on Linux

Read more on `cpu` subsystem of  `cgroups`: [Subsystems and Tunable Parameters - cpu](https://docs.redhat.com/en/documentation/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/sec-cpu.html)

Add `cgroup_enable=cpu systemd.unified_cgroup_hierarchy=0` kernel command line options.

> [!NOTE]
>
> This demo runs on Ubuntu Linux 24.04 LTS.
>
> Ubuntu, Debian, RHEL/AlmaLinux/Rocky Linux use `GRUB2`. Arch Linux uses `systemd-boot`.
>
> Read more about kernel's command-line parameters at [The kernel's command-line parameters](https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html)
>
> Read more about `GRUB2` at [Working with GRUB 2](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-working_with_the_grub_2_boot_loader) 

Edit `/etc/default/grub`. Add `cgroup_enable=cpu systemd.unified_cgroup_hierarchy=0` to `GRUB_CMDLINE_LINUX`.

```shell
sudo vi /etc/default/grub
```

![add-kernel-cmd](./container-internals.assets/add-kernel-cmd.webp) 

Update the `GRUB` configuration and reboot the system for the changes to take effect.

```shell
sudo update-grub
sudo reboot
```

Install `cgroup-tools` on the host.

```shell
sudo apt install cgroup-tools
```

List available subsystems (resource controllers) for the `cgroups`. Each subsystem has a bunch of tunables to control the resource allocation.

```shell
lssubsys -am
```

```
cpuset
cpu
cpuacct
blkio
memory
devices
freezer
net_cls
perf_event
net_prio
hugetlb
pids
rdma
misc
```

Run `lscgroup` to list all `cgroups`

```shell
lscgroup
```

Focus on `cpu,cpuacct`

```
......
cpu,cpuacct:/
cpu,cpuacct:/sys-fs-fuse-connections.mount
cpu,cpuacct:/sys-kernel-config.mount
cpu,cpuacct:/sys-kernel-debug.mount
cpu,cpuacct:/dev-mqueue.mount
cpu,cpuacct:/user.slice
cpu,cpuacct:/sys-kernel-tracing.mount
cpu,cpuacct:/init.scope
cpu,cpuacct:/system.slice
......
```

Create a `cpulimited` `cgroup`.

```shell
sudo cgcreate -g cpu:/cpulimited
ls /sys/fs/cgroup/cpu/cpulimited/
```

![create-cpulimited](./container-internals.assets/create-cpulimited.webp) 

Install `stress-ng` on the host to stress the CPU (CPU hogs). More on `stress-ng`: https://wiki.ubuntu.com/Kernel/Reference/stress-ng.

```shell
sudo apt install stress-ng
```

Verify the `cpu.cfs_quota_us` value of `cpulimited` `cgroup`. Run the stress test without CPU limit.

```shell
sudo cgset -r cpu.cfs_quota_us=-1 cpulimited
cgget -r cpu.cfs_quota_us cpulimited
sudo cgexec -g cpu:cpulimited stress-ng --matrix 0 -t 1m
```

Don't stop the stress testing process. Start a new SSH session and connect to the same host, run `htop` to observe CPU utilization.

```shell
htop
```

![stress-no-limit](./container-internals.assets/stress-no-limit.webp) 

This VM has 4 CPUs and the CPU utilization is 400% for the 4 stress test processes when there is no CPU limit set.

Excerpt from https://docs.redhat.com/en/documentation/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/sec-cpu.html#sect-cfs

>specifies the total amount of time in microseconds (µs, represented here as "*`us`*") for which all tasks in a cgroup can run during one period (as defined by `cpu.cfs_period_us`). As soon as tasks in a cgroup use up all the time specified by the quota, they are throttled for the remainder of the time specified by the period and not allowed to run until the next period. If tasks in a cgroup should be able to access a single CPU for 0.2 seconds out of every 1 second, set `cpu.cfs_quota_us` to `200000` and `cpu.cfs_period_us` to `1000000`. Note that the quota and period parameters operate on a CPU basis. To allow a process to fully utilize two CPUs, for example, set `cpu.cfs_quota_us` to `200000` and `cpu.cfs_period_us` to `100000`.
>
>Setting the value in `cpu.cfs_quota_us` to `-1` indicates that the cgroup does not adhere to any CPU time restrictions. This is also the default value for every cgroup (except the root cgroup).

Set `cpu.cfs_quota_us=200000` to allow a process to fully utilize two CPUs. Run the stress test again.

```shell
sudo cgset -r cpu.cfs_quota_us=200000 cpulimited
cgget -r cpu.cfs_quota_us cpulimited
```

![stress-with-2cpu](./container-internals.assets/stress-with-2cpu.webp) 

Now the CPU utilization is limited to 200%.

Delete the `cpulimited` `cgroup`.

```shell
sudo cgdelete cpu,cpuacct:/cpulimited
```



#### Docker Resource Constraints

Docker (`runc`) uses `cgroups` to put a limit on how much resources a container can use.

https://docs.docker.com/config/containers/resource_constraints/

##### Use `--cpu-shares` 

> Set this flag to a value greater or less than the default of 1024 to increase or reduce the container's weight, and give it access to a greater or lesser proportion of the host machine's CPU cycles. This is only enforced when CPU cycles are constrained. When plenty of CPU cycles are available, all containers use as much CPU as they need. In that way, this is a soft limit.

Run two containers with `--cpus=256` and `--cpus=768`. The `--rm` flag tells the Docker Engine to clean up the container and remove the file system after the container exits.

```shell
# This command runs on the host
docker run --rm --cpu-shares=256 -it alpine
```

Install `stress-ng` inside the container to stress the CPU (CPU hogs). Use `stress-ng` to use all available CPUs for the first container.

```shell
# These commands run inside the first stress testing container
apk add stress-ng
stress-ng --matrix 0 -t 10m
```

Don't stop the stress test inside the container. Start a new SSH session and connect to the same host, start another container with `--cpu-shares=768`.

```shell
# This command runs on the host
docker run --rm --cpu-shares=768 -it alpine
```

Install and use `stress-ng` to use all available CPUs for the second container.

```shell
# These commands run inside the second stress testing container
apk add stress-ng
stress-ng --matrix 0 -t 10m
```

Don't stop the stress testing containers. Start a new SSH session and connect to the same host, use `htop` to observe CPU utilization.

```shell
# This command runs on the host
htop
```

![cpu-shares](./container-internals.assets/cpu-shares.webp) 

##### Use `--cpus=<value>`

`--cpus=<value>` specifies how much of the available CPU resources a container can use.

Run a container without resource constraints.

```shell
# This command runs on the host
docker run --rm -it alpine
```

Use `stress-ng` to use all available CPUs.

```shell
# These commands run inside the container
apk add stress-ng
stress-ng --matrix 0 -t 1m
```

Don't stop the stress test inside the container. Start a new SSH session and connect to the same host, run `htop` to observe CPU utilization.

```shell
# This command runs on the host
htop
```

![container-without-cpu-limit](./container-internals.assets/container-without-cpu-limit.webp) 

This experiment shows that by default, a container has no resource constraints and can use as much of a given resource as the host's kernel scheduler allows.

Exit the container. Run a container with resource constraints. `--cpus="0.25"` guarantees the container at most 25% of the CPU every second.

```shell
docker run --cpus="0.25" -it alpine
```

Use `stress-ng` to use all available CPUs.

```shell
# These commands run inside the container
apk add stress-ng
stress-ng --matrix 0 -t 1m
```

Don't stop the stress test inside the container. Start a new SSH session and connect to the same host, run `htop` to observe CPU utilization.

```shell
# This command runs on the host
htop
```

![container-with-cpu-limit](./container-internals.assets/container-with-cpu-limit.webp) 

