---
title: 'How To Setup A Ubuntu Firewall'
weight: 1
---

# How to setup a firewall in Ubuntu

This guide shows the commands to setting up a firewall in Ubuntu

1. Open a terminal (either press Windows and A and click on the icon or press CTRL, ALT and T at the same time)
2. Type the following command to see if the firewall is active

```
sudo ufw status
```

3. If it is inactive run the following command to turn it on

```
sudo ufw enable
```

4. To install a graphical tool to enable you to control the firewall type the following:

```
sudo apt install gufw
```

5. You can now click on the firewall configuration icon in the Ubuntu apps screen