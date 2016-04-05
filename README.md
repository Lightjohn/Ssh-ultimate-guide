# The ultimate ssh tips & tricks 
(Or how a personnal memo became ultimate)

Here a list of tips on SSH that will make your life easier when using SSH.
I will try to add only convenient and useful information, so no complicated 
and sneaky stuff here.

I will not talk too much of the really first steps to install/use ssh, this 
guide is to enhance your daily use with SSH.

# SSH keys or passworld-less connection

Before anything we will set some keys, Why ? Because it's **CONVENIENT** and as a 
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

## Now what ?

*  Method 1 (to understand what happen):

   To use the keys, connect to the distant machine using
   
   ```
   ssh user@distant-machine
   ```
   
   and:
   
   ```
   cd .ssh // Create it if it does not exists
   nano authorized_keys
   ```
   
   Then paste the content of id_rsa.pub inside authorized_keys
   To quit: *Ctrl-x* then *y* then press *Enter*

*  Method 2 (for the next times):

   ```
   ssh-copy-id -i ~/.ssh/id_rsa.pub user@distant-machine
   ```

Now you can close the session to the distant machine and try again to connect.
If you are connected without being asked for a password, everything is good else go on reading.

## It's not working

ssh is very picky and not very talkative on the permissions so do the following 
steps on every machines ! 

Do not hesitate to touch the files if they dont exists: `touch ~/.ssh/config ~/.ssh/authorized_keys`


```
chmod 744 ~/.ssh
chmod 644 ~/.ssh/config
chmod 644 ~/.ssh/authorized_keys
chmod 644 ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/id_rsa
```

To sum: you can let people access (read, never write) the conf files but your 
private key **MUST** be read only by you.

This step can solve many of your problems.

# The main dish: the conf file

What if I say that **EVERY** options you are using for your daily usage can be 
saved so that you can reduce your ssh command to nearly nothing ?

```
ssh -X -p 2222 MyUser@machine.domain.com
``` 

can become:

```
ssh machine
``` 

Ready ? Let's modify the ~/.ssh/config file in your local home. For that you 
can use every text editor you want.

A SSH config file works like that:

```
Host machine
   optionName1 valueName1
   optionName2 valueName2
```

First if you want to apply option for every connections:

```
Host *
   Port 22
   User MyUser
```

here we specify that every connections will be on **port 22** and the user will me 
**MyUser**. Not bad.

Then let's go back to our example and create a host for this case

```
Host machine
   HostName machine.domain.com
   Port 2222
   ForwardX11 yes
```

Now you can just `ssh machine` because the username is specified for every 
connections and the others parameters are set specifically for this machine.

Here some options that are really worth to know:

`ForwardAgent yes/no`: very useful if you are going from one machine to another, 
you will jump from one machine to another without ever being asked for a password 
(if the previous steps are done: `~/.ssh/authorized_keys` on every servers)

`IdentityFile ~/.ssh/id_rsa`: you can specify for a particular host a specific 
key. 

### Warning: Powerful options here
`LocalForward 5900 127.0.0.1:5900`: This option can possibly let you so much 
things: let's imagine you want to control with VNC a machine that can only be 
reach through SSH, add this option and after you can just: `vnc localhost` and 
it will works: YOU are redirecting the distant port to your local port easily.

**Note**: in `LocalForward AAAA 127.0.0.1:BBBB`, AAAA is your local port and BBBB 
is the distant port.

Another command: Imagine you want to connect to HostB but only HostA is 
available through internet.

```
Host HostA
  HostName HostA.example.com

Host HostB
  ProxyCommand ssh -q HostA nc -q0 HostB.example.com 22
```

And now you can forget about HostA and just do `ssh HostB`.

# SSH Tools or softwares you may want to know

## ssh-agent

Earlier I show you the command `ForwardAgent yes/no`, it will store your 
password (if you put one on your key) and you will never be asked again. There 
is another good point, if **ForwardAgent** is activated, it will keep your key 
across the sessions. To add a key `ssh-add ~/.ssh/id_rsa`.

## sshfs

This tool is for you if you are working on an headless machine without the tools 
you like: you will be able to mount the distant in a local folder and explore the
distant machine hard drive like if it was a external har drive !

First install *sshfs* then create an empty foler:

```
mkdir tmp
```

If you have configured your `.ssh/config` (at this point, I hope) then:

```
sshfs server: tmp
```

And now in tmp you have your distant home. Wonderful !

To unmount just enter:

```
fusermount -u tmp
```

## rsync

Why is rsync here ? rsync is a backup/synchronisation tool, what matters here is 
it can use ssh to synchronize over the internet your data:

```
rsync machine:distantFolder/ localFolder/
```

# Note

To finish a little note:

When you are using scp or rsync you do something like that:

```
scp -r machine:distantFolder/ localFolder/
```

The meaning of **machine:** is that you are pointing to your home.

So 

```
scp -r machine:distantFolder/ localFolder/
```

is equivalent to:

```
scp -r machine:/home/you/distantFolder/ localFolder/
```


Hope you will be able to learn something.
