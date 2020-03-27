# zsh-bin

> Statically-linked, hermetic, relocatable Zsh.

- [Installation](#installation)
- [Compiling](#compiling)
- [How it works](#how-it-works)
- [Supported platforms](#supported-platforms)
- [Why?](#why)
- [No, seriously, why?](#no-seriously-why)
- [Limitations](#limitations)

## Installation

To install the latest version of Zsh, run the following command:

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/romkatv/zsh-bin/master/install)"
```

Or, if you don't have `curl`:

```sh
sh -c "$(wget -O- https://raw.githubusercontent.com/romkatv/zsh-bin/master/install)"
```

Or, if you prefer, follow [manual installation](#manual-installation) instructions.

Add `~/.zsh-bin/bin` to `PATH` and type `zsh` to start Zsh.

*Tip*: `install` has a few optional flags. Invoke it with `-h` to list them.

### Manual installation

```sh
kernel=$(uname -s | tr '[A-Z]' '[a-z]')
arch=$(uname -m | tr '[A-Z]' '[a-z]')
name="zsh-5.8-${kernel}-${arch}"
curl -fsSLO -- "https://github.com/romkatv/zsh-bin/releases/download/v3.0.0/${name}.tar.gz"
tar -xzf "$name".tar.gz
rm "$name".tar.gz
rm -rf ~/.zsh-bin
mv "$name" ~/.zsh-bin
~/.zsh-bin/share/zsh/5.8/scripts/relocate
```

The [install script](https://github.com/romkatv/zsh-bin/blob/master/install) has more robust logic
for determining which archive to download. See its source code if the simple algorithm listed
above gives you HTTP 404.

If you move or rename `~/.zsh-bin`, you'll need to call `share/zsh/5.8/scripts/relocate/relocate`
afterwards. See the last line in the instructions above.

## Compiling

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/romkatv/zsh-bin/master/build)"
```

*Tip*: `build` has a few optional flags. Invoke it with `-h` to list them.

On Linux build is done in a Docker container, so you'll need to install docker first. On non-Linux
systems build is done on the host. In the latter case it's recommended to run the script in a
freshly installed OS.

If everything goes well, `zsh-5.8-${KERNEL}-${ARCH}.tar.gz` will appear in the current directory.
This archive contains statically-linked, hermetic, relocatable Zsh 5.8. Installation of Zsh from the
archive doesn't require libc, ncurses, pcre, terminfo database or root access. As long as the target
machine has a compatible CPU and kernel, it'll work.

You can find built archives in [releases](https://github.com/romkatv/zsh-bin/releases).

The build script stores source code tarballs that have been used during compilation in `./src`. If
you run `build` again, it'll use these tarballs after verifying that their content is as expected.
This way you avoid downloading the same tarballs over and over again when running `build` multiple
times.

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
  magic marker, trailing p10k totally accidental ;-)    |              |
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

The build script currently works on Linux, macOS, FreeBSD, Cygwin and MSYS2. Prebuilt archives for
popular CPU architectures can be found in [releases](https://github.com/romkatv/zsh-bin/releases).

You can use `zsh-5.8-linux-x86_64.tar.gz` on WSL but you cannot run the build script there.

## Why?

Assuming that you want to use Zsh 5.8 (who doesn't, right?), ideally you would install it with the
official package manager for your OS. If your OS doesn't provide an option to install Zsh 5.8,
or you don't have root access to install it, you'll need to look for alternative installation
methods.

The next thing you can try is to [build Zsh from source](
  https://github.com/zsh-users/zsh/blob/master/INSTALL) on the target machine. This method allows
you to install any version of Zsh to any directory. If you don't have root access, you can choose to
install Zsh to your home directory. However, if some of the tools necessary for building Zsh are
missing (autoconf, make, gcc, yodl, ncurses, etc.), this option is also out.

If you have access to another machine with compatible CPU, kernel and runtime, and with all
necessary build tools, you can compile Zsh there and copy build artifacts to the target
machine. If you place all files in the same location and set a few custom environment variables, it
should work.

Or you can download a 3MB archive from zsh-bin, extract it, and enjoy Zsh 5.8.

## No, seriously, why?

I `ssh` to servers through a Bash wrapper script that automatically copies my admin tools and shell
configs from the local machine to remote. Here's the gist of it:

```zsh
#!/usr/bin/env bash
#
# Usage: ssh.bash [ssh-options] [user@]hostname

set -ueo pipefail
dump=$(tar -C ~ -pcz -- .bashrc admin-scripts | base64)
ssh -t "$@" "echo '$dump'" | base64 -d | tar -C ~ -pxz  && exec bash -il'
```

It archives a few local files and runs a command over SSH. This command extracts files from
the archive and starts Bash. Pretty simple.

I'm using Zsh locally but Bash remotely. I don't install Zsh on servers as it's not necessary for
running things. Some of the servers are also tricky to get Zsh onto. For example, network routers
running EdgeOS.

In March of 2020 an [announcement](
  https://www.reddit.com/r/zsh/comments/fiq9w2/bring_zsh_with_ohmyzsh_wherever_you_go_through/) was
posted on [/r/zsh](https://www.reddit.com/r/zsh/). It mentioned that "xxh uses the portable
version of Zsh". I thought it would be cool to migrate my `ssh.bash` script to Zsh and install
the portable version of Zsh on the remote machine if there isn't one already installed (this is
basically what [xxh](https://github.com/xxh/xxh) does).

This worked in some cases but not always as the version of Zsh from xxh turned out not portable
enough for my needs. I set out to build a more portable alternative and created zsh-bin. Since it
works for me, I figured it might be of use to others.

My public standalone `ssh.zsh` script is [here](
  https://github.com/romkatv/dotfiles-public/blob/master/bin/ssh.zsh). See comments at the top if
you are curious.

## Limitations

The build script doesn't work if `/bin/sh` is bash v4.4 or older. Use a newer version of bash or
a different interpreter. Try `zsh`, `dash` and `ash`. You might have one of these already installed.

This limitation can be removed but the motivation is rather low for doing this. There is no need
for the build script to be super portable. The install script and `relocate` are a different matter.
They must be very portable and they are. They work on older versions of bash just fine.

---

Not all zsh modules are enabled on all platforms:

- `zsh/db_gdbm` is enabled only on Linux.
- `zsh/attr` is disabled on FreeBSD.
- `zsh/pcre` is disabled on Cygwin.

This can be fixed. Please open an issue or better yet send a PR if you care.

---

The build script requires certain software to be installed by the user. For example, on Linux it
needs Docker but cannot install it on its own. When you run `build`, it'll tell you what's missing.

---

Builds are done natively, meaning that the target kernel and CPU architecture must be the same as
on the host. Given a Linux-x86_64 host, you can build Zsh for Linux-x86_64 and Linux-i686 but
not for Linux-aarch64 or Darwin-x86_64.

All archives in [releases](https://github.com/romkatv/zsh-bin/releases) are produced by [mbuild](
  https://github.com/romkatv/zsh-bin/blob/release/mbuild). This script builds Zsh on remote machines
over SSH. It's not an officially supported script, so please don't expect it to be stable or well
documented.

---

Zsh from zsh-bin doesn't read global rc files from anywhere. It does read user rc files of course.

This can be changed. An empty `etc` directory could be added to the archive, from which Zsh would
source `zshenv` and similar files if they exist. Please open an issue or better yet send a PR if you
care.

---

The build script doesn't know how to build man pages and help files on macOS. The problem is that
there is no `yodl` for macOS and porting it is a daunting task. To get out of this conundrum `build`
pulls man pages and help files from `zsh-5.8-linux-x86_64.tar.gz` and embeds them in
`zsh-5.8-darwin-x86_64.tar.gz`. So if you are trying to reproduce the macOS build, you'll need to
start by building Zsh for Linux-x86_64.
