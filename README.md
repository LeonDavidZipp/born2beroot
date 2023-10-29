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
Verify sudo & ssh installation
```
dpkg -l | grep sudo
dpkg -l | grep ssh
```
### sudo configuration
installing sudo
```
apt-get update -y
apt-get upgrade -y
apt install sudo
```
then add user to sudo group
```
usermod -aG sudo username
```
### Logfiles
Logfile directory & path to logfile
```
/var/log/sudo/
/var/log/sudo/sudo.log
```
Read out content from logfile using cat
### Password Operations
Password of all users except evaluator:
```
Passwort12
```
During eval change to 
```
Passwort42
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
### monitoring.sh
Displays the given contents every 10 minutes. \
Create the file
```
cat> monitoring.sh
```
Write this inside
```
#!/bin/bash
wall $'#Architecture: ' `hostnamectl | grep "Operating System" | cut -d ' ' -f5- ` `awk -F':' '/^model name/ {print $2}' /proc/cpuinfo | uniq | sed -e 's/^[ \t]*//'` `arch` \
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
