# Login Files

Thanks to
[flowbok](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html)'s
excellent article I have a seemingly very good idea of how start up files are
handled on the various systems I am on.

But I need a bit more concrete example set. So, this is my attempt at those
examples.

[flowbok's repo](https://bitbucket.org/flowblok/shell-startup) which contains
his basic startup files.

## Setup

You will need access to a system to which you have root access in order to
fully test these logins.

* Create a test user (I used 'testloginfiles'). How to do that on your system
  is beyond the scope of this project.

* Modify `/etc/profile` and add the following as the first executable line
  (replacing 'testloginfiles' with the username you created).

    test $USER == 'testloginfiles' && echo "/etc/profile (-: $-) ($(shopt login_shell))"

* Modify `/etc/bash.bashrc` and add the following as the first executable line
  (replace 'testloginfiles' with the username you created).

    [[ $USER == 'testloginfiles' ]] && echo "/etc/bash.bashrc (-: $-) ($(shopt login_shell))"

* Create `.bash_profile`, `.bashrc`, `.profile` and `test.sh` in your users
  home directory. They should each contain only one line, replacing the
  filename for each, as follows. Make sure `test.sh` is executable.

    echo "\<filename\> (-: $-) ($(shopt login_shell))"

### System Information

These tests were run in an Arch Linux environment.

## Summary

##### login (from terminal)
##### ssh user@host
##### su - user
##### su -l user
##### su --login user
##### sudo -i -u user

* interactive
* login shell
* `/etc/profile`
* `/etc/bash.bashrc`
* user's `.bash_profile`

##### ssh user@host command
##### ssh user@host -t command
##### su -c command user
##### sudo -u user command

* non-interactive
* no login shell
* no startup files executed

##### su user

* interactive
* no login shell
* `/etc/bash.bashrc`
* user's `.bashrc`

##### su - user -c command
##### su -l user -c command
##### su --login user -c command
##### sudo -i -u user command

* non-interactive
* login shell
* `/etc/profile`
* user's `.bash_profile`

Note: After user's `.bash_profile` is executed it appears that `login_shell`
is turned off for the sudo command.

## Startup Methods

There are many, many ... many login methods. I will list them here as
I discover them and list the results as I discover the results.

### Terminal

#### login (from terminal)

##### Shell set to /bin/bash

    sweetums login: testloginfiles
    Password:
    Last login: Thu May  2 09:45:06 on tty2
    /etc/profile (-: himBH) (login_shell        on)
    /etc/bash.bashrc (-: himBH) (login_shell    on)
    /home/testloginfiles/.bash_profile (-: himBH) (login_shell      on)
    [testloginfiles@sweetums ~]$

##### Shell set to /bin/sh

Logging in from the terminal produces:

    sweetums login: testloginfiles
    Password:
    Last login: Thu May  2 09:45:06 on tty2
    /etc/profile (-: himBH) (login_shell        on)
    /etc/bash.bashrc (-: himBH) (login_shell    on)
    /home/testloginfiles/.bash_profile (-: himBH) (login_shell      on)
    [testloginfiles@sweetums ~]$

### ssh

#### ssh user@host

This command gives us the same results as a terminal login:

    $ ssh testloginfiles@localhost
    testloginfiles@localhost's password:
    Last login: Thu May  2 09:46:24 2019
    /etc/profile (-: himBH) (login_shell            on)
    /etc/bash.bashrc (-: himBH) (login_shell        on)
    /home/testloginfiles/.bash_profile (-: himBH) (login_shell      on)
    [testloginfiles@sweetums ~]$

#### ssh user@host command

This runs the command (which must be a fully qualified path to the
command) and returns to the caller.

    $ ssh testloginfiles@localhost /home/testloginfiles/test.sh
    /home/testloginfiles/test.sh (-: hB) (login_shell       off)

#### ssh user@host -t command

This runs the command (which must be a fully qualified path to the
command) and returns to the caller.

    ssh -t testloginfiles@localhost /home/testloginfiles/test.sh
    /home/testloginfiles/test.sh (-: hB) (login_shell       off)
    Shared connection to localhost closed.

### su

#### su user

This method (partially?) preserves the calling environment. The `harleypig` at
the end means we're still in the calling users home directory.

From the `su` man page:

> For backward compatibility, su defaults to not change the current directory
> and to only set the environment variables HOME and SHELL (plus USER and
> LOGNAME if the target user is not root).

This means that your existing environment will be accessible from `user`'s
environment. This is a possible security risk, so you might want to keep it in
mind when using this method.

    $ su testloginfiles
    Password:
    /etc/bash.bashrc (-: himBH) (login_shell        off)
    /home/testloginfiles/.bashrc (-: himBH) (login_shell            off)
    [testloginfiles@sweetums harleypig]$

#### su -c command user

This method does not have the same problem as above.

    su -c /home/testloginfiles/test.sh testloginfiles
    Password:
    /home/testloginfiles/test.sh (-: hB) (login_shell       off)

#### su - user
#### su -l user
#### su --login user

These methods gives us the same result as a terminal login.

    $ su - testloginfiles
    Password:
    /etc/profile (-: himBH) (login_shell            on)
    /etc/bash.bashrc (-: himBH) (login_shell        on)
    /home/testloginfiles/.bash_profile (-: himBH) (login_shell      on)
    [testloginfiles@sweetums ~]$

#### su - user -c command
#### su -l user -c command
#### su --login user -c command

These methods gives us these results.

    $ su - testloginfiles -c /home/testloginfiles/test.sh
    Password:
    /etc/profile (-: hBc) (login_shell      on)
    /home/testloginfiles/.bash_profile (-: hBc) (login_shell        on)
    /home/testloginfiles/test.sh (-: hB) (login_shell       off)

#### Notes

There is not supposed to be a difference between `-`, `-l` and `--login` but
the man page for `su` warns against using the shortcuts, so we should probably
test all three options.

If the need for the remaining options appears I'll add them. Pull requests
will be considered as well.

test options --pty, --preserve-environment, --shell, --session-command?

### sudo

#### sudo -i -u user

Same as a terminal login.

    $ sudo -i -u testloginfiles
    /etc/profile (-: himBH) (login_shell            on)
    /etc/bash.bashrc (-: himBH) (login_shell        on)
    /home/testloginfiles/.bash_profile (-: himBH) (login_shell      on)
    [testloginfiles@sweetums ~]$

#### sudo -u user command

    $ sudo -u testloginfiles /home/testloginfiles/test.sh
    /home/testloginfiles/test.sh (-: hB) (login_shell       off)

#### sudo -i -u user command

Not sure why this is, but `login_shell` is turned off after `.bash_profile` is
executed.

    $ sudo -i -u testloginfiles /home/testloginfiles/test.sh
    /etc/profile (-: hBc) (login_shell      on)
    /home/testloginfiles/.bash_profile (-: hBc) (login_shell        on)
    /home/testloginfiles/test.sh (-: hB) (login_shell       off)


## Bash

https://unix.stackexchange.com/questions/382984/the-zeroth-argument-to-a-command-executed-by-exec
https://unix.stackexchange.com/questions/435625/does-a-noninteractive-login-shell-execute-profile-or-a-file-whose-name-is/435626
https://unix.stackexchange.com/questions/tagged/bash+configuration
https://unix.stackexchange.com/questions/435744/how-can-we-start-bash-with-the-first-character-of-argument-zero-being

## Sh

On modern systems, `sh` is a link to `bash` which tries to emulate the
original `sh` execution.
