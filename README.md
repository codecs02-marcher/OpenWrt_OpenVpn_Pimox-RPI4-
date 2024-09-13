# Make sure PiMox arm 64 or Proxmox is up and working in your PI/PC 

1. **Create a new network bridge in your proxmox by going in to the network settings (Example vmbr1)**

   
    
![Capture0](https://github.com/user-attachments/assets/6b56c493-9aa8-46d8-bfad-389bea69e61d)


   
2. **Download openwrt ct image (rootfs.tar.xz) from https://images.linuxcontainers.org/images/openwrt**

   I chose https://images.linuxcontainers.org/images/openwrt/23.05/arm64/default/20240910_11%3A57/rootfs.tar.xz


   
3. **In your proxmox console use wget**

       wget https://images.linuxcontainers.org/images/openwrt/23.05/arm64/default/20240910_11%3A57/rootfs.tar.xz

  
4. **run pct command to create Container in your Proxmox shell and choose your CT ID and Storage accordinly**, (Example: 250 , Local or Local:lvm)

       pct create 250 ./rootfs.tar.xz --unprivileged 1 --ostype unmanaged --hostname openwrt --net0 name=eth0 --net1 name=eth1 --storage local


    ![Capture1](https://github.com/user-attachments/assets/78655414-cdf2-4496-bee2-da255465c4c4)



6. **nano into /etc/pve/lxc/xxx.conf**

       nano /etc/pve/lxc/250.conf
   
   ![Capture2](https://github.com/user-attachments/assets/7b39fee6-4c9d-48b0-ac31-262494e8e369)


**add two lines at the bottom**

    lxc.cgroup2.devices.allow: c 10:200 rwm
    lxc.mount.entry: /dev/net dev/net none bind,create=dir


**exit from nano and run this command in Proxmox Shell**

    chown 100000:100000 /dev/net/tun


6. **find your newly created CT and go to network settings**

    edit **both network devices** and assign them a bridge (Example **eth0-vmbr0** and **eth1-vmbr1**)

   
  ![Capture4](https://github.com/user-attachments/assets/04d8ed4f-fe4e-4c7c-8561-a9c475ae514b)
  
  ![Capture5](https://github.com/user-attachments/assets/15f040bc-2e97-4874-8c78-58844b993bae)



   
## NOW YOU CAN START YOUR OPENWRT CT

***If you are using Raspberry pi like me the console will not work for this CT***

7. go back to your **Proxmox Shell** and type **"pct enter (vmid)"**

**Example:**

    pct enter 250

    
![Capture6](https://github.com/user-attachments/assets/fbd1506e-8ce6-418c-84c8-3df25023ba1a)




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


  ![Capture7](https://github.com/user-attachments/assets/8d7b613d-f8a2-4f94-835e-1420292fb685)


     
 then type **reboot** and **Enter** to reboot

again enter the CT by using **"pct enter 250"**

type **"ip a"** to know your **local ip**


 ![Capture8](https://github.com/user-attachments/assets/89d0e7ac-b2e1-4c7f-bd3b-b03f4a2353f7)



8. use your web browser to access **Luci Interface**

No password just hit **"login"** and **set a password**

In Luci Go to **Network>Interfaces**
 
(Network configuration migration, The existing network configuration needs to be changed for LuCI to function properly)

Press **"continue"**


  ![Capture11](https://github.com/user-attachments/assets/3317433d-e5ee-4f6e-82f7-feb9f0a83c3c)



***If you use **IPV6** then you can keep **"wan6"** else delete it and only keep "wan"***


Add New Interface named LAN **"lan"** with device **"eth1"** with static ip of your choice different than your existing subnet (example: 10.10.0.1)

and a subnet mask

**save >> save and apply**


  ![Capture13](https://github.com/user-attachments/assets/24e4107d-aacb-4388-a188-a5b2cf60ff55)

     

info: ***_Now if you assign this new bridge **"vmbr1"**  to a different vm/ct then it will get the ip address of openwrt dhcp_***


![Capture14](https://github.com/user-attachments/assets/56add3b4-0db5-444d-9c02-19cee0cd4cf2)



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


![Capture17](https://github.com/user-attachments/assets/ad87d19d-33ff-46f8-8908-351186da8dc5)



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


![Capture19](https://github.com/user-attachments/assets/9ad00456-a662-49bd-b6ea-26ae98671a94)



 hit **"Save"** and go back and click at the top on **"VPN>>OPENVPN**
 again click on **edit** to confirm if the **changes are saved**


![Capture20](https://github.com/user-attachments/assets/4541996a-5a0c-43c6-8686-0655bdaa202e)



 **Start your instance**
 
 
 Now go to **Network>>Interfaces>>Devices** in Luci Web Interface

 Check if **"tun0"** device is showing up


![Capture21](https://github.com/user-attachments/assets/ecec8154-9e42-4f35-8b19-5b9050f09c01)

 

 ***If "Yes" then proceed further***      
 
***_If "No" then Reboot your CT and it will fix it and start openvpn automatically_***

 Add a **New Interface** name it **"tun0"** , protocol> select **"DHCP Client"**, Device> select **"tun0"**
 

![Capture22](https://github.com/user-attachments/assets/110eea4e-0a42-4b57-bf73-4aa61bfc6605)


![Capture23](https://github.com/user-attachments/assets/48c1eabb-64ff-4577-ad38-dd80d43f6c67)
 
 
 go to **tun0 firewall** and assign firewall zone to **"wan"** the **"save"** and **"save and apply"**

 Check your **other VM** assigned **"vmbr1"**

 **run:**
     curl ifconfig.me

**to check vpn public ip**


![Capture24](https://github.com/user-attachments/assets/99bf0aa3-78c7-4808-ba8c-cd3d4a70f158)

