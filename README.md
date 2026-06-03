# Personal Nextcloud Instance using the Snap Package
This is a documentation about how I created a personal Nextcloud instance using their Snap package, hosted in Orcale Cloud free tier.

## Prerequisites
### System Requirements
- **Operating System**: Ubuntu (Server or other versions) _only_.
- **CPU**: 64-bit CPU that supports Ubuntu.
- **Memory**:
	- Minimum requirements for Ubuntu.
	- 128 MB (minimum) or 512 MB (suggested) per process for Nextcloud.
- **Storage**:
	- Minimum requirements for Ubuntu.
	- As per user's requirement of cloud storage.

### Orcale Cloud Account
- Created a Free Oracle Cloud account with it's requirements as documented [here](https://docs.oracle.com/en-us/iaas/Content/GSG/Tasks/signingup_topic-Sign_Up_for_Free_Oracle_Cloud_Promotion.htm).

- The following resources offered by Oracle Cloud free tier are a great fit for this project:
	- 2 CPU cores
	- 24 GB memory
	- 200 GB storage split into 2 blocks

	Refer to [their website](https://www.oracle.com/cloud/free/) for more details.

- Selected the preferred region from the Oracle Cloud web interface.

## Creating Nextcloud Instance
The sections below describe different stages of creating the Nextcloud instance in sequence as required.

### Create Compute Instance
- Followed the [steps to create an Ubuntu instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) with the desired specifications, that fit within free tier.
- Established SSH connection with the instance using [these steps](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/accessinginstance.htm).

### Preparing Ubuntu Server
#### Full System Update
- Update APT repository.
	```sh
	sudo apt update -y
	```
- Upgrade all system packages.
	```sh
	sudo apt dist-upgrade -y
	```
	Or,
	```sh
	sudo apt upgrade -y
	```

> [!NOTE]  
> Difference between _`upgrade`_ and _`dist-upgrade`_:
> - `upgrade` only upgrades the versions of installed packages but NEVER uninstalls or removes anything.  
> - `dist-upgrade` upgrades everything in the system, cleans up older versions and their dependencies, and upgrades the OS and kernel versions.  
>   
> Basically, use `upgrade` for only upgrading the installed packages, use `dist-upgrade` to upgrade everything (including OS and kernel) to newer versions and remove old ones.  

#### Hardening Security
- Create a **non-root user** for administration with `sudo` privileges.
	- Creating the user.
		```sh
		useradd <nextcloud-admin-username>
		```
	- Providing `sudo` privileges.
		```sh
		usermod -aG sudo <nextcloud-admin-username>
		```
- Establish a new SSH connection for the Nextcloud admin user.
	- Generating SSH key-pair in local machine.
		```sh
		ssh-keygen
		```
	- Following steps to [create local connection to instance](https://docs.oracle.com/en-us/iaas/Content/Compute/References/serialconsole.htm#creating-instance-connection-local).
	- Login using the new SSH key
		```sh
		ssh -i /path/to/private-key-file <nextcloud-admin-username>@<instance-public-IP-addr>
		```
- Tighten up SSH access
	- Open the file "**/etc/ssh/sshd_config**" for editing with `sudo` privilege.
		```sh
		sudo nano /etc/ssh/sshd_config # use your preferred text editor
		```
	- Change SSH port to custom port.
		```sh
		# Port 22 # leave original entry as comment
		Port <custom-port>
		```
	- Disable root login.
		```sh
		# PermitRootLogin <old-value> # leave original entry as comment
		PermitRootLogin no
		```
	- Disable password login
		```sh
		# PasswordAuthentication yes # leave original entry as comment
		PasswordAuthentication no
		```
	- Save the edits made to the file `/etc/ssh/sshd_config`.
	- Restart the SSH service daemon
		```sh
		sudo systemctl restart sshd.service
		```
	- _Without disconnecting current SSH connection_, test the configuration chages through a new session
		```sh
		ssh -i /path/to/private-key-file <nextcloud-admin-username>@<instance-public-IP-addr> # should fail to connect
		ssh -i /path/to/private-key-file -p <custom-SSH-port> <instance-public-IP-addr> # should connect
		```

> [!WARNING]  
> If the SSH configurations are somehow incorrect, you can <font style="color: crimson; font-weight: bold; font-style: italic">permanently lose access</font> to the instance.  
> In case of lost access to compute instance, either create new instance, or consult the Oracle Cloud support and forums for workarounds.  

- Configure firewall for limiting access

- [Optional] Enable automatic updates