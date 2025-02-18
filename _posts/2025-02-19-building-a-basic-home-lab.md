# Building a Basic Home Lab
After learning the theory on penetration testing or malware analysis, our minds want to dive into the action right away. To test new tools or to observe a malware characteristics, we need a seperate system where we can break things however we want. If something goes kaboom, I mean if something goes out of control, it should remain contained and our main workstation mustn't affected by that. So what do we need for that? Ummm.....

We need to apply virtualization. You heard right!! Virtualization basically refers to creating virtual environments using physical resources. In a simple term, machine into a machine :)

To install virtual environments, there are several softwares that we can use such as VMware, Virtualbox, Hyper-V etc. I am going to use Oracle Virtualbox. First download virtualbox from their official website: [Virtualbox Download](https://www.virtualbox.org/wiki/Downloads)

So what am I going to do here??

- After installing Virtualbox, We are going to create a Windows 10 virtual environment.
- [Download](https://www.microsoft.com/en-us/software-download/windows10) Windows ISO file from Official microsoft website
- Create Windows 10 virtual environment
- [Download](https://www.kali.org/get-kali/) Kali Linux from their official website.
- Create Kali virtual environment
- Configure the internal network between the environments so that they can interact with each other but separated from the network.

The last step is crucial if you're analyzing a malware in a system. If you want to test tools, then default network configuarations are fine.

### Installing Windows 10

- Setup virtual machine destination location
- Setup ISO file location
- Configure RAM and Hard disk according to system condition
- At least give 60GB for the hard disk space
- To skip unattended installation, fill the checkmark below. You have to install manually if you do this.

![Add iso file](https://raw.githubusercontent.com/ImdadMiran17/Building-Basic-Home-Lab/refs/heads/main/Screenshot%20(339).png)

![Set RAM Size](https://raw.githubusercontent.com/ImdadMiran17/Building-Basic-Home-Lab/refs/heads/main/Screenshot%20(340).png)

![Set Hard disk size](https://raw.githubusercontent.com/ImdadMiran17/Building-Basic-Home-Lab/refs/heads/main/Screenshot%20(341).png)


### Installing Kali Linux

- You can install kali from ISO file or from prebuilt vmware or virtualbox file. With the later, you don't have to do the manual processes again. You can just simply import the file and create the machine. I used the virtualbox file.
- Download the provided .7z file. Extract and import it to virtualbox.

When done with configuring the physical resources of the virtual environments, then it's just OS installation. If you having a problem with that, search some googling or youtubing. Do some mad scientist stuff :|


### Configuring Network

Let's dive into the fun stuff now. Select windows virtual machine in virtualbox and go to settings. Then select network tab. We have several types of networking options here.

- Not attached: Not attached to any network, not connected to internet, completely isolated.
- Network Address Translation (NAT): Default option. Connected to internet. Each host creates it's own virtual network, not connected to each other.
- NAT Network: Connected to internet. Creates one network for all the hosts.
- Bridged networking: Virtual hosts share the same network as physical host.
- Internal networking: This can be used to create a different kind of software-based network which is visible to selected virtual machines, but not to applications running on the host or to the outside world.
- Host-only networking: This can be used to create a network containing the host and a set of virtual machines, without the need for the host's physical network interface.
- Cloud networking: This can be used to connect a local VM to a subnet on a remote cloud service.
- Generic networking: Rarely used modes which share the same generic network interface, by allowing the user to select a driver which can be included with Oracle VirtualBox or be distributed in an extension pack. 

We will use internal networking as it's not connected to internet and an internal network can be created among the hosts. We can also use Not Attached for malware analysis as it is totally isolated. That said, it is totally harmless for the actual host.

In the network settings, select internal network and give a name to it. Let's say, imnetwork. Then click Ok. When a internal network is created, every virtual machine can use that network and can be a part of it. Configure the network settings of kali machine same way. Now we have to manually set the static ip address.


![](https://raw.githubusercontent.com/ImdadMiran17/Building-Basic-Home-Lab/refs/heads/main/Screenshot%20(342).png)


To set the ip address of the windows machine, we first go to taskbar. Then look for the globe icon there. Right click on the globe icon and when the menu opens, select "Open Network Settings". From the settings page, select **Change Adapter** option. It will open network adapter list. Right click on the ethernet and select properties. Now, select IPv4 and click on properties again. New dialogue box is opened now where you should choose the manual option for IP address. Let's set the ip address to ```192.168.100.10``` and subnet mask ```/24``` which translates to ```255.255.255.0```.


![](https://raw.githubusercontent.com/ImdadMiran17/Building-Basic-Home-Lab/refs/heads/main/Screenshot%20(1).png)


To set the ip address of the Kali machine, again we first go to taskbar and click on ethernet icon located at the top right corner. Click on the icon and select "Edit Connections". It will open existing connections on the host which is only one right now. Double click on Wired Connection 1 or click on the gear icon. Then Click on IPv4 Settings. It's set to automatic(DHCP) now. Change it to manual. Click on Add button then. Let's set the ip address to ```192.168.100.11``` and subnet mask ```/24``` which translates to ```255.255.255.0```.


![](https://raw.githubusercontent.com/ImdadMiran17/Building-Basic-Home-Lab/refs/heads/main/Screenshot_2025-02-16_22-55-06.png)


We can test now that our ip addresses are properly configured. In windows, open cmd and run the command ```ipconfig```. In kali, open terminal and run the command ```ifconfig```. It's properly configured now. Phew!!


![](https://raw.githubusercontent.com/ImdadMiran17/Building-Basic-Home-Lab/refs/heads/main/Screenshot%20(2).png)

![](https://raw.githubusercontent.com/ImdadMiran17/Building-Basic-Home-Lab/refs/heads/main/Screenshot_2025-02-16_22-57-24.png)


Let's check our connectivity between the virtual machines. If we run the ```ping``` command from kali against the windows machine, windows firewall will drop icmp packets. So we can run the ```ping``` command from the windows machine. That way, we can check the connectivity without the hassle. So let's run ```ping 192.168.100.11```.


![](https://github.com/ImdadMiran17/Building-Basic-Home-Lab/blob/main/Screenshot%20(3).png)


We can see that there is no packet loss. The machines are connected in the network now. ```Happy Hacking!!```















