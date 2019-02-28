---
layout: post
title: How I setup my ssh config to be able to use different Gitlab/Github ssh keys.
---

## Backstory

So the original idea for this post actually stemmed from me adding some ssh keys for private gitlab repos to be able to git clone other private repos for building docker containers in our CI/CD stuff, but this was a perfect jump start for an issue that I had and how I solved it

## What the problem was.

Well, so you see, last week I setup a new ssh key to use the deploy key feature of Gitlab to give read only access to other computers, in this case it was a gitlab-ci runner, but thats beside the point. Also if you don't know ssh stuff and are still logging in every time you wanna clone/push to a remote git repo I suggest you head over here to a (great Digital Ocean breakdown of ssh connection education](https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process).

A few days after I added the key, upon a restart of my computer, I could no longer clone projects or push changes to remote repos with ssh. I know what it is, ssh is using my newer ssh key by default, lets test that theory!

If I typed `ssh -T git@gitlab.com` it would say what I expected, a failure right?:

```
➜  ~ ssh -T git@gitlab.com
Welcome to GitLab, @fergyfresh!
```

Damn! I was hoping that would fail. Let's try to clone a private repo again:

```
➜  ~ git clone ssh@gitlab.com:path/to/my/private/repo.git
Cloning into 'repo'...
ssh@gitlab.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
:( so sad. Well, that stinks. I'm lost, or am I?


## Solution

So I remember that I had used `~/.ssh/config` to specify an `Identity` before, so we can surely do that again. The file we are going to want to modify is `~/.ssh/config`. Or in most cases, like mine, it didn't even exist so I had to create it.

We are going to want to add an entry for a host and the ssh `IdentifyFile` to use. All of my ssh keys are in the `~/.ssh/` directory:

```
host gitlab.com
    HostName gitlab.com
    IdentityFile /home/ferg/.ssh/id_rsa
    User git
```

Now, once you see that you know the power of the ssh config and can extrapolate that setup to use a different ssh key for any number of hosts. Don't forget to have the path to the private key at `IdentifyFile` to be on your computer and not just blindly cut/paste my path, with MY username.


# Questions, comments, concerns

Feel free to click one of these buttons in order to signal me with something that was messed up with this. I'd be glad to fix anything that didnt work for you.
