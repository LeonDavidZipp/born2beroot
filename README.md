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
## sudo configuration
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
### configuring sudoers group
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
## ssh configuration
SSH
- Secure Shell
- provides secure encrypted communication
- mostly used for remote access, e.g. for communicating with a server and safe data transfer \

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
## Logfiles
Logfile directory & path to logfile
```
/var/log/sudo/
/var/log/sudo/sudo.log
```
Read out content from logfile using cat
## Password Operations
### Changing Password Policy
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
### Settings before and after
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
## Host Operations
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
## User Operations
### Creation and Deletion
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
### Users in Groups
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
## Group Operations
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
## Firewall UFW
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
## monitoring.sh
Displays the given contents every 10 minutes. \
Create the file
```
cat> /usr/local/bin/monitoring.sh
```
Write this inside
```
#!/bin/bash
arc=$(uname -a)
pcpu=$(grep "physical id" /proc/cpuinfo | sort | uniq | wc -l)
vcpu=$(grep "^processor" /proc/cpuinfo | wc -l)
fram=$(free -m | awk '$1 == "Mem:" {print $2}')
uram=$(free -m | awk '$1 == "Mem:" {print $3}')
pram=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
fdisk=$(df -Bg | grep '^/dev/' | grep -v '/boot$' | awk '{ft += $2} END {print ft}')
udisk=$(df -Bm | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} END {print ut}')
pdisk=$(df -Bm | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} {ft+= $2} END {printf("%d"), ut/ft*100}')
cpul=$(top -bn1 | grep '^%Cpu' | cut -c 9- | xargs | awk '{printf("%.1f%%"), $1 + $3}')
lb=$(who -b | awk '$1 == "system" {print $3 " " $4}')
lvmt=$(lsblk | grep "lvm" | wc -l)
lvmu=$(if [ $lvmt -eq 0 ]; then echo no; else echo yes; fi)
ctcp=$(cat /proc/net/sockstat{,6} | awk '$1 == "TCP:" {print $3}')
ulog=$(users | wc -w)
ip=$(hostname -I)
mac=$(ip link show | awk '$1 == "link/ether" {print $2}')
cmds=$(journalctl _COMM=sudo | grep COMMAND | wc -l)
wall "  #Architecture: $arc
        #CPU physical: $pcpu
        #vCPU: $vcpu
        #Memory Usage: $uram/${fram}MB ($pram%)
        #Disk Usage: $udisk/${fdisk}Gb ($pdisk%)
        #CPU load: $cpul
        #Last boot: $lb
        #LVM use: $lvmu
        #Connexions TCP: $ctcp ESTABLISHED
        #User log: $ulog
        #Network: IP $ip ($mac)
        #Sudo: $cmds cmd"
```
setup cron rule
```
sudo crontab -u root -e
```
put this inside
```
*/10 * * * * /usr/local/bin/monitoring.sh
```
### Explanation:
- **awk:** text-processing tool commonly used in Unix and Unix-like operating systems to process and manipulate text data, such as parsing and extracting specific fields from lines of text. It operates on a line-by-line basis, reading input text, applying rules or patterns to each line, and performing actions based on those rules
- **uname -a:** Retrieves system information
- **sort:** Sorts the output to prepare for unique counting.
- **uniq:** Filters out duplicate lines.
- **wc -l:** Counts the number of unique lines.
- **free -m:** Displays system memory information in megabytes.
- **df -Bm:** Displays disk space usage in megabytes
- **df -Bg:** Displays disk space usage in gigabytes.
- **top -bn1:** Runs the top command once.
- **cut -c 9-:** Removes the initial characters
- **xargs:** Converts the CPU load percentages into a single line
- **wall (write all):** is used by system administrators (root users) to send important messages to all users who are currently logged into the system.
- **cron:** Linux task manager that allows us to execute commands at a certain time. 
- **grep [options] pattern [file...]:** searches for a certain pattern.
## Retrieve key
```
cd /goinfre/lzipp/vms/Debian
shasum Debian.vdi
```
## Definitions
**apt vs aptitude**
- both help installing programs without having to worry about installing the correct dependencies
- apt is a simple command-line package manager for Debian-based systems
- aptitude is a more advanced package manager with a text-based interactive interface and better dependency resolution

**APPArmor:**
- AppArmor allows the system administrator to restrict the actions that processes can perform
- used to protect systemcritical applications on Linux operating Systems which could compromise the system would they receive compromised data
- is a mandatory access control system, meaning it imposes stricter rules by implementing passwords, keys, etc instead of only checking for name, identity, etc

**UFW** 
- not a firewall itself, only simplifies configuring preexisting firewall
- makes it easy to (dis)allow ports

**Virtual Machine**
- computer on a computer also running an operating system
- uses software instead of hardware to simulate a real computer
- enable safe testing of applications or accessing dangerous data, since they are separate from the real computer
- allow for multiple different operating systems on the same computer

**Rocky vs Debian**
- Debian is easier to use and install and is comunity driven
- Rocky was developed by the red hats
- 
VML
abstraction layer between a storage device and a file system. We get many advantages from using LVM, but the main advantage is that we have much more flexibility when it comes to managing partitions. Suppose we create four partitions on our storage disk. If for any reason we need to expand the storage of the first three partitions, we will not be able to because there is no space available next to them. In case we want to extend the last partition, we will always have the limit imposed by the disk. In other words, we will not be able to manipulate partitions in a friendly way. Thanks to LVM, all these problems are solved.
By using LVM, we can expand the storage of any partition (now known as a logical volume) whenever we want without worrying about the contiguous space available on each logical volume. We can do this with available storage located on different physical disks (which we cannot do with traditional partitions). We can also move different logical volumes between physical devices. Of course, services and processes will work the same way they always have.
![image](https://github.com/LeonDavidZipp/born2beroot/assets/117377515/5aa37023-7e1a-4a17-8013-51813f5955b0)
