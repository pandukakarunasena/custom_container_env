#Build your own container using linux cgroups, namespaces and chroot

Containers use linux features such like cgroups and namespaces.

- cgroups - limit and monitor system resources allocated to a set of processes

- namespaces - create and isolated enviroment for the process

By Combining above features a container can be built which is isolated from other processes and monitor the system resources.

The linux /sys/fs/cgroup location can be used to create cgroups as it works as an API to manage the cgroups.

The below set of controllers can be found in the sys/fs/cgroup. 

- cpu
- memory
- blkio

etc....

A process can be attached to each controller by creating a separate folder inside of the controller.

The namespaces can be found for a process in the /proc directory under ns sub directory.

Follow the below steps to create isolated namespaces and cgroup to create a container environment.

###Step 1: Setting Up the Namespace


To make a separate space, we first create a new area.

We use a command called unshare and point out various spaces (--uts, --pid, --net, --mount, and --ipc). These spaces give our container its own system labels and things it needs to work.


```unshare --uts --pid --net --mount --ipc --fork```

###Step 2: Configuring the cgroups

Cgroups (control groups) are useful for handling how our container uses system resources.

We're going to set up a new cgroup for our container and limit how much CPU it can use.

`mkdir /sys/fs/cgroup/cpu/container1
echo 100000 > /sys/fs/cgroup/cpu/container1/cpu.cfs_quota_us
echo 0 > /sys/fs/cgroup/cpu/container1/tasks
echo $$ > /sys/fs/cgroup/cpu/container1/tasks`

1. `mkdir /sys/fs/cgroup/cpu/container1`: This command creates a new directory named "container1" within the /sys/fs/cgroup/cpu/ directory. This directory will be used as a control group (cgroup) specifically for managing CPU resources.

2. `echo 100000 > /sys/fs/cgroup/cpu/container1/cpu.cfs_quota_us`: This sets the CPU quota limit for the "container1" cgroup. The value 100000 represents the quota in microseconds, indicating the maximum amount of CPU time that processes in this cgroup can use over a certain period.

3. `echo 0 > /sys/fs/cgroup/cpu/container1/tasks`: By writing 0 to the "tasks" file, any previously assigned processes to the "container1" cgroup are removed. This ensures that no processes are initially associated with the cgroup.

4. `echo $$ > /sys/fs/cgroup/cpu/container1/tasks`: This command assigns the current process ID (PID) of the executing shell or script (represented by $$) to the "tasks" file within the "container1" cgroup. Subsequently, any child processes spawned by the shell or script will also be part of the "container1" cgroup, subject to the specified CPU quota limits.

###Step 3: Building the Root File System

We establish the file system for our container by utilizing debootstrap to configure a basic Ubuntu environment within a directory named "ubuntu-rootfs."

This directory functions as the foundational file system for our container.

In the terminal window, the debootstrap command is executed with the following syntax:

```
debootstrap focal ./ubuntu-rootfs http://archive.ubuntu.com/ubuntu/
```

Here, the first argument, 'focal,' specifies the Ubuntu release to install, indicating Ubuntu 20.04 (Focal Fossa) in this instance.

The second argument, './ubuntu-rootfs,' designates the directory where the Ubuntu environment is installed—in this case, the ubuntu-rootfs directory.

The third argument, 'http://archive.ubuntu.com/ubuntu/,' specifies the URL of the Ubuntu repository utilized for the installation.


###Step 4: Mounting and Chrooting into the Container

Next, the proc, sys, dev directories are mounted to the ./ubuntu-rootfs directory.

```
mount -t proc none ./ubuntu-rootfs/proc
mount -t sysfs none ./ubuntu-rootfs/sys
mount -o bind /dev ./ubuntu-rootfs/dev
chroot ./ubuntu-rootfs /bin/bash
```

`mount -t proc none ./ubuntu-rootfs/proc`: This command mounts the proc filesystem inside the "ubuntu-rootfs" directory. The proc filesystem provides information about processes and system resources. In this case, it is mounted at "./ubuntu-rootfs/proc."

`mount -t sysfs none ./ubuntu-rootfs/sys`: This command mounts the sysfs filesystem inside the "ubuntu-rootfs" directory. The sysfs filesystem exposes information about the kernel, devices, and other system-related details. It is mounted at "./ubuntu-rootfs/sys."

`mount -o bind /dev ./ubuntu-rootfs/dev`: Here, the command binds the "/dev" directory to "./ubuntu-rootfs/dev." This means that the device files in the host system's "/dev" directory are made accessible within the "ubuntu-rootfs" directory.

`chroot ./ubuntu-rootfs /bin/bash`: The chroot command changes the root directory to "./ubuntu-rootfs," creating an isolated environment. After this, it executes "/bin/bash" as the new root shell. Essentially, you are entering a chrooted environment, and any subsequent commands will be executed as if the "ubuntu-rootfs" directory is the root of the filesystem. This is useful for performing operations within the context of the "ubuntu-rootfs" environment, separate from the host system.

REF : proc_sys_dev.txt to understand the difference between proc, sys and dev

###Step 5: Running Applications within the Container

With our container environment established, we have the capability to install and execute applications within it.

For instance, in this demonstration, we will install the Nginx web server to showcase the behavior of applications within the container.

```
(container) $ apt update
(container) $ apt install nginx
(container) $ service nginx start
```