# Personalized sSH

## What's this?

This is a wrapper around SSH which will sends commands specified in `~/.psh_profile` to a remote machine, as
soon as you've logged in. Usual use case - have your own set of bash aliases
everywhere, not just locally.

Written in Perl just because I like Perl. If you like some other programming
language - feel free to port PSH, and let me know so I can buy you a pint.

## Installation

* clone this repo somewhere:

```
    git clone https://github.com/br0ziliy/psh.git
```

* install some dependencies (available in standard Fedora/CentOS repositories,
    if you use Debian - open an issue or create a pull request with the
    documentation update):

```
    yum install perl-Expect perl-TermReadKey
```

* update your PATH with the location of psh script
* create your `.psh_profile`:

```
    cat >> ~/.psh_profile<<EOF
    export TERM=linux
    alias p='ps axfuww' \
    sf='iptables -L -n -v' \
    sfn='iptables -L -n -v -t nat'
    # and so on, basically you can just add the contents of your .bashrc / .bash_profile
    printf "\033k`hostname -s`\033\\" # sets tmux/screen current window title to the server hostname
    reset
    EOF
```

Remember, whatever you put to `.psh_profile` will be executed on the remote
machine right after the login.

## Usage and examples

PSH has a nice feature of searching hosts.
The first time you log in to a machine, it adds the full hostname to
`~/.psh_inventory` file. Later on you can log in to that host using just part of
the name.
You can add the hostnames to the inventory file yourself:

```
cfengine.dc1.example.com
cfengine.dc2.example.com
cfengine.dc3.example.com
cfengine.dc4.example.com
cfengine.dc5.example.com
webapp01.dc1.example.com
db01.dc1.example.com
db02.dc4.example.com
```

With the above, when you run `psh cfengine`, you'll get output similar to the
below:

```
$ psh cfengine
Choose the server to connect to:
        0 -> cfengine.dc1.example.com
        1 -> cfengine.dc2.example.com
        2 -> cfengine.dc3.example.com
        3 -> cfengine.dc4.example.com
        4 -> cfengine.dc5.example.com
> 
```

And with `psh dc1` you'll see:

```
$ psh dc1
Choose the server to connect to:
        0 -> cfengine.dc1.example.com
        1 -> webapp01.dc1.example.com
        2 -> db01.dc1.example.com
>
```

Input the number - and whoila, you're at the needed server.
Remeber, the host list is taken from the `~/.psh_inventory` file.

If you want to pass additional parameters to `ssh` command - just append it to
the `psh` command after the hostname:

```
$ psh cfengine -l deploy -K
Choose the server to connect to:
        0 -> cfengine.dc1.example.com
        2 -> cfengine.dc2.example.com
        2 -> cfengine.dc3.example.com
        3 -> cfengine.dc4.example.com
        4 -> cfengine.dc5.example.com

> 4
```

In the example above, psh will run `ssh root@cfengine.dc5.example.com -l deploy -K` (`root@` part is hardcoded for now, send me a pull request to fix it :) )

## Known Iusses

Due to me being lazy to implement proper argument parsing with `GetOpts`, there're couple of
issues you should keep in mind:

- `root@` part is hardcoded in the ssh command (can be overridden with `-l otheruser`) parameter
- hosts with ssh listening on port other than 22 are not supported yet (psh will
    report `No hosts found, exiting` for those)
