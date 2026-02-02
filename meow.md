Step 1: Connecting to Hack The Box VPN

First, I downloaded and installed the Hack The Box VPN configuration file.
After successfully connecting to the VPN, I was able to communicate with the target machine and obtained its IP address.

Step 2: Checking Connectivity

To confirm that the target machine was reachable, I used the ping command:

ping <target_ip>


The machine responded, which confirmed that the target was online and accessible.

Step 3: Port Scanning with Nmap

Next, I performed a service scan using Nmap to identify open ports and running services:

nmap -sV <target_ip>

Result:

Port 23 was open

The service running on this port was Telnet

This indicated that the machine was allowing Telnet connections, which is an insecure protocol.

Step 4: Logging in via Telnet

Since Telnet was open, I attempted to log in using common default credentials.

telnet <target_ip>


When prompted for login credentials:

Username: root

Password: (left blank)

The login was successful, and I gained access to the system as the root user.

Step 5: Finding the Flag

After gaining access, I listed the files in the directory:

ls


I found a file named flag.txt.
To read the flag, I used:

cat flag.txt


This displayed the flag, completing the challenge successfully.
