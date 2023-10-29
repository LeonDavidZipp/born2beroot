# born2beroot
born2beroot project of the 42 core curriculum.

## Instructions for Evaluation
Connect to VM in iTerm (VM needs to be running for this!)
```
ssh lzipp@127.0.0.1 -p 4242
```
Exit running session
```
exit
```
Always reboot after changes to sudo!
```
sudo reboot
```
Always go into sudo mode first
```
su -
```
Show partitioning of disk
```
lsblk
```
Verify sudo & ssh installation & ufw & operating system
```
dpkg -l | grep sudo
dpkg -l | grep ssh
sudo systemctl status ufw
sudo service ssh status
uname -a
```
location of .vdi file
```
/goinfre/lzipp/vms/Debian/
```
### sudo configuration
Value of sudo
- increased security
- accountability (sudo logs everything being done)
- no need for having to log in as root user prevents misconfiguration potential

installing sudo
```
apt-get update -y
apt-get upgrade -y
apt install sudo
```
then add user to sudo group
```
usermod -aG groupname username
```
verify operation worked
```
getent group sudo
```
#### configuring sudoers group
go to `sudo nano /etc/sudoers` \
Add this
```
Defaults     mail_badpass
Defaults     passwd_tries=3
Defaults     badpass_message="lzipp says: Wrong password, remember you only hav>
Defaults     logfile="/var/log/sudo/sudo.log"
Defaults     iolog_dir=/var/log/sudo
Defaults     log_input,log_output
Defaults     requiretty
Defaults     secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sb>
```
### ssh configuration
install ssh
```
sudo apt-get update
sudo apt install openssh-server
```
get ssh status
```
sudo systemctl status ssh
```
In `sudo nano /etc/ssh/sshd_config`, change `#Port 22` to `to Port 4242`. Don't forget to change this in Virtual Box as well!
```
PermitRootLogin no
```
Check if port settings are correct
```
sudo grep Port /etc/ssh/sshd_config
```
restart ssh
```
service ssh restart
```
### Logfiles
Logfile directory & path to logfile
```
/var/log/sudo/
/var/log/sudo/sudo.log
```
Read out content from logfile using cat
### Password Operations
#### Changing Password Policy
Set policies here
```
sudo nano /etc/pam.d/common-password
password [success=2 default=ignore] pam_unix.so obscure sha512 minlen=10
password    requisite         pam_pwquality.so retry=3 lcredit =-1 ucredit=-1 dcredit=-1 maxrepeat=3 usercheck=0 difok=7 enforce_for_root
```
Set expiration here, then reboot using `sudo reboot`
```
sudo nano /etc/login.defs
PASS_MAX_DAYS 30
PASS_MIN_DAYS 2
PASS_WARN_AGE 7
```
Apply password policy to all users:
```
for user in $(cut -d: -f1 /etc/passwd); do
    	sudo chage -M 30 -m 2 -W 7 $user
	done
```
#### Settings before and after
Password of all users except evaluator:
```
Passwort12
```
During eval change to ... or the one below if dictionary tester still active
```
Passwort42
Pasw42,.420
```
Password of evaluator
```
Pasw123,.456
```
Change password of current user
```
passwd
```
, then press enter.
Change password of any user \
(be in su - mode!)
```
passwd username
```
, then press enter.
Apply password policy to all users:
```
for user in $(cut -d: -f1 /etc/passwd); do
    	sudo chage -M 30 -m 2 -W 7 $user
	done
```
Check which password rules apply to a certain user
```
chage -l username
```
### Host Operations
Check hostname
```
hostnamectl
```
Change hostname
```
hostnamectl set-hostname new_hostname
```
then also change the hostfile
```
sudo nano /etc/hosts
127.0.0.1       localhost
127.0.0.1       new_hostname	<--this!
```
### User Operations
#### Creation and Deletion
Add new user:
```
sudo adduser username
```
Delete user:
```
sudo userdel username
```
Check if succesful \
(simply displays all existing users)
```
cut -d: -f1 /etc/passwd
```
#### Users in Groups
Add user to group
```
sudo usermod -aG groupname username
```
Check if user in group
```
getent group username
```
Check to which groups user belongs to
```
groups username
```
### Group Operations
Create a group
```
sudo groupadd groupname
```
Delete a group
```
groupdel groupname
```
Check if operation worked
```
getent group
```
### Firewall UFW
downloading ufw
```
apt-get install ufw
```
configuring ufw
```
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 4242
```
changing rules
```
sudo ufw status numbered
sudo ufw delete number_of_rule
```
### monitoring.sh
Displays the given contents every 10 minutes. \
Create the file
```
cat> /usr/local/bin/monitoring.sh
```
Write this inside
```
#!/bin/bash
wall $'#Architecture: ' `uname -a` \
$'\n#CPU Physical: '`cat /proc/cpuinfo | grep processor | wc -l` \
$'\n#vCPU:  '`cat /proc/cpuinfo | grep processor | wc -l` \
$'\n'`free -m | awk 'NR==2{printf "#Memory Usage: %s/%sMB (%.2f%%)", $3,$2,$3*100/$2 }'` \
$'\n'`df -h | awk '$NF=="/"{printf "#Disk Usage: %d/%dGB (%s)", $3,$2,$5}'` \
$'\n'`top -bn1 | grep load | awk '{printf "#CPU Load: %.2f\n", $(NF-2)}'` \
$'\n#Last Boot: ' `who -b | awk '{print $3" "$4" "$5}'` \
$'\n#LVM Use: ' `lsblk |grep lvm | awk '{if ($1) {print "yes";exit;} else {print "no"} }'` \
$'\n#Connection TCP:' `netstat -an | grep ESTABLISHED |  wc -l` \
$'\n#User Log: ' `who | cut -d " " -f 1 | sort -u | wc -l` \
$'\nNetwork: IP ' `hostname -I`"("`ip a | grep link/ether | awk '{print $2}'`")" \
$'\n#Sudo:  ' `grep 'sudo ' /var/log/auth.log | wc -l`
```
#### Explanation:
- **wall (write all):** is used by system administrators (root users) to send important messages to all users who are currently logged into the system.
- **Cron:** Linux task manager that allows us to execute commands at a certain time. 
- **grep [options] pattern [file...]:** searches for a certain pattern.
### Definitions
- **APPArmor:** 
