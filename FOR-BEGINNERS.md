# Creating a Fedora CoreOS VM with KVM on Fedora/CentOS/Ubuntu

This guide will walk you through setting up a Fedora CoreOS virtual machine using KVM on a Fedora (or similar) system. All commands should be run as `root`.

## Step 1: Install Fedora/CentOS/Ubuntu
- Fedora is recommended for this guide, but the steps can be adapted for other distributions.

## Step 2: Install KVM

For Ubuntu, follow this [KVM installation guide](https://ubuntu.com/blog/kvm-hyphervisor).  
For Fedora, follow this [virtualization setup guide](https://docs.fedoraproject.org/en-US/quick-docs/virtualization-getting-started/).

### Fedora-specific Installation Steps:

**Install Virtualization Software:**

1. For new Fedora installations:  
   During installation, select the `Virtualization` option in the Base Group.

2. For existing Fedora installations:  
   Use the following commands to install the virtualization tools.

   ```bash
   $ dnf groupinfo virtualization
   ```

   You will see a list of mandatory, default, and optional packages.

   Install the mandatory and default packages using:

   ```bash
   dnf install @virtualization
   ```

   Alternatively, install all packages (mandatory, default, and optional) with:

   ```bash
   dnf group install --with-optional virtualization
   ```

**Start the libvirtd service:**

```bash
systemctl start libvirtd
```

**Enable libvirtd service on boot:**

```bash
systemctl enable libvirtd
```

**Verify KVM is properly configured:**

```bash
$ lsmod | grep kvm
```

If the output lists `kvm_intel` or `kvm_amd`, KVM is configured correctly.

## Step 3: Create a Folder for CoreOS Images

```bash
mkdir /home/imgs
```

## Step 4: Download and Prepare Fedora CoreOS Image

Navigate to the `/home/imgs` directory:

```bash
cd /home/imgs
```

Download the CoreOS image using `wget`:

```bash
wget https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/40.20240825.3.0/x86_64/fedora-coreos-40.20240825.3.0-qemu.x86_64.qcow2.xz
```

Install `unxz` and extract the image:

```bash
dnf install unxz
unxz fedora-coreos-40.20240825.3.0-qemu.x86_64.qcow2.xz
```

Check the contents:

```bash
ls -la
```

You should see the `.qcow2` file:

```bash
-rw-r--r--. 1 root root 1765081088 fedora-coreos-40.20240825.3.0-qemu.x86_64.qcow2
```

## Step 5: Set Up the Virtual Machine Environment

Create a directory for your virtual machines:

```bash
mkdir vms_env
cd vms_env
```

Create a file called `coreOS_template.xml`. This file will be your virtual machine configuration template.

Here is a basic XML template to define the CoreOS VM:

```xml
<domain type='kvm' id='10' xmlns:qemu='http://libvirt.org/schemas/domain/qemu/1.0'>
  <name>PXE-coreOS</name>
  <metadata>
    <libosinfo:libosinfo xmlns:libosinfo="http://libosinfo.org/xmlns/libvirt/domain/1.0">
      <libosinfo:os id="http://fedoraproject.org/coreos/stable"/>
    </libosinfo:libosinfo>
  </metadata>
  <memory unit='KiB'>4194304</memory>
  <currentMemory unit='KiB'>4194304</currentMemory>
  <vcpu placement='static'>2</vcpu>
  <resource>
    <partition>/machine</partition>
  </resource>
  <os>
    <type arch='x86_64' machine='pc-q35-8.2'>hvm</type>
    <boot dev='hd'/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <vmport state='off'/>
  </features>
  <cpu mode='custom' match='exact' check='full'>
    <model fallback='forbid'>Opteron_G3</model>
    <feature policy='require' name='ibpb'/>
    <feature policy='require' name='spec-ctrl'/>
    <feature policy='require' name='ssbd'/>
    <feature policy='require' name='virt-ssbd'/>
    <feature policy='disable' name='monitor'/>
    <feature policy='require' name='x2apic'/>
    <feature policy='require' name='hypervisor'/>
    <feature policy='require' name='vme'/>
    <feature policy='disable' name='svm'/>
  </cpu>
  <clock offset='utc'>
    <timer name='rtc' tickpolicy='catchup'/>
    <timer name='pit' tickpolicy='delay'/>
    <timer name='hpet' present='no'/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled='no'/>
    <suspend-to-disk enabled='no'/>
  </pm>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2' discard='unmap'/>
      <source file='/var/lib/libvirt/images/PXE-coreOS.qcow2' index='1'/>
      <backingStore type='file' index='2'>
        <format type='qcow2'/>
        <source file='/home/imgs/fedora-coreos-40.20240825.3.0-qemu.x86_64.qcow2'/>
        <backingStore/>
      </backingStore>
      <target dev='vda' bus='virtio'/>
      <alias name='virtio-disk0'/>
      <address type='pci' domain='0x0000' bus='0x04' slot='0x00' function='0x0'/>
    </disk>
    <controller type='usb' index='0' model='qemu-xhci' ports='15'>
      <alias name='usb'/>
      <address type='pci' domain='0x0000' bus='0x02' slot='0x00' function='0x0'/>
    </controller>
    <controller type='pci' index='0' model='pcie-root'>
      <alias name='pcie.0'/>
    </controller>
    <controller type='pci' index='1' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='1' port='0x10'/>
      <alias name='pci.1'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0' multifunction='on'/>
    </controller>
    <controller type='pci' index='2' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='2' port='0x11'/>
      <alias name='pci.2'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x1'/>
    </controller>
    <controller type='pci' index='3' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='3' port='0x12'/>
      <alias name='pci.3'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x2'/>
    </controller>
    <controller type='pci' index='4' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='4' port='0x13'/>
      <alias name='pci.4'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x3'/>
    </controller>
    <controller type='pci' index='5' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='5' port='0x14'/>
      <alias name='pci.5'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x4'/>
    </controller>
    <controller type='pci' index='6' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='6' port='0x15'/>
      <alias name='pci.6'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x5'/>
    </controller>
    <controller type='pci' index='7' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='7' port='0x16'/>
      <alias name='pci.7'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x6'/>
    </controller>
    <controller type='pci' index='8' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='8' port='0x17'/>
      <alias name='pci.8'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x7'/>
    </controller>
    <controller type='pci' index='9' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='9' port='0x18'/>
      <alias name='pci.9'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0' multifunction='on'/>
    </controller>
    <controller type='pci' index='10' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='10' port='0x19'/>
      <alias name='pci.10'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x1'/>
    </controller>
    <controller type='pci' index='11' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='11' port='0x1a'/>
      <alias name='pci.11'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x2'/>
    </controller>
    <controller type='pci' index='12' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='12' port='0x1b'/>
      <alias name='pci.12'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x3'/>
    </controller>
    <controller type='pci' index='13' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='13' port='0x1c'/>
      <alias name='pci.13'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x4'/>
    </controller>
    <controller type='pci' index='14' model='pcie-root-port'>
      <model name='pcie-root-port'/>
      <target chassis='14' port='0x1d'/>
      <alias name='pci.14'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x5'/>
    </controller>
    <controller type='sata' index='0'>
      <alias name='ide'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x1f' function='0x2'/>
    </controller>
    <controller type='virtio-serial' index='0'>
      <alias name='virtio-serial0'/>
      <address type='pci' domain='0x0000' bus='0x03' slot='0x00' function='0x0'/>
    </controller>
    <interface type='bridge'>
      <source bridge='virbr0'/>
      <target dev='vnet9'/>
      <model type='virtio'/>
      <alias name='net0'/>
      <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
    </interface>
    <serial type='pty'>
      <source path='/dev/pts/7'/>
      <target type='isa-serial' port='0'>
        <model name='isa-serial'/>
      </target>
      <alias name='serial0'/>
    </serial>
    <console type='pty' tty='/dev/pts/7'>
      <source path='/dev/pts/7'/>
      <target type='serial' port='0'/>
      <alias name='serial0'/>
    </console>
    <channel type='unix'>
      <source mode='bind' path='/run/libvirt/qemu/channel/10-PXE-coreOS/org.qemu.guest_agent.0'/>
      <target type='virtio' name='org.qemu.guest_agent.0' state='disconnected'/>
      <alias name='channel0'/>
      <address type='virtio-serial' controller='0' bus='0' port='1'/>
    </channel>
    <channel type='spicevmc'>
      <target type='virtio' name='com.redhat.spice.0' state='disconnected'/>
      <alias name='channel1'/>
      <address type='virtio-serial' controller='0' bus='0' port='2'/>
    </channel>
    <input type='tablet' bus='usb'>
      <alias name='input0'/>
      <address type='usb' bus='0' port='1'/>
    </input>
    <input type='mouse' bus='ps2'>
      <alias name='input1'/>
    </input>
    <input type='keyboard' bus='ps2'>
      <alias name='input2'/>
    </input>
    <graphics type='spice' port='5900' autoport='yes' listen='127.0.0.1'>
      <listen type='address' address='127.0.0.1'/>
      <image compression='off'/>
    </graphics>
    <sound model='ich9'>
      <alias name='sound0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x1b' function='0x0'/>
    </sound>
    <audio id='1' type='spice'/>
    <video>
      <model type='virtio' heads='1' primary='yes'/>
      <alias name='video0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x0'/>
    </video>
    <redirdev bus='usb' type='spicevmc'>
      <alias name='redir0'/>
      <address type='usb' bus='0' port='2'/>
    </redirdev>
    <redirdev bus='usb' type='spicevmc'>
      <alias name='redir1'/>
      <address type='usb' bus='0' port='3'/>
    </redirdev>
    <watchdog model='itco' action='reset'>
      <alias name='watchdog0'/>
    </watchdog>
    <memballoon model='virtio'>
      <alias name='balloon0'/>
      <address type='pci' domain='0x0000' bus='0x05' slot='0x00' function='0x0'/>
    </memballoon>
    <rng model='virtio'>
      <backend model='random'>/dev/urandom</backend>
      <alias name='rng0'/>
      <address type='pci' domain='0x0000' bus='0x06' slot='0x00' function='0x0'/>
    </rng>
  </devices>
  <seclabel type='dynamic' model='selinux' relabel='yes'>
    <label>system_u:system_r:svirt_t:s0:c266,c632</label>
    <imagelabel>system_u:object_r:svirt_image_t:s0:c266,c632</imagelabel>
  </seclabel>
  <seclabel type='dynamic' model='dac' relabel='yes'>
    <label>+107:+107</label>
    <imagelabel>+107:+107</imagelabel>
  </seclabel>
  <qemu:commandline>
    <qemu:arg value='-fw_cfg'/>
    <qemu:arg value='name=opt/com.coreos/config,file=/home/imgs/example.ign'/>
  </qemu:commandline>
</domain>
```

## Step 6: Create an Ignition File

You will need an Ignition file for the first boot of Fedora CoreOS. This file will provide initial configuration details like user credentials.

Create an Ignition file called `example.ign` with the following content:

```json
{
  "ignition": {
    "version": "3.4.0"
  },
  "passwd": {
    "users": [
      {
        "name": "core",
        "passwordHash": "$y$j9T$SolIFc83u0JvDikjVrbLA1$wzOsjWpbK31dii4jaDjAsFqyFL1zV8BgXN5Cv5rjvU6",
        "uid": 1337
      }
    ]
  }
}
```

For more information on Ignition files and how to create them, check out [Fedora CoreOS Ignition Documentation](https://docs.fedoraproject.org/en-US/fedora-coreos/producing-ign/) and [Butane YAML Docs](https://coreos.github.io/butane/config-fcos-v1_5/).

Before creating the VM, set the appropriate security context for the Ignition file:

```bash
chcon --verbose --type svirt_home_t example.ign
```

Now, create the virtual machine using your template:

```bash
virsh create --file coreOS_template.xml --validate
```

## Step 7: Access the Virtual Machine Console

To access the VM console, use the following command:

```bash
virsh console --domain PXE-coreOS
```

Login using:

```bash
login: core
password: password
```

Congratulations! You have successfully created a Fedora CoreOS VM.

## Step 8: Verify the VM Status

To view information about your newly created VM, run:

```bash
virsh dominfo --domain PXE-coreOS
```

The output should be similar to the following:

```
Id:             11
Name:           PXE-coreOS
UUID:           787960d7-3187-4905-ad80-c8fb47e3c234
OS Type:        hvm
State:          running
CPU(s):         2
Max memory:     4194304 KiB
Used memory:    4194304 KiB
Persistent:     no
Autostart:      disable
```

### Note:
- The "Persistent: no" status indicates that the VM is not persistent, meaning it will not exist after a shutdown. You can modify this behavior by converting the VM to a persistent one. Look into the `virsh define` command to achieve persistence.

Good luck setting up your CoreOS virtual machine!

--- 

Feel free to modify and adapt this document as necessary!