For this project, I am going to set up two machines, one as a FTP server and another for active packet capture. 
In my case, I decided to use a Windows 11 machine as a virtual FTP Server, and a Kali Linux machine for active reconnaissance, 
because Kali comes with pre-installed reconnaissance tools, and NMAP works much better on Kali (as root user), 
as for the Windows 11 machine, - it is my first time building an FTP Server, so it is easier to follow updated guidelines.
First, I installed Windows 11 on VirtualBox, and enabled FTP Server feature:

![Picture46](https://github.com/mikekad1/networksec/assets/62948569/60c93cb0-8a7b-44d6-81ca-9a0f3be64d83)

After this, I created directory for my site as well as a default user and gave him appropriate permissions:

![Picture47](https://github.com/mikekad1/networksec/assets/62948569/4671c0b7-5fec-4896-bf14-ccab5103178f)

Meanwhile, my Kali machine is ready to go. To ensure that everything works, I pinged my FTP Server from local host machine and finally logged into it remotely using ftp command.

![Picture48](https://github.com/mikekad1/networksec/assets/62948569/021d5481-2a86-405d-9f8d-ab301fccb551)

Now that my environment has been set-up. Time for some reconnaissance.
After firing up my attacking machine (Kali), I ran my standard command:
sudo nmap -O -A 192.168.179.8
I started my command with sudo, because otherwise I was not able to detect host and it appeared to be down. 
Next I used “-O” for Operating System detection, this could tell me a whole bunch of things, like if there are outdated security features on the system, what are the defaults etc.
“-A” is my wildcard for aggressive scanning, I use it to get as much information as I can on ports and protocols running. So, my result is…

![Picture49](https://github.com/mikekad1/networksec/assets/62948569/a07686fc-794e-4f8b-85c1-d8f2be300899)

NMAP did a pretty good job on uncovering some vulnerabilities and ports. We see a whole 5 of them, with known exploits (Netbios, RPC, FTP etc.). 
Unfortunately, version of Windows is incorrect, but this could be due to the fact that it is still pretty new… I decided to go a little further to scan an entire network so I can build a diagram.

![Picture50](https://github.com/mikekad1/networksec/assets/62948569/f8be2e4d-5bf5-43f3-993c-4f93a5c44290)

To scan entire network I entered default gateway IP with /24. Here is a diagram I compiled together from that information:

![Picture51](https://github.com/mikekad1/networksec/assets/62948569/043288f7-2583-4c14-abdb-69b768c242ac)

To capture packets containing log-in details, I am starting out with TcpDump:

![Picture52](https://github.com/mikekad1/networksec/assets/62948569/30cb3d03-a5c9-47bf-af72-d8d23f05582b)

Here I am having tcpdump listen to ftp port on eth0 device and save it to the file, meanwhile I connected my attack machine to the FTP Server. 
After sending log in details, I terminated both tcpdump and ftp and examined file for password which came up in cleartext. I than looked up username - 

![Picture53](https://github.com/mikekad1/networksec/assets/62948569/11c95912-9ab8-4b2e-b7a6-182dd1fd595c)

And it came up, also in cleartext. 
For my second test, I am using Wireshark. In the similar manner, I started up scanning program first (wireshark) and logged into FTP Server after.

![Picture54](https://github.com/mikekad1/networksec/assets/62948569/e9c01613-e809-4d0c-9288-73f7177fdf98)

While running the two side by side, I sent credentials to FTP server and than stopped both programs. It is now time to filter packets by ftp tag - 

![Picture55](https://github.com/mikekad1/networksec/assets/62948569/23ba7677-14ed-48ac-8f7b-14142b05d1fe)

Even though, we can already see the credentials, we can follow entire TCP stream to view the conversation - 

![Picture56](https://github.com/mikekad1/networksec/assets/62948569/04bdeb81-5a67-4d72-bc4b-ee65e9d57736)

Now that we see how vulnerable FTP is to credential harvesting, it is time to find out how secure is SFTP?
SFTP works over SSH and uses port 22 for data transfer. That is why, first thing I needed to do is to enable OpenSSH feature on the FTP Server machine - 

![Picture57](https://github.com/mikekad1/networksec/assets/62948569/2ad2ff5c-3047-41c8-98eb-bdfd242cbc94)

After setting up OpenSSH, I made sure to enable the Service and restarted the machine. Next thing I did was, start up tcpdump on Kali and made SSH connection to the SFTP Server:

![Picture58](https://github.com/mikekad1/networksec/assets/62948569/c50e4728-8ac7-484f-85dc-1c9638f18065)

After logging in, I was in the system directory – 

![Picture59](https://github.com/mikekad1/networksec/assets/62948569/b003d7a5-f06b-4107-864a-dc7e94c171f5)

I decided to check the captured results. However…

![Picture60](https://github.com/mikekad1/networksec/assets/62948569/2877c3a4-95fd-4644-a6a0-c3ec027b42d0)

It all looks like complete mess of characters. 
To ensure that data is indeed encrypted I repeated the same process with Wireshark. When I followed the communication stream between server and client, I got - 

![Picture61](https://github.com/mikekad1/networksec/assets/62948569/f92163a5-fae3-416c-af7e-b0f82ddbfa19)

Communication is definitely encrypted.         

Although most of the crucial data is encrypted, we see some data in plaintext. Actually this data is necessary to be displayed because it specifies which algorithms are used for encryption.
Aside from that, we can see what versions of SSH being used by server and client. This could be useful if there are known vulnerabilities from outdated versions. The unpatched versions of SSH could be vulnerable to brute force password attacks. That is why it is highly advisable to disable password-based authentication.
If such a vulnerability to be exposed, attacker could use lateral movement attack to gain root privileges, and/or log-in remotely as root.
Compromise could come from other vulnerabilities on the systems in the same local network. In this case, if there is a key assigned to trusted host, attacker could take advantage of it.
In conclusion, SFTP is most definitely a better solution of the two, however it is not 100% guarantee against remote access attacks, therefore additional security measures need to be taken.





