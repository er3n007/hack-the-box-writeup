Introduction

Databases are structured collections of data that can be easily accessed, managed, and updated. In modern environments, databases play a critical role by storing information related to transactions, inventory, customer details, and application data.

One such database technology is Redis, an in-memory key-value database. Unlike traditional databases that store data on disks or SSDs, Redis primarily stores data in RAM, allowing extremely fast data access and minimal response times. Because RAM is much faster than secondary storage, Redis is widely used for caching, session management, and temporary data storage.

For example, a web application may first query Redis to retrieve frequently requested data. If the data is not present, the application fetches it from a primary database (such as MySQL or MongoDB) and temporarily stores it in Redis for faster access in future requests.

This lab focuses on enumerating an exposed Redis server, interacting with it using the redis-cli utility, dumping its contents, and retrieving the flag.

 Enumeration
 Connectivity Check

To verify that the target system was reachable, an ICMP echo request was sent:

ping <target_ip>


After confirming successful responses, the ping command was stopped as connectivity was established.

 Port and Service Scanning

Next, an nmap scan was performed to identify open ports and running services:

nmap -sV <target_ip>


From the scan results, it was observed that:

Port 6379 was open

The service running on this port was Redis

This confirmed that a Redis server was exposed and accessible remotely.

 What is Redis?

Redis (REmote DIctionary Server) is an open-source NoSQL key-value data store commonly used as a database, cache, and message broker. Data is stored as key-value pairs and is optimized for fast read and write operations.

Redis stores data primarily in memory but periodically writes snapshots to disk to ensure persistence and recovery in case of failure.

🛠 Installing redis-cli

To interact with the Redis server, the redis-cli command-line utility is required. It was installed using:

sudo apt install redis-tools


Alternatively, Redis can also be accessed using tools like netcat, but redis-cli provides a more convenient and feature-rich interface.

 Enumerating the Redis Server

To view available options and usage of redis-cli, the help menu was displayed:

redis-cli --help


To connect to the remote Redis server, the following command was used:

redis-cli -h <target_ip>


Upon successful connection, a Redis prompt appeared, indicating direct interaction with the Redis server.

 Server Information

One of the most useful Redis enumeration commands is:

info


This command returns detailed information about the Redis server, including version details, memory usage, and database statistics.

From the Keyspace section of the output, it was observed that:

Only database 0 existed

The database contained stored keys

 Selecting the Database

Redis supports multiple logical databases. The default database (index 0) was selected using:

select 0

 Listing Keys

To list all keys present in the database, the following command was used:

keys *


This revealed the stored keys within the Redis database.

 Retrieving the Flag

After identifying the key containing the flag, its value was retrieved using:

get <key>


This command returned the flag value, successfully completing the challenge.
