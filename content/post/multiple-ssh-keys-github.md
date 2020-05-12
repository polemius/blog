---
title: Use the multiple ssh keys with github
date: "2019-05-27T21:21:21"
description: The blog post about how configurate the github with multiple ssh keys
---

It is a quick note about how to handle multiple ssh keys for github.com.

First of all lets create two diffirent ssh keys:

```bash
$ ssh-keygen -t rsa -b 4096 -C "first_email@example.com" -f .ssh/first_key
$ ssh-keygen -t rsa -b 4096 -C "second_email@example.com" -f .ssh/second_key
```
 
After that create the config file in `.ssh` directory:

```javascript
$ touch .ssh/config
```

and put the configuration in:

```bash
Host first.github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/first_key

Host second.github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/second_key
```
	
Now you can use different keys depends of path.

```bash
$ ssh git@first.github.com
$ ssh git@second.github.com
```

