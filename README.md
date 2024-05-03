# Network Analysis PCAP Lab (WIP)

This lab is a report/walkthrough of the <a href="https://blueteamlabs.online/home/challenge/network-analysis-web-shell-d4d3a2821b">'Network Analysis – Web Shell'</a> lab from Blue Team Labs Online. The purpose of this lab is to investigate a given PCAP file to find suspicious actions in the web traffic captured by a SIEM.
__________________________________________________________________________________________________________________________________________________________________________

## Tools Used
- Wireshark
- URLexam-1.2(URL Decoder)
___________________________________________________________________________________________________________________________________

## Lab Walkthrough with Screenshots

For this lab, a PCAP file containing malware was given to investigate and find answers given to the questions in the screenshot below.

![image](https://github.com/schallon/Network-Analysis-PCAP-Lab/assets/55300128/464aecd6-fdf4-44aa-bf87-297734bc9e00)

After opening the PCAP file and opening the Protocol Hierarchy statistics shown below, there are a few key protocols to notice. SMB, DNS, SSH, and HTTP are protocols, some clear text, that can allow for lateral movement within the network allowing attackers to change their vectors of attack.

![image](https://github.com/schallon/Network-Analysis-PCAP-Lab/assets/55300128/f0dfb69b-a5e6-4cf7-8c07-8daae64e0fc1)

Next is to check the conversations. As shown in the screenshot below, a majority of conversations took place between two different conversations: 10.251.96.4 to/from 10.251.96.5 and 172.20.10.5 to 172.20.10.2. These two conversations added up to a total of 17,207 packets and just over 4.2 MB.

![image](https://github.com/schallon/Network-Analysis-PCAP-Lab/assets/55300128/d8f871d0-57ed-4d99-9853-e432e658e4ca)

Now looking at the TCP conversations, a vast majority of the entries in the list come from 10.251.96.4 using port 41675. This is indicative of port scanning and by looking a bit closer, almost all of the byte totals are the same, 118. However, the scan of port 80 and 22 have different byte totals of 184 indicating that they most likely responded with a SYN/ACK. Port 80 is used for HTTP and is commonly used by web servers while port 22 is typically used for communication most often SSH.

![image](https://github.com/schallon/Network-Analysis-PCAP-Lab/assets/55300128/eb50b4c1-4b07-4601-a68e-25d53badc322)

Looking near the bottom of the list, 10.251.96.5 has a total of 4 entries one of which uses 10.251.96.4's port 4422. The remaining three entries use port 80 indicating that it could possibly be a web server. The remaining entries are all from 172.20.10.2 and 172.20.10.5, where port 80 was used indicating another possible web server.

![image](https://github.com/schallon/Network-Analysis-PCAP-Lab/assets/55300128/aef6ab4b-8661-4dc3-8327-e2505e71f1f9)

Now to the actual packet trace. The first instance of HTTP communication occurs at packet 14 with communication from 172.20.10.5 to 172.20.10.2. By following the HTTP stream, the HTTP stream shows that 172.20.10.2 is an Ubuntu Web Server using Apache/2.4.29 with port 80 open. This aligns with previous findings from the conversation statistics.

![image](https://github.com/schallon/Network-Analysis-PCAP-Lab/assets/55300128/b9b3d788-d437-4869-8f97-ecc7e3e17777)
![image](https://github.com/schallon/Network-Analysis-PCAP-Lab/assets/55300128/e5d7a242-7bb6-493b-85cb-1912adad536b)

Looking further down in the packet trace, item 38 is a POST to 172.20.10.2 and more specifically, it is to a login.php page. The username and password entered for the POST were admin and Admin%401234 respectively. However, the %40 in the password is actually a special character. When decoded using a URL decoder, it shows that %40 is @ meaning the actual password is Admin@1234.

![image](https://github.com/schallon/Network-Analysis-PCAP-Lab/assets/55300128/9c351579-fa96-46dd-ab6d-2b2de1139ab9)
![image](https://github.com/schallon/Network-Analysis-PCAP-Lab/assets/55300128/bd67a946-ea9e-4f3c-bee7-e604aacfd87d)

A little bit further down in the packet trace DNS and ARP protocols begin to appear. The DNS packets are queries to ubuntu that resolve to IP addresses of 34.122.121.32, 35.232.111.17, and 35.224.170.84 which seems to be legitimate traffic based on what has already been found.

![image](https://github.com/schallon/Network-Analysis-PCAP-Lab/assets/55300128/719284fa-343b-45ad-95f3-c7ce63d1ea38)

Looking further, the port scanning from the conversations analysis seems to start with item 117. As previously thought, both port 80 and 22 returned a SYN/ACK when scanned as shown in the screenshot below, meaning that they are open ports.

![image](https://github.com/schallon/Network-Analysis-PCAP-Lab/assets/55300128/ce87a45c-fb77-4429-abb9-6541c3268679)

