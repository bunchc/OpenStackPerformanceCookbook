# Chapter 2 - OpenStack Compute Tuning

In this chapter we will address blah blah some things related to openstack compute. As with all prior chapters, we will first preset a discussion of the options and conclusions. For detailed results you can either consult the appendix or this book's companion website.

Specifically, in this chapter we will cover the tuning and performance implications of the following:
- Hypervisor Tuning
- Overcommit
- Ephemeril Disk Placement
- Host Aggregates
- Performance testing Nova

## Hypervisor Tuning

Hypervisor Tuning is one of those subject areas that like much of this book deserve an entire volume unto themselves. Indeed in the case of VMware vSphere and XEN there are already volumes better suited to such. Thusly, as the authors investigated this area and how to handle it, we decided on the following approach:

- Provide general practices cross hypervisors
- Provide specifics tunables for Linux-KVM

### General Hypervisor Tuning

As stated before, OpenStack Nova, aka Compute, can be powered by a number of Hypervisors up to and including bare metal. 

### Tuning Linux-KVM

The approach we'll take for tuning Linux-KVM will focus on three areas:

- CPU
- Memory
- 

While OpenStack Compute (Nova) has a pluggable backend that allows for a broad selection of hypervisors like VMware's ESXi, Bare Metal, or even Docker, the authors have found that that we encounter KVM in the wild far more often.



## Overcommit

OpenStack Compute, also known as Nova, allows you to tune the Overcommit ratios for both Memory and CPU. These in turn can have *huge* performance implications. That is, if you set them too high, your performance can and will drop off non-linearly, and if you set them too low, you run the risk of under-utilizing the resources on hand.

### tl;dr Summary:
The authors recommend 

## Ephemeril Instance Storage

**TODO** Some words here describing ephemeril storage and how it bothers performance.

### tl;dr Summary:
AIO=THREADS vs AIO=NATIVE: use aio=threads (or leave off/default)
The recommended "general KVM" settings are reported to be: format=raw,cache=none,aio=native. However, specifying aio=native requires a change to:

```
root@534359-computeHIO01:~# diff /usr/lib/python2.7/dist-packages/nova/virt/libvirt/config.py{.orig,} 
480a481
>         self.io = "native"
496a498,499
>             if self.io is not None:
>                 drv.set("io", self.io)
```

(More information here: [http://www.sebastien-han.fr/blog/2013/08/12/openstack-unexplained-high-cpu-load-on-compute-nodes/](http://www.sebastien-han.fr/blog/2013/08/12/openstack-unexplained-high-cpu-load-on-compute-nodes/))

And for an RPC deployment should be avoided at this time for this reason. Also, there was no noticeable improvement between aio=threads (default) vs aio=native.

### EXT3 vs EXT4 vs XFS

Conclusion: Use XFS where possible.
After extensive testing, we have found it best to use XFS where you can. XFS will give better write performance overall. In some tests (random write) there doesn't seem to be much to differeniate between all filesystems, however in overall pure write speeds, XFS comes up on top in all tests.

>**AVOID EXT3!** EXT3 gives terrible performance. Unfortunately, it's the default when creating ephemeral storage. That is, when a flavor has been created to be X boot, Y ephemeral – the ephemeral will, by default, be created with EXT3.

This can be changed in nova.conf: 
```
virt_mkfs=['default=mkfs.ext3 -L %(fs_label)s -F %(target)s', 'linux=mkfs.ext3 -L %(fs_label)s -F %(target)s', 'windows=mkfs.ntfs --force --fast --label %(fs_label)s %(target)s']
```

The default is to use EXT3. Highly recommended to change to ext4 make command instead. Where you can though, change to XFS. The problems with this approach with RPC:

    There is no cookbook support for virt_mkfs, so we're left with the default of EXT3. Additionally, when specifying XFS, xfsprogs doesn't exist on the default images supplied causing the autocreate to fail. The workarounds are to specify this in a cloud-init script – please do this.

### RAW vs QCOW2

Conclusion: QCOW2 for RPC
Although there are marginal gains for RAW, given that the RPC cookbooks do not support the libvirt_images_type option required to specify libvirt_images_type=raw, there was little difference in QCOW2 performance vs RAW.

### Testing

In all tests I simply ran the following:

iozone -Rac -f /var/iozonetest -b /tmp/$(hostname).xls 
iozone -Rac -f /mnt/iozonetest -b /tmp/$(hostname)-mnt.xls   

### Ephemeril Disk Conclusion:
After extensive testing, where we are deploying PCIe Warp Drives, the following recommendations are to be made:

- Use software raid 1 in absence of a hardware raid controller
- Format /dev/md0 as xfs, use defaults: ```mkfs.xfs /dev/md0 ```
- Mount /dev/md0 as /var/lib/nova, with fstab: ```/dev/md0 /var/lib/nova xfs nobootwait,noatime,nodiratime 0 0```
- Where performance is needed in the guest, ensure ephemeral storage uses XFS: ```umount /mnt; mkfs.xfs –f /dev/vdb; mount /mnt```
- Default is EXT3 – terrible performance in all situations, at a minimum use EXT4
- If EXT4 is preferred, can use virt_mkfs configuration option in /etc/nova/nova.conf. RPC doesn't support this change.
