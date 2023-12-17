# Setting up VMs on Debian

## Install KVM and QEMU

Using the instructions from the [Debian KVM Guide](https://wiki.debian.org/KVM),
we will install:

- [QEMU](https://wiki.debian.org/QEMU) (or Quick Emulator) -- a full system emulator; emulates CPU, memory, and emulated devices to run a guest OS
  - [QEMU Documentation](https://www.qemu.org/docs/master/)
- [KVM](https://wiki.debian.org/KVM) (or Kernel-based Virtual Machine) -- hypervisor that allows executing guest code directly on the host CPU (instead of running a virtualized CPU)
  - [KVM Documentation](https://www.linux-kvm.org/page/HOWTO)

While you can run QEMU by itself, KVM allows us to run guest code directly in the host CPU.
For more information on this relationship, see:
- [KVM and QEMU as Linux Hypervisor](https://medium.com/@jain.sm/kvm-and-qemu-as-linux-hypervisor-18271376449), which has a helpful diagram showing the relationship between QEMU, KVM, Linux, and the (physical) processor
- [Enabling Real-Time Mobile Cloud Computing through Emerging Technologies](https://www.researchgate.net/publication/281177318_Hardware_and_Software_Aspects_of_VM-Based_Mobile-Cloud_Offloading) also has a helpful diagram

```bash
sudo apt install qemu-system libvirt-daemon-system
```

## Install virt-manager

Install [virtinst](https://wiki.libvirt.org/UbuntuKVMWalkthrough.html) in order to provision VMs from the command line.
- [Virtual Machine Manager Documentation](https://virt-manager.org/)
  (very light documentation)
  (more information is available in the virt-manager [man pages](https://github.com/virt-manager/virt-manager/tree/main/man))
- [Virtualisation tools](https://discourse.ubuntu.com/t/virtualisation-tools/13436) provides an example of using tools in the `virtinst` package (namely, `virt-install`)

Add the current user to the libvirt group with:
`sudo usermod -a -G libvirt $USER`

## Starting the Default Virtual Network

Run the following:
```bash
sudo virsh net-list --all  # to confirm default network is not running
sudo virsh net-start default
```

## Creating a Guest VM

To create a Debian guest from the command-line, use:

```bash
sudo virt-install --virt-type kvm --name <your-vm-name> \
--location http://deb.debian.org/debian/dists/bookworm/main/installer-amd64/ \
--os-variant debian12 \
--disk size=50 --memory 8000 \
--graphics none \
--console pty,target_type=serial \
--extra-args "console=ttyS0"
```

- Note that disk size is in GBs. A Kubernetes worker node should be given several tens of GBs.
- For more information on `virt-install` options see the [`virt-install` man page](https://github.com/virt-manager/virt-manager/blob/main/man/virt-install.rst).

To look up OS-specific options that can be used when provisioning a VM:

```bash
sudo apt install -y libosinfo-bin
osinfo-query os | grep debian
```

To update the `osinfo-query` database:
```bash
# Per the instructions at https://libosinfo.org/download/
# visit https://releases.pagure.org/libosinfo/
# and download the latest osinfo-db-DATE.tar.gz file
sudo apt install -y osinfo-db-tools
sudo osinfo-db-import --system ./osinfo-db-DATE.tar.xz
```

A fully automated installation can be achieved using
[preseed](https://wiki.debian.org/DebianInstaller/Preseed).

## Deleting Existing Guest VMs

If you need to clear an existing VM
(including for an installation that was aborted),
first check to see if the VM is listed using `virsh`.
If it is listed, you can use `virsh` to remove the VM.

```bash
virsh list --all
virsh shutdown <your-vm-name>
virsh undefine <your-vm-name>
```

Whether your VM is listed by `virsh` or not,
you can clear your VM's disk files
by using `rm` directly:

```bash
rm /var/lib/libvirt/images/<your-vm-name>.qcow2
```

For more information on the QCOW2 (QEMU Copy-On-Write 2) image format, see:
- [QEMU Copy-On-Write image formate specification](https://github.com/libyal/libqcow/blob/main/documentation/QEMU%20Copy-On-Write%20file%20format.asciidoc)
- [QCOW Wikipedia Entry](https://en.wikipedia.org/wiki/Qcow)

## Setting up bridge networking

https://wiki.debian.org/KVM#Setting_up_bridge_networking
