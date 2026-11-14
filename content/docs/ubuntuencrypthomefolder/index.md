---
title: 'How To Encrypt Your Ubuntu Home Folder'
weight: 1
---

# How to encrypt your Ubuntu Home folder

This guide shows you how to encrypt your Ubuntu home folder without reinstalling Ubuntu.

1. Backup any important files and data
2. Go to the apps screen (Windows and A key) and click on settings
3. Search for users
4. Click unlock and then add a user and call it tempadmin and give it admin access and set a password for the user
5. Log out of Ubuntu as your user and login as tempadmin
6. Skip through the welcome screens
7. Open a terminal and type the following commands to install the encryption utilities and encrypt the home folder (Replace <yourusername> with the username of the user you want to encrypt)

```
sudo apt update
sudo apt install -y ecryptfs-utils cryptsetup
sudo ecryptfs-migrate-home -U <yourusername>
```

8. Log out of temp admin and back in as your user
9. Make sure you can open all of your files
10. Open a terminal and type the following commands:

```
cd /home
ls
```

11. You will see a folder for your user, tempadmin and then your user again with random letters (I.E. gary_123ksdfk). Remove the folder by typing the following command:


```
sudo rm -rf /home/gary_123ksdfk (or whatever the folder is called on your machine)
```

12. Go to the settings screen and users again and click on the temp admin user and click remove user.
