# WazuhInstall

# PROJECT NAME

Installing and Configuring Wazuh

## Objective

Today I am going to install the open-source SIEM Wazuh. Wazuh Manager will be deployed on my Ubuntu machine and I will deploy my Kali Linux machine as an agent. 
At the end I will run a Configuration Assesment and harden my system by adding and modifying rules. 


### Tools Used

-Wazuh
-CLI

## Steps

First step is to run the following command. I am going to use curl to download and install

![image](https://github.com/user-attachments/assets/e224f892-1a9e-4ef0-bbb8-94c12049be60)

Installation of Wazuh Indexer, Dashboard, and Server are complete. I am provided with my credentials to access the Dashboard.

![image](https://github.com/user-attachments/assets/f19d467f-228a-4188-bdc3-4317c21cbdb0)

The dashboard is simply my IP address. Im on a Ubuntu VM. You can find out the IP address by running command ip addr.

![image](https://github.com/user-attachments/assets/35c9c137-47fb-4e5a-9835-1cd516798262)

When I go to the address, I am met with a warning. Not a problem. I have just set this up and know its safe as its a self signed certificate. 

![image](https://github.com/user-attachments/assets/c33c068f-2ea4-4a69-8f27-38cbdddeff60)

But just to be safe you can always check the certificate information.

![image](https://github.com/user-attachments/assets/73c1ebd1-a9bf-46fe-9790-68e7013cf9df)

![image](https://github.com/user-attachments/assets/c664d5de-8593-44f6-9ab4-f98d4411410b)

I accept the risk and continue and here is where I login with the credentials provided.

![image](https://github.com/user-attachments/assets/71b049f0-56f1-4781-bac0-3d39d79c0761)

Okay I am in. Its a nice interface but I don’t have any agents deployed. 

![image](https://github.com/user-attachments/assets/18fbf7d9-90ab-4dc2-8fb1-e189018eced8)

Before I can do anything I will need to update Kali Linux with sudo apt-get update. During the installation there was a requirement to have ports 1514 1515 and 443 open. In order to open the necessary ports, I need to install ufw.

![image](https://github.com/user-attachments/assets/7138829b-ca52-4728-867f-8599c4dbb164)

I enable ufw with sudo ufw enable. I run sudo ufw allow to allow the necessary ports and check the status to see the list.

![image](https://github.com/user-attachments/assets/ecc3c24d-c95b-443b-91f9-cd21d3c4e272)

I go back to my Ubuntu machine where I have the Wazuh Dashboard. I will be adding a DEB amd64.
I enter the IP address of the server which was my Ubuntu machine. I also assign an agent name. 

![image](https://github.com/user-attachments/assets/b8139c86-21a5-4c64-988c-9febcf2ed97a)

Here it provides me with a command to run to deploy and start wazuh on my Kali Linux machine.

![image](https://github.com/user-attachments/assets/2398ac08-25d8-4635-935b-5acc25b3b8dc)

I run the command.

![image](https://github.com/user-attachments/assets/3dadfaff-8540-4947-9558-d2f33c56ad5d)

The connection timed out. For some reason my machine was not connecting to the packages needed to download wazuh agent. Luckily, there is another way to download the wazuh agent with apt-get.

I run the below command and the agent is successfully installed.

![image](https://github.com/user-attachments/assets/5d332f30-f3a9-4c37-b495-93156629f67a)

I run command daemon-reload enable wazuh agent start wazuh agent

To see if it worked I run the command systemctl status wazuh-agent

Looks like it worked. 

![image](https://github.com/user-attachments/assets/f314e451-2370-42af-9c72-5cbba3262b0a)

I go back to the Wazuh Dashboard on my Ubuntu machine.

Perfect I’m up and running.

![image](https://github.com/user-attachments/assets/74beb7e9-6a38-4294-9c9d-d5a5bdf82366)

Now that my Kali Linux machine is up and running. I want to check out the configuration file 
nano /var/ossec/etc/ossec.conf

Here we can see that file integrity monitoring it set to no. Changing this to yes would enable real time monitoring of files. I don’t need this right now so I will just leave it as no.

![image](https://github.com/user-attachments/assets/dc308ca8-bd21-47fd-bc13-2cd9c788bc3b)

I see here we have 10 medium severity alerts on my dashboard.

![image](https://github.com/user-attachments/assets/c08e00b0-16e0-46f3-b3ac-d8ff0a030b35)

I can go in and take a peak. Everything is very smooth. The layout reminds me of Splunk.

Here I have an alert that warrants some digging.

![image](https://github.com/user-attachments/assets/e931fddd-0feb-4d20-8034-9f17af83916e)

Now normally this would warrant some concern but this is a fairly new system and I’d be shocked if this was anything serious. I looked this up and there are plenty of people on Github with the same issue. This is simply a false positive. As of this writing, I could not find a permanent solution. 

I am going to run a configuration assessment here and see what suggestions Wazuh has.

![image](https://github.com/user-attachments/assets/3448100b-4eaf-4a01-8547-734e364d9480)

I can see here 13 failed and 3 passed. Lets take a look at a failed assessment and see what the solution could be.

![image](https://github.com/user-attachments/assets/32391a47-524d-48fb-981d-31d51f88624b)

Here I am going to have passwords expire after 365 days. Sudo nano /etc/login.defs

I can see that it is currently set to 9999. Not good. 

![image](https://github.com/user-attachments/assets/62a43d5b-181b-48e9-b08b-12e5facb4103)

I change it to 365. I scroll down and notice max login retries is set at 5 but can be overwritten by PAM.

I save the file and head over to that configuration file to investigate.

![image](https://github.com/user-attachments/assets/bb1b27d9-336f-4d1f-b5ca-38875d284b9d)

Sudo nano /etc/pam.d/common-auth

I add the below rule to the configuration file. The below rule would mitigate against a potential brute force attack. 

#Lock out users after 5 unsuccessful consecutive login attempts.

auth required pam_tally2.so onerr=fail audit silent deny=5 unlock_time=900

![image](https://github.com/user-attachments/assets/2aa8c9ab-2c3f-4883-83dc-39d2ea84e07c)

I save and exit the file. 

Overall this is a great SIEM. I cannot believe its FREE. The interface is smooth and easy on the eye. Those that have used Splunk will find it familiar. I do hope they address the false positive issue soon. I am going to continue with the Configuration Assesment and harden my security systems.  

End. 
