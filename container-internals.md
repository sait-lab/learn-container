## Container Internals

<img src="./container-internals.assets/container-a-box-for-one-app.webp" alt="container-a-box-for-one-app" style="zoom:50%;" />  

 

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



#### Process ID (PID) namespace

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

```
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
sudo ps -e -o pidns,pid,ppid,args | grep sleep
sudo lsns -t pid
```

![pidns-pid](./container-internals.assets/pidns-pid.webp) 





### Control groups (cgroups)

>Cgroups help to limit the use of resources so that a single container is not utilizing all the resources available. It allows managing various system resources such as:
>
>- CPU - limit CPU utilization.
>- Memory - limit memory usage.
>- Disk I/O - limit disk I/O.
>- Network - limit network bandwidth.
>
>With the help of cgroups docker engine helps to share available hardware resources with the container and puts a limit on how much resources the container can use.

![diag-cgroup](./container-internals.assets/diag-cgroup.webp) 

Credit: Nginx blogs

