# Securing linux server

## SSH related

- If this server already has a public IP, you’ll want to lock down root access immediately. In fact, you’ll want to lock down SSH access entirely, and make sure that only you can get in. Add a new user, and add it to an admin group (preconfigured in /etc/sudoers to have access to sudo).

```sh
$ sudo addgroup admin
Adding group 'admin' (GID 1001)
Done.

$ sudo adduser spenserj
Adding user `spenserj' ...
Adding new group `spenserj' (1002) ...
Adding new user `spenserj' (1001) with group `spenserj' ...
Creating home directory `/home/spenserj' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for spenserj
Enter the new value, or press ENTER for the default
    Full Name []: Spenser Jones
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
Is the information correct? [Y/n] y

$ sudo usermod -aG admin spenserj
```

- create a private key on your computer, and disable that pesky password authentication on the server.

```sh
$ mkdir ~/.ssh
$ echo "ssh-rsa [your public key]" > ~/.ssh/authorized_keys

# in /etc/ssh/sshd_config
PermitRootLogin no
PermitEmptyPasswords no
PasswordAuthentication no
AllowUsers spenserj
```

- Reload SSH to apply the changes, and then try logging in in a new session to ensure everything worked. If you can’t log in, you’ll still have your original session to fix things up.

## Update the server

```sh
$ sudo apt-get update && sudo apt-get upgrade
```

## IPTables

```sh
$ sudo mkdir /etc/iptables
$ sudo vi /etc/iptables/rules
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT DROP [0:0]

# Accept any related or established connections
-I INPUT  1 -m state --state RELATED,ESTABLISHED -j ACCEPT
-I OUTPUT 1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# Allow all traffic on the loopback interface
-A INPUT  -i lo -j ACCEPT
-A OUTPUT -o lo -j ACCEPT

# Allow outbound DHCP request - Some hosts (Linode) automatically assign the primary IP
#-A OUTPUT -p udp --dport 67:68 --sport 67:68 -j ACCEPT

# Outbound DNS lookups
-A OUTPUT -o eth0 -p udp -m udp --dport 53 -j ACCEPT

# Outbound PING requests
-A OUTPUT -p icmp -j ACCEPT

# Outbound Network Time Protocol (NTP) request
-A OUTPUT -p udp --dport 123 --sport 123 -j ACCEPT

# SSH
-A INPUT  -i eth0 -p tcp -m tcp --dport 22 -m state --state NEW -j ACCEPT

# Outbound HTTP
-A OUTPUT -o eth0 -p tcp -m tcp --dport 80 -m state --state NEW -j ACCEPT
-A OUTPUT -o eth0 -p tcp -m tcp --dport 443 -m state --state NEW -j ACCEPT

COMMIT

$ sudo iptables-apply /etc/iptables/rules
$ sudo vi /etc/network/if-pre-up.d/iptables
#!/bin/sh
iptables-restore < /etc/iptables/rules

$ sudo chmod +x /etc/network/if-pre-up.d/iptables
$ sudo /etc/network/if-pre-up.d/iptables
```

## Fail2ban

```sh
$ sudo apt-get install fail2ban
```


[ref1](http://spenserj.com/blog/2013/07/15/securing-a-linux-server/)

[ref2](http://blog.csdn.net/jlds123/article/details/38110779)
