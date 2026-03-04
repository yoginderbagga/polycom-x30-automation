Building a Simple VC Healthchecks System using Ansible Automation

A healthcheck system lets you monitor the device state, connectivity, and other important status details right from your terminal / CLI to proactively monitor and identify the issue before they even arise. 

Using this Ansible playbook from your Linux machine you can connect to a Poly Studio X32 ( Video Conference Device ) in the Red Hat office and fetch the crucial details that can help you to troubleshoot the device effectively without login to the SignalFX.

How to Run and Configure Ansible to act as a VC Healthcheck System

Prerequisites:

Ansible Packages 
Ansible vault 
Grafana-server ( localhost ) 
Red Hat VPN
Access to Polycom VC



Step 1: Install both the Ansible packages “ansible-core” and “ansible”
Step 2: Copy the playbook Polycom_Access_test.yml in a specific folder where you like to keep the Ansible Project. 


ybagga@ybagga-thinkpadt14sgen2i:~/Ansible_Project2$

Step 3: Create an inventory file in your Ansible_Project folder and add managed host details i.e the device you want to monitor. In this case, we add the IP address of the video bar 10.76.245.4



ybagga@ybagga-thinkpadt14sgen2i:~/Ansible_Project2$ cat inventory
[polycom_host]
10.76.245.4
ybagga@ybagga-thinkpadt14sgen2i:~/Ansible_Project2$ 

Step 4: Create an encrypted ansible file to store the credentials variable secret2.yml

ybagga@ybagga-thinkpadt14sgen2i:~/Ansible_Project2$ vim secret2.yml 

polycom_user: GES_TestAdmin
polycom_password: GES_Test@@

ybagga@ybagga-thinkpadt14sgen2i:~/Ansible_Project2$ ansible-vault encrypt secret2.yml 
New Vault password: 
Confirm New Vault password: 
Encryption successful



Step 5: Create ansible.cfg file 



ybagga@ybagga-thinkpadt14sgen2i:~/Ansible_Project2$ cat ansible.cfg 
[defaults]

inventory = /home/ybagga/Ansible_Project2/inventory
# forks 5 is default setting that Ansible will manage at most 5 hosts simultaneously
forks = 5
# Suppresses Python interpreter warnings on your ThinkPad
interpreter_python = auto_silent



Step 6: Run the playbook with both the options : Reboot or non-Reboot and verify the results at Polycom Web UI https://10.76.245.4/#/login

Run the playbook with non-Reboot option : In this scenario, Ansible playbook will connect to the host device and fetch its health check details on CLI as playbook output. And you can also configure Grafana for monitoring purposes to re-direct the output at Grafana Dashboard. 
Run the playbook with reboot option : ( Use it caution, to verify meeting is not in use during the reboot time, book the room while testing )  

Option A)

ybagga@ybagga-thinkpadt14sgen2i:~/Ansible_Project2$ ansible-playbook VC_Healthchecks.yml --ask-vault-pass
Vault password: 

PLAY [Check Polycom VC Status via Session-Based REST API (Secure)] **********************************************************************************************************

TASK [1. Login - POST Credentials to Get Session Cookie] ********************************************************************************************************************
ok: [10.76.245.4]

TASK [2. Get System Status using the Session Cookie] ************************************************************************************************************************
ok: [10.76.245.4]

TASK [3. Display Parsed System Status] **************************************************************************************************************************************
ok: [10.76.245.4] => {
    "msg": "✅ Poly Studio X30 Status - ASLEEP\n---------------------------------------\nSoftware Version: 4.3.0\nSystem Name: video-bar-3n140.pnq.redhat.com\nUptime: 0.3055555555555556 hours\nNetwork Link: LAN_UP (1000 Mbps)\n"
}

TASK [4. Send Device Health Status to Grafana] ******************************************************************************************************************************
ok: [10.76.245.4]

TASK [5. Prompt for Action] *************************************************************************************************************************************************
[5. Prompt for Action]
Type 'y' to REBOOT. Press any other key to EXIT safely.:
ok: [10.76.245.4]

TASK [6. Exit Gracefully if not 'y'] ****************************************************************************************************************************************

PLAY RECAP ******************************************************************************************************************************************************************
10.76.245.4                : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   





Option B)


ybagga@ybagga-thinkpadt14sgen2i:~/Ansible_Project2$ ansible-playbook VC_Healthchecks.yml --ask-vault-pass
Vault password: 

PLAY [Check Polycom VC Status via Session-Based REST API (Secure)] **********************************************************************************************************

TASK [1. Login - POST Credentials to Get Session Cookie] ********************************************************************************************************************
ok: [10.76.245.4]

TASK [2. Get System Status using the Session Cookie] ************************************************************************************************************************
ok: [10.76.245.4]

TASK [3. Display Parsed System Status] **************************************************************************************************************************************
ok: [10.76.245.4] => {
    "msg": "✅ Poly Studio X30 Status - ASLEEP\n---------------------------------------\nSoftware Version: 4.3.0\nSystem Name: video-bar-3n140.pnq.redhat.com\nUptime: 0.3213888888888889 hours\nNetwork Link: LAN_UP (1000 Mbps)\n"
}

TASK [4. Send Device Health Status to Grafana] ******************************************************************************************************************************
ok: [10.76.245.4]

TASK [5. Prompt for Action] *************************************************************************************************************************************************
[5. Prompt for Action]
Type 'y' to REBOOT. Press any other key to EXIT safely.:
ok: [10.76.245.4]

TASK [6. Exit Gracefully if not 'y'] ****************************************************************************************************************************************
skipping: [10.76.245.4]

TASK [7. Execute Reboot] ****************************************************************************************************************************************************
ok: [10.76.245.4]

TASK [8. Log Out] ***********************************************************************************************************************************************************
skipping: [10.76.245.4]

PLAY RECAP ******************************************************************************************************************************************************************
10.76.245.4                : ok=6    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   





Verification : 
As it can be seen, the host is unreachable as you selected to perform the reboot, so the device is restarting and pin unreachable.

ybagga@ybagga-thinkpadt14sgen2i:~/Ansible_Project2$ ping -c 4 10.76.245.4
PING 10.76.245.4 (10.76.245.4) 56(84) bytes of data.
From 10.76.255.136 icmp_seq=1 Destination Host Unreachable
From 10.76.255.136 icmp_seq=2 Destination Host Unreachable
From 10.76.255.136 icmp_seq=3 Destination Host Unreachable
From 10.76.255.136 icmp_seq=4 Destination Host Unreachable

--- 10.76.245.4 ping statistics ---
4 packets transmitted, 0 received, +4 errors, 100% packet loss, time 3053ms
pipe 4


Below web-UI screenshot, shows when the device was not rebooted. 

<img width="1918" height="957" alt="Screenshot From 2026-03-04 08-08-32" src="https://github.com/user-attachments/assets/7fc0b6db-1352-4124-b21b-b7c6df7c29df" />






Part B: Show the results on Grafana Dashboard :

<img width="1920" height="1080" alt="Grafana_Output" src="https://github.com/user-attachments/assets/818c0083-ee65-412f-aa6a-1e0bb79cdaac" />





(Polycom WebUI : Login to the webUI interface to verify if the device is UP and running properly.)

<img width="1918" height="957" alt="Screenshot From 2026-03-04 08-45-51" src="https://github.com/user-attachments/assets/ee790651-79fe-4b67-895a-b266b1761205" />

