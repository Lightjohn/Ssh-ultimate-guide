# The ultimate ssh tips & tricks 
(Or how a personnal memo became ultimate)

Here a list of tips on SSH that will make your life easier when using SSH.
I will try to add only convenient and useful information, so no complicated 
and sneaky stuff here.

I will not talk too much of the really first steps to install/use ssh; this 
guide is to enhance your daily use with SSH.

# SSH keys or password-less connection

Before anything we will set some keys, Why? Because it's **CONVENIENT** and as a 
bonus, it's secure.

```
ssh-keygen
```

OR

```
ssh-keygen -b 4096 
```
If you are paranoid (it will create a stronger key), then press enter until you see a nice ascii art image
(you can enter a passphrase if you are not in a secure environnment).

Now in your home you should see:

```
.ssh/
├── id_rsa
├── id_rsa.pub
```

## Now what?

*  Method 1 (to understand what happen):

   To use the keys, connect to the distant machine using
   
   ```
   ssh user@distant-machine
   ```
   
   and:
   
   ```
   cd .ssh // Create it if it does not exist
   nano authorized_keys
   ```
   
   Then paste the content of **id\_rsa.pub** inside authorized_keys
   To quit: *Ctrl-x* then *y* then press *Enter*

*  Method 2 (for the next time):

   ```
   ssh-copy-id -i ~/.ssh/id_rsa.pub user@distant-machine
   ```

   or if you are on mac:

   ```
   cat ~/.ssh/id_rsa.pub | ssh user@distant-machine 'cat - >> ~/.ssh/authorized_keys'
   ```

Now you can close the session to the remote machine and try again to connect.
If you are connected without being asked for a password, everything is good else continue reading.

## It's not working

SSH is very picky and not very talkative on the permissions so do the following 
steps on every machine! 

Do not hesitate to create the files if they don't exist: `touch ~/.ssh/config ~/.ssh/authorized_keys`


```
chmod 744 ~/.ssh
chmod 644 ~/.ssh/config
chmod 644 ~/.ssh/authorized_keys
chmod 644 ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/id_rsa
```

To summarize: you can let people access (read, never write) the conf files but your 
private key **MUST** be read only by you.

This step can solve many of your problems.

# The main dish: the conf file

What if I say that **EVERY** option you are using for your daily usage can be 
saved so that you can reduce your ssh command to nearly nothing?

```
ssh -X -p 2222 MyUser@machine.domain.com
``` 

can become:

```
ssh machine
``` 

Ready? Let's modify the ~/.ssh/config file in your local home. For that you 
can use any text editor you want.

An SSH config file works like this:

```
Host machine
   optionName1 valueName1
   optionName2 valueName2
```

First, if you want to apply an option for every connection:

```
Host *
   Port 22
   User MyUser
```

Here we specify that every connection will be on **port 22** and the user will be **MyUser**. Not bad.

Then let's go back to our example and create a host for this case

```
Host machine
   HostName machine.domain.com
   Port 2222
   ForwardX11 yes
```

Now you can just `ssh machine` because the username is specified for every connection and the other parameters are set specifically for this machine.

Here some options that are really worth knowing:

`ForwardAgent yes/no`: very useful if you are going from one machine to another; you will jump from one machine to another without ever being asked for a password 
(if the previous steps are done correctly: `~/.ssh/authorized_keys` is on every server)

**NOTE**: Do not activate this option on any server, only on those you trust.

`IdentityFile ~/.ssh/id_rsa`: You can specify a specific key for a particular host . 

### Warning: Powerful options here:
`LocalForward 5900 127.0.0.1:5900`: This option can possibly let you do so many things: Let's imagine you want to control a machine that can only be reached through SSH with VNC, add this option and after you can just: `vnc localhost` and 
it will work: YOU are easily redirecting the distant port to your local port.

**Note**: In `LocalForward AAAA 127.0.0.1:BBBB`, AAAA is your local port and BBBB is the remote port.

Another command: Imagine you want to connect to HostB but only HostA is available through internet.

```
Host HostA
  HostName HostA.example.com

Host HostB
  ProxyCommand ssh -q HostA nc -q0 HostB.example.com 22
```

And now you can forget about HostA and just do `ssh HostB`.

# SSH Tools or software you may want to know

## ssh-agent

Earlier I showed you the command `ForwardAgent yes/no`. It will store your password (if you put one on your key) and you will never be asked again. There is another good point, if **ForwardAgent** is activated, it will keep your key across the sessions. To add a key `ssh-add ~/.ssh/id_rsa`.

## sshfs
You will be able to mount the remote in a local folder and explore the remote machine's hard drive like if it was a external hard drive!

First install *sshfs* then create an empty folder:

```
mkdir tmp
```

If you have configured your `.ssh/config` (at this point, I hope) then:

```
sshfs server: tmp
```

And now in tmp you have your temote home. Wonderful!

To unmount just enter:

```
fusermount -u tmp
```

## rsync

Why is rsync here? Rsync is a backup/synchronisation tool. What matters here is 
it can use ssh to synchronize your data over the internet:

```
rsync machine:remoteFolder/ localFolder/
```

## web proxy

You are in an insecure place but want to access the internet? Ssh can do it for you:

```
ssh -D 8888 machine
```

Will open a SOCKS proxy on localhost.

Now in Chrome/Firefox/what you want, add a proxy (I use FoxyProxy plugin on Firefox/Chrome) with this information:

```
Proxy adress: localhost:8888
Is SOCKS: yes
SOCKS version: 5
```

Don't forget to choose this new proxy in FoxyProxy and now all your future connections (except DNS queries) will go through *machine* in an encrypted way.

# Note

* Quickly execute a SSH command:
	
	There is a way to just send a command to the server and get the result locally.
	
	Instead of:

	```
	ssh machine
	command here
	exit
	```

	you can simply:

	```
	ssh machine "command here"
	```


* To finish a little note:
	
	When you are using scp or rsync you do something like this:
	
	```
	scp -r machine:remoteFolder/ localFolder/
	```
	
	The meaning of **machine:** is that you are pointing to your home.
	
	So 
	
	```
	scp -r machine:remoteFolder/ localFolder/
	```
	
	is equivalent to:
	
	```
	scp -r machine:/home/you/temoteFolder/ localFolder/
	```
	
Hope you have learned something.
