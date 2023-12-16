# Setting up VMs on Debian

## KVM

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
