# zsh-bin

> Statically-linked, hermetic, relocatable Zsh.

## Installation

To install Zsh on x86_64 Linux box without root access, run the following commands:

```sh
mkdir -p ~/apps
cd ~/apps
curl -fsSLO https://github.com/romkatv/zsh-bin/releases/download/v1.0.0/zsh-5.8-linux-x86_64-static.tar.gz
tar -xzf zsh-5.8-linux-x86_64-static.tar.gz
zsh-5.8-linux-x86_64-static/share/zsh/5.8/scripts/relocate
rm -f zsh-5.8-linux-x86_64-static.tar.gz
```

*Note*: If there is no `curl` on your machine, try replacing `curl -fsSLO` with `wget`.

After that you can invoke `~/apps/zsh-5.8-linux-x86_64-static/bin/zsh` and it'll just work,
although you'll probably want to add `~/apps/zsh-5.8-linux-x86_64-static/bin` to `PATH` for
convenience.

Zsh won't read or write any files outside of its installation directory --
`~/apps/zsh-5.8-linux-x86_64-static` in the example above -- with the exception of per-user rc files
such as `${ZDOTDIR:-~}/.zshrc`. If you move or rename this directory, you'll need to call
`share/zsh/5.8/scripts/relocate/relocate` afterwards. This script reconfigures Zsh so that it loads
builtin functions from the new location.

## Compiling

```sh
git clone https://github.com/romkatv/zsh-bin.git
cd zsh-bin
./build-zsh-5.8-static
```

The build script requires Docker and internet connection.

If everything goes well, `zsh-5.8-linux-x86_64-static.tar.gz` will appear in the current directory.
This archive contains statically-linked, hermetic, relocatable Zsh 5.8. Installation of Zsh from the
archive doesn't require libc, terminfo, ncurses or root access. As long as the target machine has a
compatible CPU and Linux kernel, it'll work.

You can find built archives in [releases](https://github.com/romkatv/zsh-bin/releases).

## Why?

Assuming that you want to use Zsh 5.8, ideally you'll want to install it with the official package
manager for your OS. If your OS doesn't provide an option to install Zsh 5.8, or you don't have root
access to install it, this option is out.

The next thing you can try is to [build Zsh from source](
  https://github.com/zsh-users/zsh/blob/master/INSTALL) on the target machine. This method allows
you to install any version of Zsh to any directory. If you don't have root access, you can choose to
install Zsh to your home directory. However, if some of the tools necessary for building Zsh are
missing (autoconf, make, gcc, yodl, ncurses, etc.), this option is also out.

If you have access to another machine with compatible CPU, kernel and runtime, and with all
necessary build tools, you can compile Zsh there and copy build artifacts to the target
machine. If you place all files in the same location and set a few custom environment variables, it
should work.

If all else fails, zsh-bin is here for you. Download a 3MB archive, extract it, and enjoy Zsh 5.8.

## No, seriously, why?

I `ssh` to servers through a Bash wrapper script that automatically copies my admin tools and shell
configs from the local machine to remote. Here's the gist of it:

```zsh
#!/usr/bin/env bash
#
# Usage: ssh.bash [ssh-options] [user@]hostname

set -ueo pipefail
dump=$(tar -C ~ -pcz -- .bashrc admin-scripts | base64)
ssh -t "$@" 'echo "'$dump'" | base64 -d | tar -C ~ -pxz  && exec bash -il'
```

It archives a few local files and then runs a command over SSH. This command extracts files from
the archive and starts Bash. Pretty simple.

I'm using Zsh locally but Bash remotely. The reason being that I don't install Zsh on servers as
it's not necessary for running things. Some of the servers are also tricky to get Zsh onto. For
example, network routers running EdgeOS.

In March of 2020 an [announcement](
  https://www.reddit.com/r/zsh/comments/fiq9w2/bring_zsh_with_ohmyzsh_wherever_you_go_through/) was
posted on [/r/zsh](https://www.reddit.com/r/zsh/). It mentioned that "xxh uses the portable
version of Zsh". I thought it would be cool to migrate my `ssh.bash` script to Zsh and install
"the portable version of Zsh" on the remote machine if there isn't one already installed (this is
basically what [xxh](https://github.com/xxh/xxh) does).

This worked in some cases but not always as the version of Zsh from xxh turned out not portable
enough for my needs. I set out to build a more portable alternative and created zsh-bin. Since it
works for me, I figured it might be of use to others.
