# Make sure PiMox or Proxmox is up and working in your PI/PC

1. **Create a new network bridge in your proxmox by going in to the network settings (Example vmbr1)**

   
2. **Download openwrt ct image (rootfs.tar.xz) from https://images.linuxcontainers.org/images/openwrt**

   I chose https://images.linuxcontainers.org/images/openwrt/23.05/arm64/default/20240910_11%3A57/rootfs.tar.xz


3. **In your proxmox console use wget**

       wget https://images.linuxcontainers.org/images/openwrt/23.05/arm64/default/20240910_11%3A57/rootfs.tar.xz


4. **run pct command to create Container in your Proxmox shell and choose your CT ID and Storage accordinly**, (Example: 250 , Local or Local:lvm)

       pct create 250 ./rootfs.tar.xz --unprivileged 1 --ostype unmanaged --hostname openwrt --net0 name=eth0 --net1 name=eth1 --storage local


5. **nano into /etc/pve/lxc/xxx.conf**

       nano /etc/pve/lxc/250.conf

**add two lines at the bottom**

    lxc.cgroup2.devices.allow: c 10:200 rwm
    lxc.mount.entry: /dev/net dev/net none bind,create=dir


**exit from nano and run this command in Proxmox Shell**

    chown 100000:100000 /dev/net/tun


7. **find your newly created CT and go to network settings**

    edit **both network devices** and assign them a bridge (Example **eth0-vmbr0** and **eth1-vmbr1**)

## NOW YOU CAN START YOUR OPENWRT CT

***If you are using Raspberry pi like me the console will not work for this CT***

go back to your **Proxmox Shell** and type **"pct enter (vmid)"**

**Example:**

    pct enter 250

**You are now inside the openwrt ct shell**

**run command:**

    vi /etc/config/firewall

type **"i"** to enter **edit mode**

go the bottom and add these lines

    config rule                                             
            option src              wan                            
            option dest_port        80                             
            option proto    tcp                                                              
            option target   ACCEPT 
                                 
 
 press **"ESC"** to exit **"edit mode"**
 
 type **:wq** the press **Enter** to exit

 then type **reboot** and **Enter** to reboot

again enter the CT by using **"pct enter 250"**

type **"ip a"** to know your **local ip**

use your web browser to access **Luci Interface**

No password just hit **"login"** and **set a password**

In Luci Go to **Network>Interfaces**
 
(Network configuration migration, The existing network configuration needs to be changed for LuCI to function properly)

Press **"continue"**

***If you use **IPV6** then you can keep **"wan6"** else delete it and only keep "wan"***


Add New Interface named LAN **"lan"** with device **"eth1"** with static ip of your choice different than your existing subnet (example: 10.10.0.1)

and a subnet mask

**save >> save and apply**


info: ***_Now if you assign this new bridge **"vmbr1"**  to a different vm/ct then it will get the ip address of openwrt dhcp_***

in **dhcp server** option go to **Advanced Setting** and enable **Dynamic DHCP**

in **Firewall** setting assign a firewall - choose **lan** if not already selected


    ping google to check if ping is working as It should

and

    curl ifconfig.me to check you public ip of your default ISP

 ### AT THIS POINT OF TIME IF YOU JUST WANNA USE OPENWRT THE YOU ARE DONE AND YOU CAN CONFIGURE YOUR OPENWRT AS PER YOUR LIKINGS.

## CONTINUE IF YOU WANT TO ROUTE EVERYTHING USING AN OPENVPN

For this demo I am using free proton vpn config


in **Luci web interface** go to **System>>Software**
**Actions: Update lists**

**Filter >> type openvpn**

**install:**

    openvpn-openssl
    luci-app-openvpn

refresh **Luci Web Interface** and you will a new option at the top called **"VPN"**



got to **VPN>>OPENVPN**

Use **"OVPN configuration file upload"** for **.ovpn** files

give it a name in **"instance name"** (Example: JP)

click **"choose file"** and **"upload"** your .ovpn file

Edit the newly saved instance 



put **username and password** of your .ovpn file at the **bottom box**

Inside "Section to add an optional 'auth-user-pass' file with your credentials (/etc/openvpn/JP.auth)"


find a line **"auth-user-pass"** in the top box 

add **"/etc/openvpn/{instance name}.auth"** after **"auth-user-pass"**

example: 

    auth-user-pass /etc/openvpn/JP.auth


**remove** these lines from **above box** if you these 

    script-security 2
    up /etc/openvpn/update-resolv-conf
    down /etc/openvpn/update-resolv-conf "


 hit **"Save"** and go back and click at the top on **"VPN>>OPENVPN**
 again click on **edit** to confirm if the **changes are saved**


 **Start your instance**
 
 
 Now go to **Network>>Interfaces>>Devices** in Luci Web Interface

 Check if **"tun0"** device is showing up

 ***If "Yes" then proceed further***      
 
***_If "No" then Reboot your CT and it will fix it and start openvpn automatically_***

 Add a **New Interface** name it **"tun0"** , protocol> select **"DHCP Client"**, Device> select **"tun0"**
 
 go to **tun0 firewall** and assign firewall zone to **"wan"** the **"save"** and **"save and apply"**

 
