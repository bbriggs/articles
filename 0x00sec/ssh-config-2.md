_This is the second in a series on ssh config files_

### Introduction

The previous article in this series was an introduction to the ssh config file, how it can be used, and a teaser of some of the possibilities. You've probably already figured out that it's quite easy to set common config strappings such as ports, usernames, hostnames, etc. from the ssh config. We're going to move beyond that and talk about really powerful tricks now. This will be focusing on ssh-agent forwarding. 

### What is the ssh-agent?

The following will mostly be a regurgitation and explanation of the the ssh-agent(1) [man page](https://linux.die.net/man/1/ssh-agent). 

>ssh-agent is a program to hold private keys used for public key authentication (RSA, DSA). The idea is that ssh-agent is started in the beginning of an X-session or a login session, and all other windows or programs are started as clients to the ssh-agent program.

What this means is that the ssh-agent spins up a little socket and stores your key information locally on your machine. When another machine needs to perform authentication, the auth request is forwarded back over SSH and is done locally on your machine and then the result is sent back. Private keys never get sent across the connection. Given that this is a socket exported as an environment variable, you can also see how it could easily be exploited. In fact, the man page has this to say:

> A unix-domain socket is created and the name of this socket is stored in the SSH_AUTH_SOCK environment variable. The socket is made accessible only to the current user. This method is easily abused by root or another instance of the same user.

So, you know, be careful and stuff.

How or why is this useful, you may ask? 

> The idea is that the agent is run in the user's local PC, laptop, or terminal. Authentication data need not be stored on any other machine, and authentication passphrases never go over the network. However, the connection to the agent is forwarded over SSH remote logins, and the user can thus use the privileges given by the identities anywhere in the network in a secure way.

You can forward your identity and authenticate to machines you can't otherwise directly connect to without having to store keys on any intermediary boxes.

Consider the following example:

You <---> Bastion box <---> private network

Now remember that you can't connect directly to the private network. You have to ssh to the bastion first, then the private net. How you you ssh into a machine on the private net using keys from your local laptop? You _could_ keep a copy of your keys on the box (or another set) and ssh around using those. However, we've all seen the danger that [loose ssh keys can cause](https://www.oreilly.com/learning/how-netflix-gives-all-its-engineers-ssh-access) (that's an awesome talk by the way. 100% recommend) 

So we reduce the exposure by using ssh-agent forwarding. 

### But fraq! How do I forward?

It's actually quite simple. You have to do two things (possibly three)

1. start the ssh-agent
1. add your keys to the ssh-agent using `ssh-add <key name>`
1. Specify in the config that you want to forward your agent

[Here's a good, short guide on getting that process started](http://sshkeychain.sourceforge.net/mirrors/SSH-with-Keys-HOWTO/SSH-with-Keys-HOWTO-6.html)

Now for setting the forwarding in your config: 

```
Host jumpbox
  IdentityFile ~/.ssh/id_rsa
  ForwardAgent yes
```
This tells your ssh client that when you `ssh bastion` you'll forward your ssh-agent (hopefully you remembered to start it!) and use the identity specified to connect to the jumpbox. Note, this is not telling the ssh-agent what keys to use. 

### Test it out

Go ahead and connect to your jumpbox and verify that your keys aren't on there by doing `ls ~/.ssh/`. Next, make sure you see `SSH_AUTH_SOCK` in your environment variables (just echo it). Once you have confirmed those two things, try to connect from the jumpbox to a machine that requires your ssh keys. You should be able to auth and log in from the jumpbox using the agent forwarded from your local machine. Magic!

### Further reading:

- https://linux.die.net/man/1/ssh-agent
- http://sshkeychain.sourceforge.net/mirrors/SSH-with-Keys-HOWTO/SSH-with-Keys-HOWTO-6.html
- https://developer.github.com/guides/using-ssh-agent-forwarding/
