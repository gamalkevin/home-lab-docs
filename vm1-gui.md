# 📦️ VM1 - KVM with GUI & Ubuntu Server
## 📄 Preface

For the first VM, I'll be using GUI (`virt-manager`) just for the sake of familiarity and feature testing. The installed OS will also reflect the GUI-nature of this practice.

## 🧰 Spec Data

### VM: Testbench 1
- **Purpose:** Learning and familiarizing
- **OS:** Ubuntu Server 24.04
- **Software:** *TBC*
- **Network:** 
	+ **Mode:** NAT with port forwarding:9090
	+ **IP:** 192.168.122.130
- **Credentials:** 
	+ **Name:** `Warden`
	+ **Server Name:** `Shawshank`
	+ **Username:** `warden`
	+ **PW:** `W@rd3n`
- **Notes:** *TBC*
- **Created:** 21-08-2025 

## 🪜 Installation Steps
1. Launch `virt-manager`
2. Click **“Create a new virtual machine”**.
3. Steps in the wizard:
	1. **Local install media** → point to an ISO: `ubuntu-24.04.3-live-server-amd64.iso`.
	2. **Choose OS type/version** → `Ubuntu 22.04 LTS`. *For this, I had to untick `Automatically detect from the installation media/source`  since auto detection only supports up to **ubuntu-22.04**.*
	3. **Memory & CPUs** → `4096 MiB` of RAM, and `2` cores.	
	4. **Storage** → create a new disk image: `25 GiB`.
	5. **Network** → I left it at `Virtual network 'default': NAT`.
	6. I also checked the `Customize configuration before Install` to see what's available.
4. After taking a look at the pre-installation configuration window, I decided not to change it any further and just clicked `Begin Installation`.

## 🚀 Installer Launch:
1. The Ubuntu ISO that I'm using greeted me with an aptly minimalistic-looking GRUB. Perhaps this is the standard for enterprise-level OSes.
2. I chose the first entry: `Try or Install Ubuntu Server`
3. The Live session experience is unlike desktop Ubuntu; 
	1. For starters, it uses `Subiquity` installer (*compared to Calamares in desktop ISOs*). It doesn't have the 'beauty' of the usual desktop, only TUI (Text-based User Interface), though you get the blazing fast performance, even on VM, since it only uses a minimal resource to draw the text. 
	2. Arguably, it is still much easier to use compared to other more 'advanced' server OSes.
4. When asked to **"Choose the base for the installation"**, I chose the first option: `Ubuntu Server` and not the `Ubuntu Server (minimized)` as I'm still learning. I also checked `Search for third-party drivers`. I press `done`.
5. Next come the `Network configuration`. I will leave it as is, using the default config.
Installation started at `11:56` local time.
6. Then for `Proxy configuration`, I left it blank.
7. Afterwards, the `Ubuntu archive mirror configuration` page automatically detected my location and uses `http://th.archive.ubuntu.com/ubuntu/`. It also automatically tested the mirror.
8. For `Guided storage configuration`, I left it default: `Use an entire disk`, as it is contained in a VM anyway.
9. Here's my profile config:
	1. **Name**			: `Warden`
	2. **Server Name**	: `Shawshank`
	3. **Username**		: `warden`
	4. **PW**			: `W@rd3n`
10. I skipped the option to `Enable Ubuntu Pro`; might enable this later on to test it.
11. I opted to **install OpenSSH server** for testing remote access. Then I imported SSH keys using my GitHub account.
12. Next, I chose two `Featured server snaps` out of curiosity:
	1. `docker` Container runtime; and
	2. `powershell`
13. Afterwards, it's off to installation, starting at `21:23 UTC+7`.
14. Installation finished at `21:30 UTC+7`, continue to reboot.

## 💿️ First Boot
- First boot greeted me with CLI screen.
- Let's update `apt` first:
	+ `sudo apt update`, then 
	+ `sudo apt install nala` (*personal favorite*)
- Then update the system:
	+ `sudo nala upgrade`

### Using `cockpit`
I have yet to learn how to copy-paste terminal outputs from the VM (*for documentation purpose*), so I opted to use `cockpit` and access the VM using web browser from my host machine (*Parrot OS*).

#### Configuring network on the VM
- I need to **activate port forwarding** of the VM's networking.
	+ Unfortunately `virt-manager` doesn’t have a direct UI for forwarding ports, so I had to edit the XML directly using `virsh`:
		* `sudo virsh net-edit default`
		* First it asked which editor to use. I chose `nano`.
	+ Then I added **port redirection** so it looks like this:
	```
		<forward mode='nat'>
    		<port protocol='tcp' hostport='9090' guestport='9090'/>
  		</forward>
	```
	+ Then restart the network:
		* `sudo virsh net-destroy default`
		* `sudo virsh net-start default`

#### Checking `cockpit`
- After that, I check the VM to see if `cockpit` is running:
	+ `sudo systemctl status cockpit`
	+ In the output, it says the `cockpit.service` is `inactive (dead)`.
	+ However, the next line says:
	```
	TriggeredBy: • cockpit.socket
	```
	+ The dot behind `cockpit-socket` is green, indicating that it's listening.
	+ Initially I was confused as why the cockpit service is dead. Turns out, that is how **cockpit** works:
		* `cockpit.socket` is the one that actually listens on the port (usually 9090).
		* `cockpit.service` doesn’t run all the time. It only starts on demand when someone connects to the socket. That’s why it was loaded but dead. That’s totally normal!
- With that done, let's check if it's listening on port 9090:
	* `ss -tlnp | grep 9090`
	* Output showed `LISTEN 0 	4096 *:9090	*:*`
- Then I opened `https://<VM-IP>:9090` on my browser, and I can finally log in through it.
	* *Note: Check the VM ip using `ip a`*

## 🛠️ To-dos
- Create snapshot for 'clean baseline'
- Install tools
	+ `unzip`
	+ `net-tools`
	+ `btop` for monitoring
	+ 

