# 🛠️ Prerequisites Installation & Setup

Since I'm still learning, this particular documentation will be quite thoroughly written for personal reference in the future.

I'm using **Parrot Security 6.4** (lorikeet) with MATE 1.26.0 as the host system. Here's my complete system specs:
```
┌─[kev@parrot]─[~]
└──╼ $neofetch
  `:oho/-`
`mMMMMMMMMMMMNmmdhy-                      kev@parrot 
 dMMMMMMMMMMMMMMMMMMs`                    ---------- 
 +MMsohNMMMMMMMMMMMMMm/                   OS: Parrot Security 6.4 (lorikeet) x8 
 .My   .+dMMMMMMMMMMMMMh.                 Host: Aspire A315-41 V1.18 
  +       :NMMMMMMMMMMMMNo                Kernel: 6.12.32-amd64 
           `yMMMMMMMMMMMMMm:              Uptime: 2 days, 9 hours, 6 mins 
             /NMMMMMMMMMMMMMy`            Packages: 3719 (dpkg), 5 (flatpak) 
              .hMMMMMMMMMMMMMN+           Shell: bash 5.2.15 
                  ``-NMMMMMMMMMd-         Resolution: 1366x768 
                     /MMMMMMMMMMMs`       DE: MATE 1.26.0 
                      mMMMMMMMsyNMN/      WM: Metacity (Marco) 
                      +MMMMMMMo  :sNh.    Theme: Icy-Dark [GTK2/3] 
                      `NMMMMMMm     -o/   Icons: ara [GTK2/3] 
                       oMMMMMMM.          Terminal: mate-terminal 
                       `NMMMMMM+          Terminal Font: Fira Code weight=450 1 
                        +MMd/NMh          CPU: AMD Ryzen 5 2500U with Radeon Ve 
                         mMm -mN`         GPU: AMD ATI Radeon Vega Series / Rad 
                         /MM  `h:         Memory: 2418MiB / 18898MiB 
                          dM`   .
                          :M-                                     
                           d:                                     
                           -+
                            -


```


## ⚙️ Installation

### Prior to the installation, I needed to update `apt` first:
```
sudo apt update
``` 

### Then using `nala` to install KVM:
```
sudo nala install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager -y
```

Installation complete.

## ⚙️ Post-Install Checks
### Checking CPU for KVM support:
```
┌─[kev@parrot]─[~]
└──╼ $egrep -c '(vmx|svm)' /proc/cpuinfo
8

```

It is supported. *(Any ≥ 1 means KVM is supported)*

### Next, checking the modules
```
┌─[kev@parrot]─[~]
└──╼ $lsmod | grep kvm
kvm_amd               217088  0
kvm                  1392640  1 kvm_amd
ccp                   155648  4 kvm_amd
irqbypass              12288  1 kvm
```
Let's break the output down line-by-line:

`kvm_amd 217088 0`
1. This is the AMD-specific KVM kernel module.
2. Size: ~217kB.
3. The last number (0) means currently no VMs are running using this module.
4. If there are active VMs, this would show the number of references.

`kvm 1392640 1 kvm_amd`
1. This is the core KVM module — the backbone of hardware virtualization.
2. Size: ~1.3MB.
3. The 1 means it’s currently being used by one other module (kvm_amd).
4. Basically: kvm provides the framework, and kvm_amd plugs in  CPU-specific virtualization features.

`ccp 155648 4 kvm_amd`
1. CCP = Cryptographic Coprocessor driver (on AMD CPUs).
2. Provides hardware acceleration for crypto operations, sometimes tied in with virtualization.
3. The 4 means four references are using this module, one of which is kvm_amd.

`irqbypass 12288 1 kvm`
1. This lets KVM bypass kernel interrupt handling to deliver interrupts more directly to VMs.
2. Improves performance for I/O heavy workloads.
3. The 1 means it’s used by kvm.

#### **Bottom line**
- KVM is installed, loaded, and working on the AMD CPU.
- The modules are in place, and the system is ready to run VMs with QEMU/KVM.
- No VMs are active right now (hence the zeros).

## ⚙️ Starting the libvirt daemon
```
┌─[kev@parrot]─[~]
└──╼ $sudo systemctl enable --now libvirtd
[sudo] password for kev: 
Synchronizing state of libvirtd.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable libvirtd
Use of uninitialized value $service in hash element at /usr/sbin/update-rc.d line 26, <DATA> line 44.
Use of uninitialized value $service in hash element at /usr/sbin/update-rc.d line 26, <DATA> line 44.
```

### Note the warning:
```
Use of uninitialized value $service in hash element at /usr/sbin/update-rc.d line 26, <DATA> line 44.
```
This comes from **update-rc.d**, which is part of the old SysV init system. Systemd tries to stay in sync with SysV scripts for compatibility.

On Parrot OS (Debian-based), `systemctl enable` calls into `systemd-sysv-install`, which in turn invokes `update-rc.d`. That script is written in Perl, and it’s trying to reference a variable `$service` that’s not properly set.

*It’s basically a **cosmetic bug** in Debian/Parrot’s packaging — it doesn’t break anything. The `libvirtd` was still enabled and started fine.*

### Let's see the **service status check**:
```
┌─[kev@parrot]─[~]
└──╼ $sudo systemctl status libvirtd
● libvirtd.service - Virtualization daemon
     Loaded: loaded (/lib/systemd/system/libvirtd.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-08-20 22:48:27 +07; 23s ago
TriggeredBy: ● libvirtd-admin.socket
             ● libvirtd.socket
             ● libvirtd-ro.socket
       Docs: man:libvirtd(8)
             https://libvirt.org
   Main PID: 257887 (libvirtd)
      Tasks: 19 (limit: 32768)
     Memory: 6.1M
        CPU: 343ms
     CGroup: /system.slice/libvirtd.service
             └─257887 /usr/sbin/libvirtd --timeout 120

Ogos 20 22:48:26 parrot systemd[1]: Starting libvirtd.service - Virtualization daemon...
Ogos 20 22:48:27 parrot systemd[1]: Started libvirtd.service - Virtualization daemon.
```
Output says:
- **Loaded**: yes, systemd recognizes the service file.
- **Enabled**: correct, it will now auto-start at boot.
- **Active (running)**: ✅ libvirtd is running, PID 257887.
- **Tasks**: 19 lightweight processes are being managed.
- **Memory**: 6.1 MB → pretty normal footprint.
- **CPU**: ~343ms → it’s idle and barely using resources.

The “TriggeredBy” sockets (`libvirtd.socket`, etc.) are systemd socket-activation units — libvirtd can be lazily started by connections if not running, but since I forced it with `--now`, it’s already up.

To be 100% sure everything sticks across reboots, let's try this:
```
systemctl is-enabled libvirtd
systemctl is-active libvirtd
```

First one says `enabled` and the second one says `inactive`, while it should says `active`. I wonder why...
- Turns out, I left it on idle, and it went inactive by itself, since `libvirtd` on modern distros often isn’t supposed to run as a *persistent daemon* anymore. Instead, it’s managed by socket activation:
	- The sockets (`libvirtd.socket`, `libvirtd-ro.socket`, `libvirtd-admin.socket`) listen in the background.
	- When a client (like `virsh` or `virt-manager`) connects, systemd starts `libvirtd` automatically.
	- If idle, systemd may stop it again.
- Sure enough, when I run `sudo virsh list --all`, the `libvirtd` became active again.

### Last step
Adding my user to `libvirt` group, so elevated privilege is not needed for `virt-manager` (*no need for sudo*).
```
sudo usermod -aG libvirt $(whoami)
newgrp libvirt
```
That's it for prerequisites install. 

In the next chapter, we will explore both the CLI and GUI for creating VMs.


*Thanks for reading!*

*Regards,*
*Kev.*