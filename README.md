# zsh-bin

> Statically-linked, hermetic, relocatable Zsh.

Usage:

```sh
./build-zsh-5.8-static
```
Creates an archive containing statically-linked, hermetic, relocatable Zsh 5.8. Installation of Zsh
from the archive doesn't require libc, terminfo, ncurses or root access. As long as the target
machine has a compatible CPU and Linux kernel, it'll work.

Running `./build-zsh-5.8-static` on x86_64 Linux will create `zsh-5.8-linux-x86_64-static.tar.gz` in
the current directory.

To install Zsh from the archive, copy it to the target machine's home directory and run:

```sh
mkdir -p ~/apps
cd ~/apps
tar -xzf ~/zsh-5.8-linux-x86_64-static.tar.gz
zsh-5.8-linux-x86_64-static/share/zsh/5.8/scripts/relocate
rm ~/zsh-5.8-linux-x86_64-static.tar.gz
```

After that you can invoke `~/apps/zsh-5.8-linux-x86_64-static/bin/zsh` and it'll just work,
although you'll probably want to add `~/apps/zsh-5.8-linux-x86_64-static/bin` to `PATH` for
convenience.

If you move or rename `~/apps/zsh-5.8-linux-x86_64-static`, remember to call `.../relocate`
afterwards. This script configures Zsh so that it loads builtin functions from the new location.

You can download built archives from [releases](https://github.com/romkatv/zsh-bin/releases).
