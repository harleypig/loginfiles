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

    echo "<filename> (-: $-) ($(shopt login_shell))"

## Startup Methods

There are many, many ... many login methods. I will list them here as
I discover them and list the results as I discover the results.

### Terminal

* login (from terminal)

Logging in from the terminal produces:

    sweetums login: testloginfiles
    Password:
    Last login: Thu May  2 09:45:06 on tty2
    /etc/profile (-: himBH) (login_shell        on)
    /etc/bash.bashrc (-: himBH) (login_shell    on)
    /home/testloginfiles/.bash_profile (-: himBH) (login_shell      on)
    [testloginfiles@sweetums ~]$

### SSH

* ssh user@host

Running this command gives us the same results as a terminal login:

    $ ssh testloginfiles@localhost
    testloginfiles@localhost's password:
    Last login: Thu May  2 09:46:24 2019
    /etc/profile (-: himBH) (login_shell            on)
    /etc/bash.bashrc (-: himBH) (login_shell        on)
    /home/testloginfiles/.bash_profile (-: himBH) (login_shell      on)
    [testloginfiles@sweetums ~]$

* ssh user@host command

Running this runs the command (which must be a fully qualified path to the
command) and returns to the caller.

    $ ssh testloginfiles@localhost /home/testloginfiles/test.sh
    /home/testloginfiles/test.sh (-: hB) (login_shell       off)

* ssh user@host -t command

### su

* su user
  * interactive
  * non-login

This method (partially?) preserves the calling environment. The
`harleypig` at the end means we're still in the calling users home directory.

    $ su testloginfiles
    Password:
    /etc/bash.bashrc (-: himBH) (login_shell        off)
    Can't locate strict.pm:   lib/strict.pm: Permission denied at /usr/bin/vendor_perl/bash-complete line 7.
    BEGIN failed--compilation aborted at /usr/bin/vendor_perl/bash-complete line 7.
    /home/testloginfiles/.bashrc (-: himBH) (login_shell            off)
    [testloginfiles@sweetums harleypig]$

* su -c test.sh user
  * non-interactive
  * non-login

    su -c /home/testloginfiles/test.sh testloginfiles
    Password:
    /home/testloginfiles/test.sh (-: hB) (login_shell       off)

This method (partially?) preserves the calling environment. The
`harleypig` at the end means we're still in the calling users home directory.

    $ su testloginfiles -c bash pwd
    Password:
    bash: cannot set terminal process group (-1): Inappropriate ioctl for device
    bash: no job control in this shell
    /etc/bash.bashrc (-: hiBH) (login_shell         off)
    Can't locate strict.pm:   lib/strict.pm: Permission denied at /usr/bin/vendor_perl/bash-complete line 7.
    BEGIN failed--compilation aborted at /usr/bin/vendor_perl/bash-complete line 7.
    /home/testloginfiles/.bashrc (-: hiBH) (login_shell     off)
    [testloginfiles@sweetums harleypig]$

* su - user
* su -l user
* su --login user

* su user
* su -c user command

#### Notes

There is not supposed to be a difference between `-`, `-l` and `--login` but
the man page for `su` warns against using the shortcuts, so we should probably
test all three options.

If the need for the remaining options appears I'll add them. Pull requests
will be considered as well.

test options --pty, --preserve-environment, --shell, --session-command?

### sudo

## Bash

https://unix.stackexchange.com/questions/382984/the-zeroth-argument-to-a-command-executed-by-exec
https://unix.stackexchange.com/questions/435625/does-a-noninteractive-login-shell-execute-profile-or-a-file-whose-name-is/435626
https://unix.stackexchange.com/questions/tagged/bash+configuration
https://unix.stackexchange.com/questions/435744/how-can-we-start-bash-with-the-first-character-of-argument-zero-being

## Sh

On modern systems, `sh` is a link to `bash` which tries to emulate the
original `sh` execution.
