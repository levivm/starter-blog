---
cover_image: 'https://thepracticaldev.s3.amazonaws.com/i/8qoa7qwomdnixm0yf6yg.png'
date: 2019-07-17T03:42:38.840Z
description: How to use SSH and SSH Agent forwarding
draft: true
published: true
title: How to use SSH properly and what is SSH Agent Forwarding
tags:
  - beginners
  - devops
  - webdev
  - security
---

The SSH (Secure Shell) is widely used to provide secure access to remote systems, we have few ways to do it. Basically, everybody who uses it knows about the routine password access. But, there are some issues with this approach, mainly, it allows brute-force password guessing.

SSH provides better authentication process. We are going to recap basic ways to do SSH and view more advanced options in order to be able to use our ssh keys in the external server without putting them directly over there. 

First method is using private/public keys between local machine and the remote one. 


### Using SSH Public Key access

As we know, if we try to connect to a server using basic SSH. It will prompt interactive shell asking us the password. In order to avoid for the server prompting to enter a password each time that we want to connect. We create a pair of public and private keys.  

1. Start key generation program

```
$ ssh-keygen -o -a 100 -t ed25519
```
2. Enter the path to the file that will hold the key.

```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key(/home/johndoe/.ssh/id_rsa): 
```

You should enter the path to the file that will hold the key, by default is `id_rsa` on your `.ssh` directory. 

This will create two files, a private key, and a public one. The public key will have `.pub` appended to its name. 

3. Enter a passphrase for using your key.

This is passphrase is used to encrypt your key. It will be requested for further connections. You can leave it blank but is __strongly discouraged__. 

```
Enter passphrase(empty for no passphrase):  <Type the passphrase>
```
4. Re-enter your passphrase

```bash
Enter same passphrase again: <Type the passphrase>
Your identification has been saved in /home/johndoe/.ssh/id_rsa.
Your public key has been saved in /home/johndoe/.ssh/id_rsa.pub.
The key fingerprint is:
0e:fb:3d:57:71:73:bf:58:b8:eb:f3:a3:aa:df:e0:d1 johndoe@myLocalHost
```

5. Verify the results 

You should have both keys in your selected directory. If everything went well, we are ready to use ours keys. 

6. Copy the public key on the remote host.

We need to add our public key `id_rsa` on our remote host, the path is `$HOME/.ssh/authorized_keys`

7. Connecting through SSH using our keys

```
ssh -i ~/.ssh/id_rsa user@our_host_ip
```

### SSH Agent
We already know how to use keys in order to connect through Secure Shell, but, there is an issue, it requires unlocking private key with a secret passphrase upon each connection. 

To avoid this, we need to use `ssh-agent`, a program that runs in background and stores your keys in memory. 

1. Enable ssh-agent

```
# start the ssh-agent in the background
$ eval "$(ssh-agent -s)"
Agent pid 69599
```

2. Add the SSH key to the ssh-agent

```
$ ssh-add ~/.ssh/id_rsa
```
If you named your key differently, you need to replace `id_rsa` with your key name. 


Now you can connect to the host, it will ask for passphrase just once. Your next connection would be passphrase free. 

```
ssh -i ~/.ssh/id_rsa user@our_host_ip
```
By default ssh use id_rsa key, so we can avoid `-i ~/.ssh/id_rsa` so, we can use

```
ssh user@our_host_ip
```

### SSH agent forwarding

The previous approach looks like the perfect solution, but what happens if we want to connect to any service that requires our ssh authentication. 

Let's say, we need to pull a repo in the remote host using our github keys. We need to repeat previous process but on the host machine. It would be risky if anyone grants access to our host because he could retrieve our private keys and has access to private configured services. 

To avoid the previous scenario, we are going to use our final resource: SSH agent forwarding. 

>SSH agent forwarding can be used to make deploying to a server simple. It allows you to use your local SSH keys instead of leaving keys (without passphrases!) sitting on your server.

Let's configure and test SSH forwarding using github as remote service to pull our code into the host.

#### Setting up SSH agent forwarding

Remember that we need to already have our key associated with any github repo. 

So, let's configure:

1. Create or open up the file at `~/.ssh/config`
2. Enter the following text, replacing myhost.com with our server domain name or IP 

```
Host myhost.com
  ForwardAgent yes
```

There is a shortcut to archive this, if we don't want to create a config file, we have another option, using `-A` flag with the `ssh` command.

```
ssh -A user@myhost.com 
```

>-A option enables forwarding of the authentication agent connection.
>
It means that, it forwards your SSH auth schema to the remote host. So you can use SSH over there as if you were on your local machine. 

#### Testing SSH agent forwarding

In order to test if our agent forwarding is working, let's ssh into our remote host and test it out. 

```
$ ssh app@myhost.com
app@debian:~$ ssh -T git@github.com
Hi levivm! You've successfully authenticated, but GitHub does not provide shell access.
```
It works.  Now we can pull/push against any github repo that we have access to it.

If there something wrong, please, let me know. 

Note: you don't forward the key itself, you forward the agent, so basically, you can add many keys as you want. In case your github repo use different keys.

You can check here how to do it
https://superuser.com/questions/1140830/ssh-agent-forwarding-using-different-usernames-and-different-keys 
