Step 1: Connectivity Check

First, I verified connectivity to the target by sending ICMP echo requests.

ping <target_ip>


The target responded successfully, confirming it was reachable.

Step 2: Service Enumeration

I performed a scan to identify open ports and services running on the target.

nmap -sC -sV <target_ip>


From the scan results:

FTP service was running on port 21

FTP version identified was vsFTPd 3.0.3

The operating system detected was Linux

 Step 3: Connecting to FTP

I connected to the FTP service using the FTP client.

ftp <target_ip>

Step 4: Anonymous Login

To test for anonymous access, I logged in using:

Username: anonymous
Password: anonymous


The server returned the response:

230 Login successful

Step 5: Listing Files

After logging in, I listed the files and directories available on the FTP server using:

ls


or

dir

Step 6: Downloading Files

I downloaded the discovered file from the FTP server using:

get <filename>


The file was successfully downloaded for further analysis.# hack-the-box-writeup
