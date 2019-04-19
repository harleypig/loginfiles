# Login Files

Thanks to
[flowbok](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html)'s
excellent article I have a seemingly very good idea of how start up files are
handled on the various systems I am on.

But I need a bit more concrete example set. So, this is my attempt at those
examples.

## Setup

You will need access to a system to which you have root access.

* Create a test user (I used 'testloginfiles'). How to do that on your system
   is beyond the scope of this project.

* Modify `/etc/profile` and add the following as the first executable line
  (replacing 'testloginfiles' with the username you created).

    test $USER == 'testloginfiles' && echo "/etc/profile (-: $-) ($(shopt login_shell))"

* Modify `/etc/bash.bashrc` and add the following as the first executable line
  (replace 'testloginfiles' with the username you created).

    [[ $USER == 'testloginfiles' ]] && echo "/etc/bash.bashrc (-: $-) ($(shopt login_shell))"

* Create `.bash_profile`, `.bashrc`, `.profile` and `test.sh` in your users home
  directory. They should each contain only one line, replacing the filename
  for each, as follows. Make sure `test.sh` is executable.

    echo "<filename> (-: $-) ($(shopt login_shell))"

## Startup Methods

There are many, many ... many login methods. I will list them here as
I discover them and list the results as I discover the results.

### Terminal

* login (from terminal)

### SSH

* ssh user@host
* ssh user@host command
* ssh user@host -t command

### su

* su user
* su -l user

### sudo

## Bash

## Sh

On modern systems, `sh` is a link to `bash` which tries to emulate the
original `sh` execution.
