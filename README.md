# LAB 01: DEPLOY WEB APPLICATION ON INFRASTRUCTURE AS A SERVICE
## Authors
 * Emmanuel Schmid
 * Wojciech Myszkorowski

## Informations
* IAM User: Administrator
* Compte: 1652-1422-3307
* key name: cloud-key-pair-euwest3 
* group name: cloud_SG_erwest3
* mysql root password: toortoor

## Table of Contents
1. [TASK 1: SET UP](#TASK-1:-SET-UP)
2. [TASK 2: CREATE AN AMAZON EC2 INSTANCE](#TASK-2:-CREATE-AN-AMAZON-EC2-INSTANCE)
3. [TASK 3: INSTALL A WEB APPLICATION](#TASK-3:-INSTALL-A-WEB-APPLICATION)
4. [TASK 4: CREATE AN ADDITIONAL EBS VOLUME AND USE SNAPSHOTS](#TASK-4:-CREATE-AN-ADDITIONAL-EBS-VOLUME-AND-USE-SNAPSHOTS)
5. [TASK 5: PERFORMANCE ANALYSIS OF YOUR INSTANCE (OPTIONAL)](#TASK-5:-PERFORMANCE-ANALYSIS-OF-YOUR-INSTANCE-(OPTIONAL))
6. [TASK 6: RESOURCE CONSUMPTION AND PRICING](#TASK-6:-RESOURCE-CONSUMPTION-AND-PRICING)


## TASK 1: SET UP
###Available regions
* USA Est (Virginie du Nord)
* USA Est (Ohio)
* USA Ouest (Californie du Nord)
* USA Ouest (Oregon)
* Asie Pacifique (Mumbai)
* Asie Pacifique (Séoul)
* Asie Pacifique (Singapour)
* Asie Pacifique (Sydney)
* Asie Pacifique (Tokyo)
* Canada (Centre)
* UE (Francfort)
* UE (Irlande)
* UE (Londres)
* UE (Paris)
* Amérique du Sud (São Paulo)

The closest regions from Yverdon-les-bains is **Paris** 


## TASK 2: CREATE AN AMAZON EC2 INSTANCE
###What is the smallest and the biggest instance type (in terms of virtual CPUs and memory) that you can choose from when creating an instance?   
<center>
| |     Smallest	|	Biggest		|
| :--- |:---: | :---: |
| **Instance** |	t2.nano		|	x1.32xlarge	|
| **vCPU**	|	1	|	128	|
| **RAM**	|	0.5	|	1952	|
| **Stockage**	|	EBS uniquement	|	2 x 1920 (SSD)	|

</center>


###How long did it take for the new instance to get into the running state? 
* It takes **about two minutes** to prepare the vm.

###From the EC2 Management Console copy the public DNS name of the instance into the report.
* ec2-52-47-149-170.eu-west-3.compute.amazonaws.com

### Once you have successfully logged into your EC2 instance, run the hostname and uname -a commands and paste their outputs into the report.
* hostname output
```
ip-172-31-36-77
```
* uname -a output
```
Linux ip-172-31-36-77 4.4.0-1049-aws #58-Ubuntu SMP Fri Jan 12 23:17:09 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux

```

### Try to ping the instance from your local machine. What do you see? Explain. Change the configuration to make it work. Ping the instance, record 5 round-trip times.
```
--- 52.47.149.170 ping statistics ---
20 packets transmitted, 0 received, 100% packet loss, time 19451ms
```
The security group does not allow ping ipv4 (icmp) so you must add a rule in the security group to allow pings.

```
--- 52.47.149.170 ping statistics ---
7 packets transmitted, 7 received, 0% packet loss, time 6008ms
rtt min/avg/max/mdev = 33.740/35.333/36.248/0.805 ms
```

### Determine the IP address seen by the operating system in the EC2 instance by running the ifconfig command. What type of address is it? Compare it to the address displayed by the ping command earlier. How do you explain that you can successfully communicate with the machine?

```
eth0      Link encap:Ethernet  HWaddr 0e:51:12:de:a6:2e  
          inet addr:172.31.36.77  Bcast:172.31.47.255  Mask:255.255.240.0
          inet6 addr: fe80::c51:12ff:fede:a62e/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
          RX packets:589 errors:0 dropped:0 overruns:0 frame:0
          TX packets:555 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:267302 (267.3 KB)  TX bytes:60734 (60.7 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:192 errors:0 dropped:0 overruns:0 frame:0
          TX packets:192 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:14456 (14.4 KB)  TX bytes:14456 (14.4 KB)
```

After inspection with ```ifconfig```, we founded that the address IP do not match with the public IP. The IP address **172.31.36.77** is a local address IP.
Communcation with the machine is possible because a remaping of IPs addresses is performed. This technique is call : **NAT**.



## TASK 3: INSTALL A WEB APPLICATION
### Add a screenshot of the page you created in Drupal to the report.

![alt text](./ownDrupalPage.png)

### Add the Elastic IP Address you created to the report.

The adress IP is **52.47.174.97**.

### Why is it a good idea to create an Elastic IP Address for a web site (our web application)? Why is it not sufficient to hand out as URL for the web site the public DNS name of the instance?
An Elastic IP Address is a static and public IPv4 address which can be associated to an instance. It allows a instance (web application) to be reachable from internet by a unique IP address.  
Immediatly after stopping/terminating the instance using the service, the Elastic IP address can be associted again. It means that is possible to mask the failure by quickly remapping the address to another instance in your account.  



## TASK 4: CREATE AN ADDITIONAL EBS VOLUME AND USE SNAPSHOTS
* Examine the existing EBS Volume. In the EC2 Console navigate to Instances and select our Instance. In the details view click on Description and look for Root device. Click on the link and click on the EBS ID. This will open the EBS Volume in the console.
	* What is the size of the EBS Volume? **8 Gb**
	* Inside the Instance examine the mounted root volume with df -h /. What capacity does the operating system see and how much space is left?
```
ubuntu@ip-172-31-36-77:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1      7.7G  1.6G  6.1G  21% /
```

### Copy the Availability Zone of your Instance and Volume into the report.
The availability zone is **eu-west-3c**.

### Copy the available space after formatting and mounting into the report
```
ubuntu@ip-172-31-36-77:~$ df -h /mnt/disk
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvdf       976M  1.3M  908M   1% /mnt/disk

```



## TASK 5: PERFORMANCE ANALYSIS OF YOUR INSTANCE (OPTIONAL)

### Provide the URLs of the Geekbench results for the EC2 instance and your local machine.
|	Geekbench results	|    URL	|
|:---| :--- |
| **Instance**	|	<http://browser.primatelabs.com/geekbench3/8564057>	|
| **Local**	|	<http://browser.primatelabs.com/geekbench3/8564061>	|


### Provide system information about the EC2 instance.

| 		|    Instance	|	Local	|
| :--- |:--- | :--- |
| **Operating System**   | Ubuntu 16.04.3 LTS 4.4.0-1049-aws x86_64 | Ubuntu 16.04.3 LTS 4.13.0-36-generic x86_64|
| **Model**   | Xen HVM domU | LENOVO 20ARS0MG00  |
| **Processor**  | Intel Xeon E5-2676 v3 @ 2.39 GHz processor | Intel Core i7-4600U @ 3.30 GHz 1 processor, 2 cores, 4 threads  |
| **Processor ID**   | GenuineIntel Family 6 Model 63 Stepping 2 | GenuineIntel Family 6 Model 69 Stepping 1 |
| **L1 Instruction Cache**   | 32 KB | 32 KB x 2 |
| **L1 Data Cache**	| 32 KB | 32 KB x 2 |
| **L2 Cache**   | 256 KB | 256 KB x 2 |
| **L3 Cache**   | 30720 KB | 	4096 KB |
| **Motherboard**   | N/A | LENOVO 20ARS0MG00 |
| **BIOS**   | Xen 4.2.amazon | LENOVO GJET90WW (2.40 )  |
| **Memory**   | 	990 MB | 7667 MB |

### Provide the single-core and multi-core performance scores for overall, integer, floating-point and memory performance of the EC2 instance.

#### Single-core
Although the local machine processor has a higher frequency, the overall performance score is relatively close.  
|	Performance	|     Instance	|   Local	|
| :--- |:---: | :---: |
| Overall	|	2708	|      2973	|
| Integer	|	2754	|      3154	|
| Floating Point	|	2551	|      2857	|
| Memory	|	2934	|      2845	|

The processors used by Amazon are reserved for a server use. Even the bios have been customized, Amazon operates the equipment 100%!  

#### Multi-core
The instance type used **t2.micro** has only one virtual processor(=core).  
|	Performance	|     Instance	|   Local	|
| :--- |:---: | :---: |
| Overall      	|	2680	|      5016	|
| Integer      	|	2723	|      5793	|
| Floating Point	|	2537	|      5258	|
| Memory	|	2883	|      2981	|

The multi-core of the local machine makes the difference. It is important to define the context of use to properly size resources.  


## TASK 6: RESOURCE CONSUMPTION AND PRICING

### How much does your instance (including disk) cost per hour? What was its cost for this lab?
* Cost per hour = **0.0116$**
* Total cost = 0.0116$ * 3h = **0.0348$**

### Change the parameters to an instance that runs continuously during the whole month. Note the total cost.

The cost for a whole month is **9.67$** per month


### If you buy a 1 TB SSD disk at Digitec it currently costs CHF 289. How much does a 1 TB ESB Volume cost for a month?

The cost for a 1 TB ESB Volume is **116$** per month
