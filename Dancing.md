Introduction

There are multiple ways to transfer files between two hosts on the same network. One commonly used protocol for this purpose is SMB (Server Message Block). SMB allows shared access to files, printers, and other network resources and is most commonly found on Windows systems.

SMB services typically operate over TCP port 445. During enumeration, seeing this port open usually indicates that SMB is running on the target. SMB functions at the Application/Presentation layer of the OSI model and relies on lower-level protocols for transport.

In many environments, SMB uses NetBIOS over TCP/IP (NBT), which explains why scans often show multiple related services when enumerating a host.

An SMB-enabled storage location is called a share. Access to these shares normally requires valid credentials (username and password). However, due to misconfiguration, administrators may sometimes allow guest or anonymous access, which can expose sensitive files. This misconfiguration is what we exploit in this lab.

Enumeration

After connecting to the VPN, the first step was to scan the target system using nmap to identify open ports and services.

nmap -sV <target_ip>


From the scan results, port 445 (SMB) was found to be open, confirming that the target was running an SMB service and likely hosting network shares.

To enumerate SMB shares, we used the smbclient tool. If it is not installed, it can be installed on Debian-based systems using:

sudo apt install smbclient


Once installed, we listed available SMB shares on the target:

smbclient -L //<target_ip>


Since no username was specified, smbclient attempted authentication using the local system username and prompted for a password. To test for misconfiguration, we left the password blank, attempting anonymous or guest authentication.

Discovered Shares

The following shares were discovered:

ADMIN$ – Administrative share (restricted)

C$ – Administrative share for the C:\ drive

IPC$ – Inter-process communication share (not browsable)

WorkShares – Custom user-created share

The administrative shares returned NT_STATUS_ACCESS_DENIED, indicating restricted access. The WorkShares share appeared custom-made and was likely misconfigured.

Foothold

We attempted to connect to the WorkShares share without credentials:

smbclient //<target_ip>/WorkShares


This attempt was successful, confirming anonymous access was enabled. Upon connection, we entered the SMB interactive shell (smb: \>).

Useful SMB commands used:

ls – list directory contents

cd – change directory

get – download files

exit – exit SMB shell

 File Discovery

Inside the WorkShares share, two directories were found:

Amy.J

James.P

Navigating into Amy.J, a file named worknotes.txt was discovered and downloaded using:

get worknotes.txt


In the James.P directory, the flag.txt file was found and downloaded in the same manner.

Capture the Flag

After exiting the SMB shell, the downloaded files were reviewed locally.

worknotes.txt contained informational notes and hints.

flag.txt contained the required flag.

Submitting the contents of flag.txt successfully completed the machine.
