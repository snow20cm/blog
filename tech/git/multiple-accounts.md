+++
categories = ["git"]
date = "2015-07-24T20:53:03-06:00"
tags = ["git"]
title = "multiple accounts"

+++

## Open ssh-agent
You might need to start ssh-agent before you run the ssh-add command:

    eval `ssh-agent -s`
    ssh-add

Note that this will start the agent for msysgit Bash on Windows. If you're using a different shell or operating system, you might need to use a variant of the command,

## Create isa keys
Then, create isa keys for each account.

## create .ssh/config:

    Host myshortname realname.example.com
        HostName realname.example.com
        IdentityFile ~/.ssh/realname_rsa # private key for realname
        User remoteusername

    Host myother realname2.example.org
        HostName realname2.example.org
        IdentityFile ~/.ssh/realname2_rsa
        User remoteusername

## check

    ssh-add -l

This will list the added isa keys.

If nothing listed, use `ssh-add isa_key` to add the key.

