---
layout: post
title:  "Create QEMU/KVM template"
date:   2025-06-04
author: Unicorn
tags: QEMU/KVM Debian
---

I use an ansible playbook for deploying virtual machines, by creating zfs datastore(s), copying a template disk image,
loading an XML configuration and define it for QEMU/KVM. During first boot I use tools like `virt-sysprep` to configure,
network interfaces, ssh keys, hostname etc.

So most of my virtual machine deployments utilize **templates**.

A few years ago I changed my storage solution to utilize ZFS. This has been pure enjoyment so far, and combined with the
features of QCOW2 images, the need for LVM becomes non existing for me.

So for this new template I have decided to not use LVM, but only create a primary partition with `ext4` followed by
a `swap` partition.

## QCOW2 disk image

A QCOW2 image is a virtual disk image, with features such as:

- Thin provisioning
- Snapshots
- Compression

Making it a highly versatile format for virtual machine disks.

### Create new image

```bash
qemu-img create -f qcow2 debian-template-2025.qcow2 20G
```

- `-f qcow2` specifies the output disk image format as qcow2.

- `debian-template-2025.qcow2` defines the path and name for the disk image.

- `20G` allocate 20 Gigabytes to this disk image.

## Virtual machine

A new virtual machine can be created in many ways. `virt-install` being one of the most popular.

When my ansible playbook creates new VM's, it alters an XML configuration with the correct parameters and use `virsh define` afterwards.

But for this example I'm going to create a brand new virtual machine without any pre-existing elements.

When I have to solve a "problem" I don't encounter that often, I like to reference the [Red Hat Documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/index)

In `Chapter 3.2` they have examples for creating new virtual machines, using `virt-install`.

```bash
virt-install --name debian-template-2025 \
             --memory 2048 \
             --vcpus 2 \
             --disk /usr/local/lib/libvirt/images/zpool1/templates/debian-template-2025.qcow2 \
             --cdrom /var/lib/libvirt/images/debian-12.11.0-amd64-DVD-1.iso \
             --network network=kvm_pool \
             --os-variant debian11
```

- `--name debian-template-2025` specifies the name of the VM.
- `--memory` defines the memory reservation in MB.
- `--vcpus` defines how many cores is applied to the VM.
- `--disk` specifies the path for the disk image.
- `--cdrom` specifies the path for the iso image used to install a new operating system.
- `--network` defines the network pool and the network interface model (not needed if a default bridge is active).
- `--os-variant` specifies what type of operating system is being used.

### Network config

I dont have any active bridged network on the host, as I'm using virtual functions through sr-iov.
So network interfaces has to be added directly to the XML config, for defining VLAN numbers.

So first power off the newly created VM: `virsh destroy --domain debian-template-2025`

Then add the vlan config to the existing XML

```XML
<interface type='network'>
  <mac address='52:54:00:de:24:8e'/>
  <source network='kvm_pool'/>
  <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
</interface>
```

**becomes**

```XML
<interface type='network'>
  <mac address='52:54:00:de:24:8e'/>
  <source network='kvm_pool'/>
  <vlan>
    <tag id='<VLAN_ID#>'/>
  </vlan>
  <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
</interface>
```

### Operating system

After creating the virtual machine, the operating system needs to be installed.

Using Debian, the installer promts for defining `root` password and a default user. I prefer setting this to placeholders,
for easier provisioning with ansible. And since ansible is changing all passwords anyway, i prefer to `KISS`.

## Prepare VM as template

After the setup of a new virtual machine, it needs to be prepared before being converted into a template.

### APT

I made the following changes to the file `/etc/apt/sources.list`

- Remove `cdrom` source
- Change `http` to `https`

### Custom configuration

This is the time, to do any custom configuration which is meant for all of the guests derived from this template.

I prefer to keep my template as lean and clean as possible and do all of the provisioning afterwards by ansible playbooks.

### Sparsify and sysprep

Before converting this VM into a template I perform a `virt-sparsify` for compressing the image file, followed by a `virt-sysprep` like so:

```bash
virt-sparsify --tmp=/tmp/ --compress debian-template-2025.qcow2 debian-template-2025-sparsify.qcow2
virt-sysprep -a debian-template-2025-sparsify.qcow2
```

## Conclusion

My new template image disk is now ready for being used, by my ansible playbook for creating new virtual machines.

The disk image has now been compressed from around `3.3GB` down to `500MB`.
