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
