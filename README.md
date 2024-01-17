This tool assesses the readiness of a node to run IBM Storage System Scale EMS as a VM. You should run the 'virt-host-validate qemu' command and pass all the checks there before running this tool.

**IMPORTANT**
This tool is does not overrule the official documentation of the product. The requirements stated on the official documenation are the authoritative ones as [IBM Storage Scale System IBM Documentation](https://www.ibm.com/docs/en/ess/6.1.5_lts)

Please have in mind some requirements are hardware related and must be checked with your hardware provider, specially the IOMMU settings on BIOS and motherboard layouts to achieve IOMMU groups in the required way.

If you believe there is a bug or a new request either [open an IBM support case](https://www.ibm.com/mysupport/s/) or a [request for enhancement (RFE)](https://www.ibm.com/support/pages/how-create-and-manage-enhancement-requests-ibm-rfe-community)

A small setup is defined by an EMS that manages no more than two ESS building blocks. A standard setup is an ESS that does not manage more than the supported number of ESS and models.The following list does not override the official documentation refered above about the host requirements:

- Must comply:
 - AMD EPYC processor, single or dual socket. (AMD EPYC Naples not supported)
 - VT-x enabled
 - KVM enabled and installed at OS
 - A PCI passthrough capable system
 - Minimum of 600 GB free on /emsvm
 - /emsvm must be mounted on a local filesystem on a local device (not: iSCSI, FC, NFS, Storage Scale, ...)
 - One quad port Ethernet card for Low Speed Network (LSN), with all ports identical
 - The LSN card must belong to its own IOMMU group, preferrible each port on its own IOMMU group
 - RedHat Enterprise Linux (8.6, 8.7, 9.0, 9.1) running in the host with a valid subscription
 - SELinux disabled on host
 - Host fully compliant with tuned virtual-host profile
 - No other VM[s] running in the host
 - No other workload[s] running in the host
 - Host cannot have Storage Scale software installed
 - At least one ConnectX-6 VPI card for High Speed Network (HSN)
 - Each HSN card must belong to its own IOMMU group, for multi port preferrible each port on its own IOMMU group
 - No more than the three HSN NIC
 - At least two HSN ports
 - No more than six HSN ports
 - For small setups:
   - Host with minimum of 16 cores and 128 GB RAM
   - At least PCIe 3.0 x16 lines for HSN slots
 - For standard setups:
   - Host with minimum of 32 cores and 256 GB RAM
   - At least PCIe 4.0 x16 lines for HSN slots 

  
- Highly recommended:
  - Host boot OS mirrored drives (SW or HW)
  - At least one Ethernet dedicated card to access the host once the EMS VM is running
   Dual port ConnectX card[s]
  - Out of band management for the host
  - Setup kdump 

**Known limitations of this tool**
- Must be run as root or via sudo
- No pull requests are accepted in this tool at this moment

**PREREQUISITES of this tool:** Before running this tool you **must** install the software prerequisites. Those are:
- Python 3.6+
- Installed RPM packages: dmidecode, pciutils, numactl-libs, tuned, virt-manager, libvirt-daemon, libvirt-client, libvirt, libguestfs-tools, virt-install, qemu-img, python3-distro, python3-libvirt, python3-requests, python3-netifaces, python3-ethtool, xz, xz-libs, lsscsi, coreutils
- FIPS node not allowed in the host

**Other information**
If you have more than one quad port Ethernet card in the system the VM will get the one with lowest PCI address. Have that in mind for setting the host IPs.

If your host IPs are not on a quad port card, the card it would never be passed to the VM

In this host:

```
# grep PCI_SLOT_NAME /sys/class/net/*/device/uevent
/sys/class/net/eth0/device/uevent:PCI_SLOT_NAME=0000:45:00.0
/sys/class/net/eth1/device/uevent:PCI_SLOT_NAME=0000:45:00.1
/sys/class/net/eth2/device/uevent:PCI_SLOT_NAME=0000:45:00.2
/sys/class/net/eth3/device/uevent:PCI_SLOT_NAME=0000:45:00.3
/sys/class/net/eth4/device/uevent:PCI_SLOT_NAME=0000:c3:00.0
/sys/class/net/eth5/device/uevent:PCI_SLOT_NAME=0000:c3:00.1
/sys/class/net/eth6/device/uevent:PCI_SLOT_NAME=0000:c3:00.2
/sys/class/net/eth7/device/uevent:PCI_SLOT_NAME=0000:c3:00.3
/sys/class/net/ib0/device/uevent:PCI_SLOT_NAME=0000:27:00.0
/sys/class/net/ib1/device/uevent:PCI_SLOT_NAME=0000:27:00.1
```
The 0000:45:00 would be passed to the VM (eth0 to eth3 on this host)


**emsvm output**
```
# ./emsvm -h
usage: emsvm [-h] (--check-host | --connect-EMS | --start-EMS | --check-EMS | --undefine-EMS) [-d] [-v]

EMSVM host check and run

options:
  -h, --help      show this help message and exit
  --check-host    Assess the readiness of the host to run EMSVM
  --connect-EMS   Connect to the EMSVM console information
  --start-EMS     Start EMSVM in this host
  --check-EMS     Check if EMSVM is UP
  --undefine-EMS  Deletes KVM domain definition from the system, no data of EMSVM is changed or lost
  -d, --debug     Print verbose messages also to shell
  -v, --version   show program's version number and exit

```
To check if the customer host passes the acceptance criteria run with ''--check-host'' parameter

It is important to look at the last line summary. A good run would look similar to:
```
[root@ems1 EMSVM]# ./emsvm --check-host
2022-10-14 01:45:37,608 INFO:	 Welcome to EMS VM version 0.34
2022-10-14 01:45:37,608 INFO:	 Please visit https://github.com/IBM/EMSVM for issues and new versions
2022-10-14 01:45:37,608 INFO:	 Log file with details for this run is saved on: /var/log/emsvm/emsvm_check_host_2022_10_14_01_45_37.log
2022-10-14 01:45:37,614 INFO:	 This system is AMD EPYC based
2022-10-14 01:45:37,614 INFO:	 This host does not have AMD EPYC 1st Generation (Naples)
2022-10-14 01:45:37,693 INFO:	 This host CPU[s] are KVM capable and has extensions enabled
2022-10-14 01:45:37,693 INFO:	 Going to check if /dev/kvm exists
2022-10-14 01:45:37,693 INFO:	 The device /dev/kvm exists
2022-10-14 01:45:37,695 INFO:	 Red Hat Enterprise Linux 8.6 is a supported OS
2022-10-14 01:45:37,695 INFO:	 Checking RPM packages status
2022-10-14 01:45:37,792 INFO:	 All required RPM packages have the expected installation status
2022-10-14 01:45:37,793 INFO:	 Looking that we have at least one NIC for High Speed Network (HSN)
2022-10-14 01:45:37,802 INFO:	 The NIC 'Infiniband controller: Mellanox Technologies MT28800 Family [ConnectX-6 Ex]' is OK as HSN NIC. Going to append PCI address '0000:27:00.0' as HSN NIC
2022-10-14 01:45:37,802 INFO:	 The NIC 'Infiniband controller: Mellanox Technologies MT28800 Family [ConnectX-6 Ex]' is OK as HSN NIC. Going to append PCI address '0000:27:00.1' as HSN NIC
2022-10-14 01:45:37,811 INFO:	 Found at least two HSN ports
2022-10-14 01:45:37,811 INFO:	 Looking that we have at least one quad port Ethernet NIC for Low Speed Network (LSN)
2022-10-14 01:45:37,816 INFO:	 The NIC 'Ethernet controller: Intel Corporation I350 Gigabit Network Connection (rev 01)' is OK as LSN NIC. Going to append PCI address '0000:45:00.' for ports 0, 1, 2 and 3
2022-10-14 01:45:37,816 INFO:	 Host has 32 core[s] which complies with 32 cores required
2022-10-14 01:45:37,816 INFO:	 Total memory is 251.38 GB, which is more than the required 247 GB
2022-10-14 01:45:38,234 INFO:	 Current tune profile is 'virtual-host'
2022-10-14 01:45:38,347 INFO:	 OS settings match the running profile
2022-10-14 01:45:38,347 INFO:	 The path /emsvm has 381.29 GB free excluding reserved space which is more than the required 300 GB
2022-10-14 01:45:38,366 INFO:	 Can connect to 'qemu:///system' and it is this host
2022-10-14 01:45:38,369 INFO:	 KVM executable file /usr/libexec/qemu-kvm exists
2022-10-14 01:45:38,369 INFO:	 
2022-10-14 01:45:38,369 INFO:	 -----------------------
2022-10-14 01:45:38,369 INFO:	 | Summary of this run |
2022-10-14 01:45:38,369 INFO:	 -----------------------
2022-10-14 01:45:38,369 INFO:	 
2022-10-14 01:45:38,369 INFO:	 Log file with details for this run is saved on: /var/log/emsvm/emsvm_check_host_2022_10_14_01_45_37.log
2022-10-14 01:45:38,369 INFO:	 Start of the checks on: 2022/10/14 01:45:37
2022-10-14 01:45:38,369 INFO:	 End of the checks on: 2022/10/14 01:45:38
2022-10-14 01:45:38,370 INFO:	 All tests were passed passed on ems1
2022-10-14 01:45:38,370 INFO:	 The path /emsvm has 381.29 GB free excluding reserved space which is more than the required 300 GB
2022-10-14 01:45:38,370 INFO:	 CPU model is AMD EPYC 7302 16-Core Processor
2022-10-14 01:45:38,370 INFO:	 This host has 2 socket(s)
2022-10-14 01:45:38,370 INFO:	 The VM CPU cores would be 30 core[s]
2022-10-14 01:45:38,370 INFO:	 The VM memory would be 239 GB
2022-10-14 01:45:38,370 INFO:	 The VM HSN PCI address[es] would be ['0000:27:00.0', '0000:27:00.1']
2022-10-14 01:45:38,370 INFO:	 The VM LSN PCI addresseses would be ['0000:45:00.0', '0000:45:00.1', '0000:45:00.2', '0000:45:00.3']
2022-10-14 01:45:38,370 INFO:	 OS is running and matching tune profile virtual-host

OK: All tests passed, you can run EMS VM on ems1 host


[root@ems1 EMSVM]#
```

A bad run might look in many ways but would show as last summary line as read "ERROR" message

To start the EMSVM run with ''--start-EMS'' parameter

To connect to the console of the EMSVM from the host run with ''--connect-EMS'' parameter. To disconnect use Ctrl + ] key combo.
```
[root@x86unode EMSVM]# ./emsvm --connect-EMS
2022-11-04 04:06:34,446 INFO:	 Welcome to EMS VM version 0.49
2022-11-04 04:06:34,446 INFO:	 Please visit https://github.com/IBM/EMSVM for issues and new versions
2022-11-04 04:06:34,447 INFO:	 Log file with details for this run is saved on: /var/log/emsvm/emsvm_connect_EMS_2022_11_04_04_06_34.log
2022-11-04 04:06:34,455 INFO:	 This system is AMD EPYC based
2022-11-04 04:06:34,456 INFO:	 This host does not have AMD EPYC 1st Generation (Naples)
2022-11-04 04:06:34,568 INFO:	 This host CPU[s] are KVM capable and has extensions enabled
2022-11-04 04:06:34,568 INFO:	 Going to check if /dev/kvm exists
2022-11-04 04:06:34,569 INFO:	 The device /dev/kvm exists
2022-11-04 04:06:34,572 INFO:	 Red Hat Enterprise Linux 8.6 is a supported OS
2022-11-04 04:06:34,572 INFO:	 Checking RPM packages status
2022-11-04 04:06:34,739 INFO:	 All required RPM packages have the expected installation status
2022-11-04 04:06:34,834 INFO:	 Could not get current version from repository, please be sure that you are running latest version from https://github.com/IBM/EMSVM
2022-11-04 04:06:39,894 INFO:	 EMSVM KVM domain exists
2022-11-04 04:06:39,895 INFO:	 KVM VM domain EMSVM is already active
Connected to domain 'EMSVM'
Escape character is ^] (Ctrl + ])

Red Hat Enterprise Linux 8.6 (Ootpa)
Kernel 4.18.0-372.26.1.el8_6.x86_64 on an x86_64

emsvm login: 
2022-11-04 04:07:30,894 INFO:	 Console to EMS VM disconnected
[root@x86unode EMSVM]# 
``` 

There are no parameters to stop or delete the EMSVM by choice. Always power off the EMSVM from the EMSVM and having extreme care on Storage Scale quorum and other paramters and situations that might affect the Storage Scale cluster that EMSVM is part of and the host is not aware of.
