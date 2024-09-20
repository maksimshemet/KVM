Here’s a blog post-styled version that includes the requested information and links:

---

# Setting Up KVM Virtualization with CoreOS: Basic Commands and Quick Reference

If you're looking to get started with KVM (Kernel-based Virtual Machine) virtualization specifically for deploying **CoreOS**, you're in the right place! This guide will walk you through some of the **basic `virsh` commands** and provide a concise **KVM VM setup** with CoreOS using **QEMU**. We’ll also provide links to official documentation for further reading and configuration options.

## Why KVM and CoreOS?

KVM is a powerful, open-source virtualization technology integrated into the Linux kernel. It allows you to run multiple virtual machines on Linux hosts, leveraging your hardware's CPU virtualization extensions. **CoreOS** (or **Fedora CoreOS**) is an ideal candidate for containerized environments, providing a minimal operating system designed for running containers at scale. Together, they offer a highly scalable and efficient solution for your virtualized workloads.

---

## Basic `virsh` Commands

### VM Management
Here are the most commonly used `virsh` commands to manage your VMs:

- **Start a VM**:  
  Start your virtual machine with optional parameters like `--console` for accessing the VM console directly.
  ```bash
  virsh start <domain> [--console] [--paused] [--autodestroy] [--bypass-cache] [--force-boot]
  ```

- **Resume a Paused VM**:  
  Bring a paused VM back to a running state.
  ```bash
  virsh resume <domain>
  ```

- **Suspend a VM**:  
  Temporarily suspend a running VM.
  ```bash
  virsh suspend <domain>
  ```

- **Shutdown a VM**:  
  Gracefully shut down the VM using ACPI.
  ```bash
  virsh shutdown <domain> --mode acpi
  ```

- **Force Stop a VM**:  
  Forcibly stop a VM if it's unresponsive.
  ```bash
  virsh destroy <domain>
  ```

- **Remove a VM**:  
  Remove a VM from the system (its configuration only, not its disk).
  ```bash
  virsh undefine <domain>
  ```

---

## Networking with `virsh`

Networking plays a crucial role in virtualization, especially for VMs that need to communicate with other network services. Here are some commands to manage networking within KVM:

- **List Available Networks**:  
  Check which virtual networks are active.
  ```bash
  virsh net-list
  ```

- **View DHCP Leases**:  
  Get information on DHCP leases from a specific virtual network.
  ```bash
  virsh net-dhcp-leases
  ```

- **Query VM Network Interfaces**:  
  To list all the network interfaces attached to a specific VM:
  ```bash
  virsh domiflist <domain>
  ```

You can find the full XML reference for configuring networks in KVM in the official [libvirt network XML documentation](https://libvirt.org/formatnetwork.html).

---

## Block Devices and Disk Management

- **List Block Devices**:  
  To view the block devices (disks) attached to a VM, use:
  ```bash
  virsh domblklist --domain <domain>
  ```

For more information on domain and device configuration, check the [libvirt domain XML documentation](https://libvirt.org/formatdomain.html).

---

## Setting up CoreOS on KVM with `virt-install`

CoreOS simplifies container management by offering an operating system tailored for clusters. Follow these steps to create and launch a CoreOS VM using `virt-install`.

1. **Define Ignition Configuration**:  
   First, define the ignition config file to be used by the CoreOS VM.
   ```bash
   IGNITION_CONFIG="/home/imgs/configs/example.ign"
   IGNITION_DEVICE_ARG=(--qemu-commandline="-fw_cfg name=opt/com.coreos/config,file=${IGNITION_CONFIG}")
   ```

2. **Install the VM**:
   Use the following command to install a new Fedora CoreOS VM:
   ```bash
   virt-install --name="PXE-coreOS" --vcpus=2 --memory=4096 \
       --os-variant="fedora-coreos-stable" --import \
       --disk="size=20,backing_store=/home/imgs/fedora-coreos-40.20240825.3.0-qcow2" \
       --network bridge=virbr0 "${IGNITION_DEVICE_ARG[@]}"
   ```

For more details on provisioning Fedora CoreOS on QEMU/KVM, see the official [Fedora CoreOS QEMU provisioning guide](https://docs.fedoraproject.org/en-US/fedora-coreos/provisioning-qemu/).

---

## Boot Device Configuration

When configuring boot devices in KVM, you can specify which device should be used during the boot process. The `dev` attribute takes values such as `fd`, `hd`, `cdrom`, or `network`.

Here’s an example of a boot device configuration:
```bash
<boot dev='hd'/>
```

This configuration allows you to prioritize the boot order for each device. For more detailed configurations, refer to the [libvirt domain XML documentation](https://libvirt.org/formatdomain.html).

---

## Additional Tools for CoreOS

### SELinux Configuration

When setting up CoreOS, you'll need to configure SELinux labels to ensure the VM has access to certain files. For example:
```bash
chcon --verbose --type svirt_home_t ${IGNITION_CONFIG}
```

### Generating Secure Passwords

If you need a secure password for your VM or any other service, use the `mkpasswd` utility from the `whois` package:
```bash
mkpasswd
```

---

## Useful Links for Further Reading

Here are some additional resources that might be useful as you dive deeper into KVM and CoreOS setups:

- [Libvirt Network XML Documentation](https://libvirt.org/formatnetwork.html)
- [Libvirt Domain XML Documentation](https://libvirt.org/formatdomain.html)
- [Virsh Command Reference](https://libvirt.org/manpages/virsh.html#cd)
- [Butane Configuration Tool for Fedora CoreOS](https://coreos.github.io/butane/specs/)
- [Fedora CoreOS Provisioning with QEMU/KVM](https://docs.fedoraproject.org/en-US/fedora-coreos/provisioning-qemu/)

---

## Conclusion

This post covered the **basic `virsh` commands** and the essential steps for setting up a CoreOS VM on KVM. We hope this guide helps you get started with **KVM virtualization** for containerized environments. Be sure to explore the linked resources for more advanced configurations and fine-tuning!

Happy virtualizing!
```

This blog post provides a streamlined guide for setting up CoreOS on KVM with helpful links to official documentation, making it easy for beginners to get started. Let me know if you need any further adjustments!