

CSUS, College of Engineering and Computer Science

**Department of Computer Science**

**CSC 154 - Computer System Attacks and Countermeasures**

<center>
    <h1>
        Lab - IRC Botnet
    </h1>
</center>

**Goal:** To develop and fully understand the working mechanisms of basic IRC botnets.

**Instructions:** Please refer to attached lab instructions within this document.

**Deliverable:** To be filled by Dr. Dai.

**Requirement:** To be filled by Dr. Dai.

------

[TOC]

------

# 1. Lab Overview

The learning objective of this lab is for students to gain first-hand experience regarding socket developing and managing a small-sized botnet by putting what they have learned about sockets and botnets from class into action. Throughout the lab, students will gain an understanding of  the following topics about how ...

- Important it is to debug and read RFCs
- IRC, and socket programming in general, works
- IRC is used to send commands to botnet slaves
- IRC is used to receive information from botnet slaves
- Easy it is to detect IRC-based botnets

​	In this lab, students will create an IRC-based botnet from scratch using [Kali Linux](https://www.kali.org/) as the operating system, [Python 3.x](https://www.python.org/) (3.7.2 at the time of this writing) as the programming language, [InspIRCd](http://www.inspircd.org/) as the IRC server, and [WeeChat](https://weechat.org/) as the IRC client to manage the botnet. 



# 2. Prerequisites

One Kali Linux VM capable of connecting to the internet is enough to demonstrate the botnet's capability, however multiple VMs may be used. Please note that port forwarding is required, but not covered in this lab, in order to connect to the botnet server (Kali Linux VM) from a different VM, hence the need for only one Kali Linux VM.



# 3. Lab Tasks

Log into the Kali Linux VM and connect to the internet.



## 3.1. Install WeeChat, InspIRCd

Once connected to the internet, WeeChat and InspIRCd can be installed using `apt-get` in a terminal with the following command:

```bash
sudo apt-get install inspircd weechat
```

![1550175572363](C:\Users\Sason\AppData\Roaming\Typora\typora-user-images\1550175572363.png)

Verify InspIRCd was download and installed with the following command: `service inspircd status`. The output should show the service's location and inactive status.

![1550175544870](C:\Users\Sason\AppData\Roaming\Typora\typora-user-images\1550175544870.png)



## 3.2. Configure InspIRCd

The IRC server configuration file is located at */etc/inspircd/inspircd.conf*. The majority of the config file will be left alone to keep the lab as basic and bug-free as possible.

![1550176320512](C:\Users\Sason\AppData\Roaming\Typora\typora-user-images\1550176320512.png)

Edit the file by typing `vim /etc/inspircd/inspircd.conf` and make the same changes as the screenshot below. If `vim` isn't your editor of choice, feel free to use `gedit` with the command `gedit /etc/inspircd/inspircd.conf`.

Notable configuration changes:

- Line 6: Change server name from "irc.local" to "irc.botnet.local"
- Line 7: Change server description from "Local IRC Server" to "Local IRC Botnet Server"
- Line 10: Change admin name from "Root Penguin" to "Botnet Master"
- **Line 11: Change admin nick from "Nick" to "Master"** (this is the most important and necessary config change)

![1550176540323](C:\Users\Sason\AppData\Roaming\Typora\typora-user-images\1550176717964.png)



## 3.3. Start InspIRCd (IRC server)

The IRC server is now properly configured and can be started by typing the following command in a terminal: `service inspircd start`. Verify the IRC server was launched with `service inspircd status` and notice the status is "active (running)".

![1550177288133](C:\Users\Sason\AppData\Roaming\Typora\typora-user-images\1550177288133.png)



## 3.3. Verify InspIRCd configuration with WeeChat

Launch WeeChat by typing `weechat` in a terminal. Use the following steps to connect to the IRC server and verify it works:

1. Type `/server add botnet localhost` and press enter to add the botnet server to your server list.
2. Type `/connect botnet` and press enter to join IRC server.
3. Type `/nick Master` and press enter to become the privileged IRC admin.



## 3.4. Program the Botnet





# Common Errors

