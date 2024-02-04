unshare --uts --pid --net --mount --ipc --fork

mkdir /sys/fs/cgroup/cpu/container1
echo 100000 > /sys/fs/cgroup/cpu/container1/cpu.cfs_quota_us
echo 0 > /sys/fs/cgroup/cpu/container1/tasks
echo $$ > /sys/fs/cgroup/cpu/container1/tasks

debootstrap focal ./ubuntu-rootfs http://archive.ubuntu.com/ubuntu/

mount -t proc none ./ubuntu-rootfs/proc
mount -t sysfs none ./ubuntu-rootfs/sys
mount -o bind /dev ./ubuntu-rootfs/dev
chroot ./ubuntu-rootfs /bin/bash

apt update
apt install nginx
service nginx start