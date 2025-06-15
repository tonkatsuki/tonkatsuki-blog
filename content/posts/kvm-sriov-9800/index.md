+++
title = 'Getting X722 SR-IOV working with Cisco 9800-CL on KVM'
date = 2025-06-15T00:00:10-05:00
tags = ["networking", "wireless", "linux"]
featured_image = '/posts/kvm-sriov-9800/feature.png'
draft = false
+++

# Intro

I wanted to take time to detail some steps in setting up SR-IOV on Intel X722 cards to enable having your Cisco 9800-CL controller be able to do PCI Passthrough with KVM as your hypervisor.

This should work with any X710 series card as well. Note that Cisco only officially supports X710 or Ciscoized X710 adapters, but the X722 worked fine. We tried for many months with UCS VIC 12000 series cards, but Cisco has left us hanging dry as they won't officially support them, and from what I can tell, the 9800-CL lacks the drivers in the image itself to be able to make them work.

One final piece: I was not able to get any version of the 9800-CL for KVM working with this except for IOS-XE 17.15.x and above. I'm not sure if this was because my card was a X722 or not, as the SR-IOV guide posted by Cisco should predate 17.15.x's release, but regardless, this was the version I used. As of 6/15/25 it's not a gold star release from TAC *unless* you are running Wi-Fi7 AP's, but given it's the most recent extended release, it's been fine to use in my environment.

Tested OS and Kernel: Ubuntu 24.04.2 LTS Kernel 6.8.0-60

All the commands are expecting sudo, so ensure you are running as root or prepend `sudo` to each command.

Let's get started!

# BIOS

Ensure you have Intel IOMMU enabled in your BIOS as well as SR-IOV. You may need to enable SR-IOV directly on the root interface as well via Network Options. I won't cover this as this will be server vendor dependent, but just check your server guide on enabling those.

# Netplan

I am using Ubuntu, but feel free to use whatever distro you like, just be aware distro-specific settings may change this, but many things we are doing are kernel level and should be the same from distro to distro.

One of the differences is not all distros use Netplan. This isn't strictly necessary to mess with, but I recommend you do this for sure:
````text
/etc/netplan/10-standard-netcfg.yaml
````

```YAML 
network:
  version: 2
  ethernets:
  eno1np0:
    dhcp4: no
  eno2np1:
    dhcp4: no

  vlans:
    bond0.10:
      id: 10
      link: bond0
      addresses:
        - 10.0.10.10/24
      routes:
        - to: 0.0.0.0/0
        - via: 10.0.10.1
    bond0.50:
      id: 50
      link: bond0

bonds:
  bond0:
    interfaces:
      - eno1np0
      - eno2np1
    parameters:
      mode: active-backup
      primary: eno1np0
      mii-monitor-interval: 100
```

First we'll have two uplinks to the server in a bond without LACP, just using active-backup. Feel free to change the mode to 802.3ad and do something like vPC or MCLAG however. We'll have bond0.10 be a trunked VLAN for ubuntu host management, and then we'll also declare a bond0.50 for VLAN 50 so we have one NIC via VirtIO go to the 9800-CL so that we can have Gi1 be our server management address. Feel free to set up another bond if you plan on doing HA SSO as that would be mapped to Gi3, in this case, I don't use HA SSO on my 9800-CL's so I won't set that up.

Apply that netplan configuration via `netplan apply`. If you used cloud-init make sure to disable cloud-init by making a file to disable it:

>echo "network: {config: disabled}" | sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg

Also delete the default cloud init in your netplan config:

>rm /etc/netplan/50-cloud-init.yaml

# APT Packages

We'll need the following packages installed:

>sudo apt install qemu-system-x86 qemu-utils uml-utilities socat virtinst cockpit cockpit-machines cockpit-pcp

Note the last three are for cockpit, a webUI management that can also manage virtual machines, and the last package is for metrics collections. They're entirely optional but honestly it can be helpful having, so I recommend them. The remaining packages are for our QEMU virtualization.

# GRUB

We need to update grub to enable IOMMU
>sudo vi /etc/default/grub

On the line `GRUB_CMDLINE_LINUX_DEFAULT=` add "intel_iommu=on iommu=pt" so the line looks like so:
>GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on iommu=pt"

Save and exit and run `sudo update-grub` to update grub. We will need to reboot, but don't do so just yet.

# IAVF

One small note here. The X710 drivers use i40e as the default driver. However, when using SR-IOV, we create virtual functions on a physical function card. The driver for VF's is IAVF. We want to disable IAVF on the **host** because we do not want the host to ever take control of the VF's or attempt to load them. This ensures if you ever reboot your wireless controller or any VMs using SR-IOV that they do not have issues. The IAVF driver will still load on the 9800-CL itself and be able to use the VF's.
>echo "blacklist iavf" | sudo tee /etc/modprobe.d/blacklist-vf.conf

Once doing so, update-initramfs. If you're doing anything with kernel changes adjust accordingly.

>update-initramfs -u


# Persistent VF Settings

We need to create a systemd service that runs on boot to enable certain SR-IOV settings. This is detailed in the Cisco 9800-CL guide [here](https://www.cisco.com/c/en/us/td/docs/wireless/controller/9800/technical-reference/c9800-cl-dg.html#AppendixDEnablingandusingtheSRIOVNICinKVM) under Step 4 for the KVM guide. I will rewrite it here with a minor modification I recommend:

````text
/usr/bin/sriov-config
````
```BASH {style=emacs}
#!/bin/sh 
echo 1 > /sys/class/net/eno1np0/device/sriov_numvfs 
ip link set dev eno1np0 vf 0 trust on 
ip link set eno1np0 vf 0 spoofchk off 
ip link set eno1np0 vf 0 mac 3c:31:fc:92:95:96

echo 1 > /sys/class/net/eno4np3/device/sriov_numvfs 
ip link set dev eno2np1 vf 0 trust on 
ip link set eno2np1 vf 0 spoofchk off 
ip link set eno2np1 vf 0 mac 5e:7c:55:f7:f7:97
```
Note: The MAC Addresses I chose are simply random.
Next make it executable:
>chmod 777 /usr/bin/sriov-config

Finally we'll make a service to run at boot:
````text
/usr/lib/systemd/system/sriov.service
````
```systemd
[Unit] 
Description=SR-IOV configuration
After=network-online.target rc-local.service
Wants=network-online.target
Before=getty.target
[Service] 
Type=oneshot 
ExecStart=/usr/bin/sriov-config 
[Install] 
WantedBy=multi-user.target
```
And finally enable it via `systemctl --now enable sriov.service`

The change I made was on the SR-IOV service side, notably, make sure it occurs after network-online and requires it. This way the network card is verified to be up before applying the SR-IOV settings.

# KVM Deployment

If you've done the above and not rebooted yet, please do so now. You will need to do so in order to blacklist the IAVF driver and also enable IOMMU.

The final piece is deploying the KVM machine. We will do this via command line, but feel free to log into your cockpit instance via https://{your_host_ip}:9090 and do this manually there.

Before we can add the PCI passthrough device via KVM, let's find out the PCI ID:
>lspci -vvv | grep Eth

You should have a device called **Intel Corporation Ethernet Virtual Function 700 Series** with a PCI ID at the start, in this case, mine were 08:02.0 and 08:0e.0

You can now run the virt-install script with the bottom two lines specifying the PCI passthrough device:
Note: Cisco has a decent guide detailing how to do this in various ways [here](https://www.cisco.com/c/en/us/td/docs/wireless/controller/9800/technical-reference/c9800-cl-dg.html#Deployingthe9800CLonLinuxKVM) and [here](https://www.cisco.com/c/en/us/td/docs/wireless/controller/9800/9800-cloud/installation/b-c9800-cl-install-guide/installing_the_controller_in_kvm_environment.html).
```bash
sudo virt-install \
  --os-variant=rhel10-unknown \
  --virt-type=kvm \
  --name wifi-01 \
  --ram 40960 \
  --vcpus=19 \
  --hvm \
  --cdrom=/var/lib/libvirt/images/C9800-CL-universalk9.17.15.03.iso \
  --disk path=/var/lib/libvirt/images/wifi-01.qcow2,size=16,bus=virtio,format=qcow2 \
  --graphics vnc \
  --network type=direct,source=bond0.50,source_mode=bridge,model=virtio \
  --host-device=pci_0000_08_02_0 \
  --host-device=pci_0000_08_0e_0 \
```

This should deploy automatically. Note modify the ram and vCPUs to your liking based on template versions. As of 17.15.x this is the configuration for the largest template possible, and I was personally fine with that as these are on hosts with only the wireless controllers, so there were plenty of resources to use.

# 9800-CL Configuration

I won't go deeply into the 9800-CL configuration since it's well documented by Cisco. My only tidbits are:

- If you are doing LACP/802.3ad, create a Port Channel in the 9800-CL under Configuration --> Interface --> Logical.
- Ensure your Gi1, aka the VirtIO intf, is NOT your Wireless Management interface. Use a trunked VLAN on the SR-IOV config.
- Ensure you prune VLANs on the switch side. There is a security implication with using SR-IOV so please ensure your uplinked ports are using only the specific VLANs you require.

Once the WLC is booted, you can run `show platform software vnic-if interface-mapping` to verify your Gi2/Gi3 should be using the `net_iavf` driver. Ensure the mac addresses match what you set up on the script as well. If the mac addresses change per reboot, they will not map to the correct Gi2/Gi3 interfaces and you may break your wireless traffic!
