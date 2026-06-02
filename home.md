# EXAM PROJECT REPORT | HEARTBLEED VULNERABILITY EXPLOITATION | GUIDO LORENZO VALENTINO
 
## INTRODUCTION

This report describes the analysis, the verification and the exploitation of the Heartbleed vulnerability, classified as CVE-2014-0160. The workflow can be divided into three main phases:

* **Deployment of the target environment:** Configuration and launch of the vulnerable service via Docker, simulating the target host.
* **Information gathering and vulnerability assessment:** Utilization of the Nmap network scanner to analyze the dedicated network port and subsequently validate the presence of the vulnerability on the target host.
* **Exploitation:** Leveraging the vulnerability to extract chunks of data from the server's RAM.

The activities were conducted within a controlled laboratory environment using the Kali Linux distribution as the attack machine and the Docker environment, provided by Vulhub, to simulate the target machine.

---

## HEARTBLEED VULNERABILITY

The Heartbleed vulnerability is identified as CVE-2014-0160. This security bug is located within the OpenSSL cryptography library, which is widely used in the TLS protocol to manage secure connections. The vulnerability is classified as a buffer over-read, a condition where software allows more data to be read than should be permissible.   

The security bug occurs in the Heartbeat mechanism, which is used to keep the connection alive: the client sends a string while also declaring its length and the server responds by sending that string back while matching the declared length.   

The issue is that OpenSSL blindly trusts the input and does not perform any validation to check the match between the actual length of the string and the declared value. If an attacker sends a string but lies about the length by declaring a larger value, the server, in an attempt to match the declared length, will also send back all the bytes in memory located beyond the stored string. This triggers the buffer over-read.   

During this laboratory activity, after scanning the target host to confirm the presence of the vulnerability, the exploitation phase was carried out using a Python script. The script reproduced the attack by sending a forged TLS Heartbeat request (altering the length field), allowing the extraction and subsequent analysis of the data residing in the target server's memory.  

---


## PHASE 1: LAUNCHING THE VULNERABLE SERVICE

Once logged into the Kali Linux virtual machine and having opened the terminal, the first operation performed was navigating into the specific folder containing the lab files. To achieve this, the following command was executed:  


`cd vulhub/openssl/CVE-2014-0160`


Subsequently, the Docker laboratory environment was started by running the command:   


`sudo docker compose up -d`


The system required the administrator password. Once the procedure was completed, the container hosting the vulnerable server was successfully started and active, ready for the subsequent commands.

---

## PHASE 2: PORT VERIFICATION AND VULNERABILITY ASSESSMENT

Once the vulnerable service was up and running, the scanning and security verification phase began. Specifically, the Nmap network scanner was used by executing the following command:   


`nmap -p 443 --script ssl-heartbleed localhost`   


This command utilizes Nmap to query standard port 443 on the local machine (localhost) and apply the specific script named ssl-heartbleed, which is designed to check for the bug by sending test payloads to the server. Following the execution, it was observed that the state of port 443 was closed, meaning that the service was not listening on that specific port. To find the correct port number, the following command was executed:   


`sudo docker ps`


From the terminal output, it was verified that the server's internal port 443 had been mapped to port 8443 of the host machine. The scan was then repeated using the correct port number:  


`nmap -p 8443 --script ssl-heartbleed localhost`


In this second attempt, the port state was open and the script output provided confirmation, explicitly identifying the server as VULNERABLE to the Heartbleed bug.

---

## PHASE 3: VULNERABILITY EXPLOITATION

Once the presence of the Heartbleed vulnerability was validated via Nmap, the actual exploitation phase was carried out by running the command:


`python3 ssltest.py localhost -p 8443`


This command launches the Python script named ssltest.py, targeting port 8443 of the local machine. The script is specifically designed to send an altered TLS Heartbeat request, lying about the input length to induce the server into a buffer over-read and extract approximately 63.000 bytes of data from the system memory.
 
From the terminal output, it was possible to verify that the script completed the operation successfully, receiving the response from the Heartbeat and displaying the first extracted data. In this first attempt, a large portion of the dump consisted almost entirely of zeros (meaning empty memory areas) and did not reveal any sensitive information. This result was expected: since the Docker container had just been launched, the RAM was still clean and contained only the few bytes related to the initial configuration of the service.
 
To make the test realistic, it was decided to simulate user activity to generate data within the server's memory. The Firefox browser was opened to connect to the vulnerable server's address. Having generated this real network traffic on the server, the exploit command was executed again:

 
`python3 ssltest.py localhost -p 8443`
 
This time, the output obtained was different: the attack was successful, allowing to clearly see the metadata and headers of the HTTP request previously sent by Firefox in plain text.

---

## FINAL CONSIDERATIONS

In a real-world environment, the server's RAM would host a critical amount of sensitive information, such as private cryptographic keys, session cookies and plaintext login credentials. The potential impact of this vulnerability is therefore devastating.   

It is important to highlight that this attack is highly situational: the data obtained depends exclusively on the information passing through the system's memory at the exact moment the Heartbeat packet is sent. Consequently, in a real scenario, an attacker would execute the script cyclically and automatically until gathering data of actual interest.   

Furthermore, it should be noted that executing this attack required only a few commands and no complex strategic effort. This highlights that high-impact threats do not necessarily require sophisticated skills or complicated strategies to be executed. Unsurprisingly, vulnerabilities that combine a maximum ease of exploitation with critical potential damage are classified as the most dangerous ones. The Heartbleed vulnerability is definitely one of them.   

Finally, it is crucial to emphasize that Heartbleed was a software code implementation bug rather than a logical or structural flaw within the TLS protocol itself. This distinction allowed for an immediate and effective mitigation, centered around the release of a corrective update. It was enough to update the OpenSSL libraries to a patched version to fully resolve the vulnerability, which is why this bug is today, widely resolved and obsolete.


---


## REFERENCES & SOURCES

The following resources and documentation were utilized for the preparation, deployment and execution of this laboratory activity:
 
* **Vulnerable Environment:** Vulhub Repository for CVE-2014-0160 – [https://github.com/vulhub/vulhub/tree/master/openssl/CVE-2014-0160](https://github.com/vulhub/vulhub/tree/master/openssl/CVE-2014-0160)
* **Network Scanner Documentation:** Nmap Scripting Engine (NSE) `ssl-heartbleed` Reference – [https://nmap.org/nsedoc/scripts/ssl-heartbleed.html](https://nmap.org/nsedoc/scripts/ssl-heartbleed.html)
* **Containerization Platform:** Docker Compose Official Documentation – [https://docs.docker.com/compose/](https://docs.docker.com/compose/)
