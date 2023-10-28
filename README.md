# born2beroot
born2beroot project of the 42 core curriculum.

## Instructions for Evaluation
Always reboot after changes to sudo!
```
sudo reboot
```
Always go into sudo mode first
```
su -
```
### Password Operations
Password of all users except evaluator:
```
Passwort12
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
Change password of any user (be in su - mode!)
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
		-->127.0.0.1       new_hostname<--
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
Check if succesful (simply displays all existing users)
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
