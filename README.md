# zsh-bin

> Statically-linked, hermetic, relocatable Zsh.

- [Installation](#installation)
- [Compiling](#compiling)
- [How it works](#how-it-works)
- [Supported platforms](#supported-platforms)
- [Why?](#why)
- [No, seriously, why?](#no-seriously-why)

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

## How it works

A regular build of Zsh cannot be transplanted to another machine due to having dependencies on
system files and hard-coded absolute paths to Zsh's own autoloadable functions and scripts. This
section explains how zsh-bin solves these problems.

zsh-bin uses fully static linking to avoid dependencies on dynamic libraries and program loader. It
includes an extensive terminfo database that it falls back to if there is no suitable entry
for `$TERM` in the system database. These two measures remove all dependencies on files outside of
zsh-bin.

The main `zsh` binary in zsh-bin contains hard-coded absolute paths to autoloadable functions and
scripts, just like in the regular build of Zsh, but they are laid out in such a way as to allow for
replacement through binary patching. In a nutshell, when you build Zsh normally, the C source code
contains something like this:

```c
#define FPATH_DIR "/usr/share/zsh/5.8/functions"
```

If you start `zsh` and check the value of `fpath` parameter, you'll see that it contains
`/usr/share/zsh/5.8/functions`. This comes from `FPATH_DIR` that got fixed during compilation.

When you build Zsh with zsh-bin scripts, the C code looks a bit different:

```c
#define FPATH_DIR_TAG ":iLWDLaG9dUlsxzEQp10k:fpath:"
#define FPATH_DIR ((const char *)(tagged_fpath_dir + sizeof(FPATH_DIR_TAG) - 1))
volatile char tagged_fpath_dir[sizeof(FPATH_DIR_TAG) + 4096] = {
  FPATH_DIR_TAG "/usr/share/zsh/5.8/functions"
};
```

`FPATH_DIR` still resolves to `/usr/share/zsh/5.8/functions`, so `fpath` parameter has the same
value as before. What's different is the content of `zsh` binary.

Regular `zsh` binary:

```text
                  FPATH_DIR points here
                            |
                            v
????????????????????????????/usr/share/zsh/5.8/functions·???????????????????????
            ^                                           ^              ^
            |                                           |              |
            |                                     NUL terminator       |
            |                                                          |
            +---------- other data and code----------------------------+
```

`zsh` from zsh-bin:

```text
                  FPATH_DIR points here
                            |
                            v
:iLWDLaG9dUlsxzEQp10k:fpath:/usr/share/zsh/5.8/functions·***********************
            ^                                           ^              ^
            |                                           |              |
  magic marker, trailing 10k totally accidental ;-)     |              |
                                                        |              |
                                                  NUL terminator       |
                                                                       |
                                enough space for a 4096-character-long directory
```

Now it's possible to "relocate" autoloadable functions by finding `:iLWDLaG9dUlsxzEQp10k:` inside
`zsh` and writing a new directory after it. `relocate` script included in zsh-bin does just
that. It's written in POSIX sh, so it'll run anywhere. Here's the relevant part of `relocate`
(simplified):

```sh
magic=iLWDLaG9dUlsxzEQp10k
bin=$(LC_ALL=C tr -c '[:alnum:]:' ' ' <"$zsh")
prefix="${bin%:$magic:fpath:*}:$magic:fpath:"
dd if=/dev/zero of="$zsh" bs=1 seek=${#prefix} count=4096 conv=notrunc
echo "$new_fpath_dir" | dd of="$zsh" bs=1 seek=${#prefix} count=${#dir} conv=notrunc
```

## Supported platforms

`build-zsh-5.8-static` has been tested only x86_64 Linux. In theory it should work with other
popular CPU architectures.

There is currently no support for kernels other than Linux. You cannot run the build script on WSL
but you can use `zsh-5.8-linux-x86_64-static.tar.gz` there.

## Why?

Assuming that you want to use Zsh 5.8 (who doesn't, right?), ideally you would install it with the
official package manager for your OS. If your OS doesn't provide an option to install Zsh 5.8,
or you don't have root access to install it, this option is out.

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
