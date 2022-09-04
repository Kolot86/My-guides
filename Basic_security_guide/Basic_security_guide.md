# Basic security guide 

## OBS! If it’s a first time you making SSH keys authentication. I strongly advise to try it on empty server which one you can reboot in case if you do something wrong! 

Hello. 

Here is basic security guide that every beginner validator should know and use as first layer of security. 

This guide is for complete beginners  and it’s just my adaptation of guide from digital ocean, with a little extra info from me https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04#step-3-authenticating-to-your-ubuntu-server-using-ssh-keys

Feel free to use original. 

First, we begin to learn about SSH, or secure shell. It is an encrypted protocol, used to administer and communicate with servers. On our server vi going to use Ubuntu 20.04 for installation.

The first step is to create a key pair on your privet computer. To do this you can use command line interference (like PowerShell for Window in my case):
```bash
ssh-keygen
```
After entering the command, you should see the following output:
```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/your_home/.ssh/id_rsa):
```
Press enter to save the key pair into the .ssh/ subdirectory in your home directory, or specify an alternate path.
You should then see the following prompt:
```bash
Enter passphrase (empty for no passphrase):
```
Here you optionally may enter a secure passphrase, which is highly recommended

You should then see the output similar to the following:
```bash
C:\Users\Asus>ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\Asus/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\Asus/.ssh/id_rsa.
Your public key has been saved in C:\Users\Asus/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:rlYGysE4G2t3wT4FEvAFzHSwUXP+3PnlkugSyf401ho asus@DESKTOP-SIB6BG6
The key's randomart image is:
+---[RSA 3072]----+
|  .=*== .        |
|   .+=.+         |
|   ooo ..        |
|  + o + .o . .   |
|   * + +S.o.o   .|
|  + + +.o +  + + |
| . . . +.. .E = .|
|      ..  o+ + . |
|     ..    o+    |
+----[SHA256]-----+
```
You now have a public and private key that you can use for authentication. The next step is to place the public key on your server so that you can use SSH-key-based authentication to log in.

## Copying the Public Key to Your Ubuntu Server
We going to copy public key manually to our ubuntu server.

First you can open id_rsa.pub in PowerShell or other command line interference like this:
```bash
cat ~/.ssh/id_rsa.pub
```
Output in my case:
```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC+Z4Q0Ibukn8bxp6YwXWnUTo6eCaCf2mnARawC95sD+9vhPkhvYC7+YoSkbMa4g6xxt8mssGGK738peGc4BiPu6iSzcpTAF1pAkQqTHdbokP6Bl5WAf53LuH/NS45P4UyNK4XIMyeEHTCuMlSB351n4niC9K/g0Wk+7yoNSbT71oDE+SWdnMq7cW3ofP/l8QTj/ijL3gCScnfwDKZTCvubxue6hOEabdQZnrFW4avxLr9DNEkzbrcX7AWSE6hLjldjm4scqEOgGec64DEJips6Dho0EUDL4W4+onq0NtNopYik8m+AURY0oTmcCGjoTVHcoiQqFAoMT0fXhHaZBC8h4KEu21/MI68Ch+PsZr/KVc4+tvRDhqdQ5EpszYC2LB/497fatrwADy9M0cLp6DCetPXwXj8UFaFp/XmYb/s1XQCg6f9y+IPtYR8jz30t1g4oNAtfy7EJziD/vCkiR8MseSTQhc0MMAlgWbHB8LScktpjrUWGen9ZRz7F5KPlnls= asus@DESKTOP-SIB6BG6
```
Access your remote host using whichever method you have available.I prefer MobaXterm
```bash
ssh root@Your_remoute_server_IP
```
Once you have access to your account on the remote server, you should make sure the ~/.ssh directory exists. This command will create the directory if necessary, or do nothing if it already exists
```bash
mkdir -p ~/.ssh
```
Now, you can create or modify the authorized_keys file within this directory. Copy data from id_rsa.pub with this command:
```bash
nano ~/.ssh/authorized_keys
```
Press "crl + o" to save and enter. To close "ctrl + x"
You can also upload the whole file with MobaXterm

Finally, we’ll ensure that the ~/.ssh directory and authorized_keys file have the appropriate permissions set:
```bash
chmod -R go= ~/.ssh
```
Now you can connect to your server with ssh keys

ssh "root@your_remote_host_IP"

In my case it’s looks like this:
```bash
PS C:\Users\Asus> ssh root@78.47.166.77
The authenticity of host '78.47.166.77 (78.47.166.77)' can't be established.
ECDSA key fingerprint is SHA256:QNrtYP0m1bA0DiD39aS8CNl8mc+6VyG4JaIEHWirpLc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? y
Please type 'yes', 'no' or the fingerprint:
```
Type yes and then press ENTER to continue
If you did not supply a passphrase for your private key, you will be logged in immediately. If you supplied a passphrase for the private key, you will be prompted to enter it now. After authenticating, a new shell session should open for you with the configured account on the Ubuntu server.
Now you can login using you ssh keys and switch off password  authentication 

But we going to create non root user also 
## Creating non root user 
First we going to login to our server ass root user and create non root user.
To add new user we  using command:
adduser <Your_user>
I did so
```bash
adduser kolot
```
Output
```bash
Adding user `kolot' ...
Adding new group `kolot' (1000) ...
Adding new user `kolot' (1000) with group `kolot' ...
Creating home directory `/home/kolot' ...
Copying files from `/etc/skel' ...
New password:
```
You need to create a password for your user. After you have created password, you can write additional information about your user if you want, or leave it empty

After, we can add our user in sudo grop:
```bash
usermod -aG sudo <your_user>
```
In my case
```bash
usermod -aG sudo kolot
```
Right now, we can only use ssh keys for a root account. We going to setup ssh keys for user account also.
We can use this command to create .ssh directory and copy our public key the same way as we did before
```bash
cd /home/<whrite your user name>
mkdir .ssh
nano .ssh/authorized_keys
```
In my case it's looks like this:
```bash
cd /home/kolot
mkdir .ssh
nano .ssh/authorized_keys
```



Now you can also connet as non root user to your server with ssh keys lets try. Here is exemple with my user, change to your own

```bash
ssh kolot@78.47.166.77
```
If everything works well, we can connect again to our server as root users, switch off password authentication and deny root longing. So, it will be possible to use only specified user and ssh keys authentication, to login to our server. 

## Switch off password authentication and deny root login 

Login to you server check if everything works good, that you can connect as root and ass user, with ssh keys to your server. If ok, do next step. Open your ssh config and switch off PasswordAuthentication and root logging
```bash
sudo nano /etc/ssh/sshd_config
```
Those parameters should look like this   

```bash
PermitRootLogin no
PasswordAuthentication no
```
To actually activate these changes, we need to restart the sshd service:
```bash
sudo systemctl restart ssh
```
Congratulation! 
Now you can feel yourself safer.
