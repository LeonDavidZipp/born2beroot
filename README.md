# born2beroot
born2beroot project of the 42 core curriculum.

## Instructions for Evaluation
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
'''
sudo usermod -aG groupname username
'''
Check if user in group
'''
getent group username
'''
Check to which groups user belongs to
'''
groups username
'''
