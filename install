#!/bin/sh
#
# Type `install -h` for usage and see https://github.com/romkatv/zsh-bin
# for documentation.

{

set -ue

if [ -n "${ZSH_VERSION:-}" ]; then
  emulate sh -o err_exit -o no_unset
fi

readonly url_base=https://github.com/romkatv/zsh-bin/releases/download/v6.1.1

readonly archives='
file:zsh-5.8-cygwin_nt-10.0-i686.tar.gz;   md5:a34f196174a41d6e0c72e4afbec51f1e; sha256:036d13c17ac8228c771bcfae015bc3a79e2adddeb6fefd84f6a4222e4961be5c;
file:zsh-5.8-cygwin_nt-10.0-x86_64.tar.gz; md5:cf5470d1f632cbc376a2e01e57d88c1f; sha256:9cd94b24a170c8b912c6540c5a343cc7c8c1903d144945310674a60add3bb169;
file:zsh-5.8-darwin-arm64.tar.gz;          md5:9cd3a105447d15e7643ba1d788a26f85; sha256:db777549692029b7c1c0474cdc2affc2dc87a74a2971dfa7be1be1ffcab695e3;
file:zsh-5.8-darwin-x86_64.tar.gz;         md5:40f860c05a2051399e6ca2fafb8695be; sha256:fba37a2fc286b0d8acfe73659a7ddce802689b8d5e76095d267651a182f7181f;
file:zsh-5.8-freebsd-amd64.tar.gz;         md5:4b262be949ca4fcb262748275cd104d6; sha256:2034bd1c085b39e7eae651ac6de7d11ccb2b2fdd471b46a84340b3235916b690;
file:zsh-5.8-linux-aarch64.tar.gz;         md5:f9acb1382ce0bd00bace00614fd36051; sha256:5caca77bcdaa218ec12e79f2e65c53c39058e0ed4f4f299991d18a800ca7c06d;
file:zsh-5.8-linux-armv6l.tar.gz;          md5:0e4934249018a1b409f0f34d155e14ef; sha256:937d15b49d978106320740c6563a6bb1a7e64e8ca569dc83fbea365d973f2de8;
file:zsh-5.8-linux-armv7l.tar.gz;          md5:2a698ed7aa69e47754e645ae58d4cd7b; sha256:9bab9143011f0b13d65bd437687771cc94f8e269ade99b6ecd3191da01708bc5;
file:zsh-5.8-linux-i386.tar.gz;            md5:c51c6a631c584a63044def4338f0e244; sha256:b6b189d7ee02684e6c5b20410dbea54af1049b066f8b1ffbd24ac7ef6f4bcd80;
file:zsh-5.8-linux-i586.tar.gz;            md5:49cccaa24e4654e1deb2a80e54b04469; sha256:e68c32ff9d9be83016706f6e359fcd4708d70e4de453f11130bc2c13f3f48b8d;
file:zsh-5.8-linux-i686.tar.gz;            md5:aaba0ee3959f4bceca5901ce85f858bb; sha256:7d6305ac92454c3bb9246df154a97df558c3d2236dcb23dc6d76e3ffb5b25ffd;
file:zsh-5.8-linux-x86_64.tar.gz;          md5:e04d16b71e4a9dbcab6d39c78b64124b; sha256:6df668fb6e9a12874e0d80518d582f2e99e512d4a4532fa73d938360aaddc838;
file:zsh-5.8-msys_nt-10.0-i686.tar.gz;     md5:98cfe37d011be05beb28be99785cf9ab; sha256:a5d8042abe351fdc84b07cac865951f20e3f8bcd77c5c27b199914e8c65a76f4;
file:zsh-5.8-msys_nt-10.0-x86_64.tar.gz;   md5:500cb53ae4bcbf6bbd7cfd1d0e822de4; sha256:258209e6f3a22daf21493e21a4b1f59769e5f2e243bf95033e90444dab784111;'

readonly lf="
"

if [ -t 1 ]; then
  readonly _0="$(printf '\033[0m')"
  readonly _B="$(printf '\033[1m')"
  readonly _U="$(printf '\033[4m')"
  readonly _R="$(printf '\033[31m')"
  readonly _G="$(printf '\033[32m')"
  readonly _Y="$(printf '\033[33m')"
else
  readonly _0=
  readonly _B=
  readonly _U=
  readonly _R=
  readonly _G=
  readonly _Y=
fi

if [ -t 0 -a -t 1 ]; then
  read_choice() {
    choice=''
    stty -icanon min 1 time 0
    while :; do
      c="$(dd bs=1 count=1 2>/dev/null && echo x)"
      choice="$choice${c%x}"
      n="$(printf '%s' "$choice" | wc -m)"
      [ "$n" -eq 0 ] || break
    done
    stty "$saved_tty_settings"
    [ "$choice" = "$lf" ] || echo
  }
  cleanup1() {
    trap - INT TERM EXIT
    stty "$saved_tty_settings"
  }
  saved_tty_settings="$(stty -g)"
  trap cleanup1 INT TERM EXIT
else
  read_choice() { IFS= read -r choice; }
  cleanup1() { :; }
fi

usage="$(cat <<END
Usage: install [OPTIONS] [-a <sha256|md5>]...
               [OPTIONS] -f FILE
               [OPTIONS] -u URL

If '-f FILE' is specified, install Zsh from the specified *.tar.gz
file produced by the build script.

If '-u URL' is specified, download the file and install as if with
'-f FILE'.

If neither '-f' nor '-u' is specified, download the appopriate file
from https://github.com/romkatv/zsh-bin/releases and install as if
with '-f FILE'. If '-a <sha256|md5>' is specified at least once,
abort installation if integrity of the downloaded package cannot
be verified with at least one of the listed hashing algorithms.

Options:

  -q

    Produce no output on success.

  -d DIR

    Install to this directory. If specified more than once, present
    an interactive dialog to choose the directory. Empty argument
    means a custom directory (requires manual user input). If '-d'
    is not specified, the effect is idential to this:

      -d /usr/local -d ~/.local -d ""

    Except on Termux:

      -d "\$PREFIX"/local -d ~/.local -d ""

  -e VALUE

    Whether to add the newly installed Zsh to /etc/shells. This will
    allow users to designate this instance of Zsh as their login shell.
    VALUE can be 'yes', 'no' or 'ask' (the default). If /etc/shells
    does not exist, all values are equivalent to 'no'.

  -s FD

    On success, write the path to the installation directory to this
    file descriptor.
END
)"

absfile() {
  if [ ! -e "$1" ]; then
    >&2 echo "${_R}error${_0}: file not found: ${_U}$1${_0}"
    return 1
  fi
  local dir base
  dir="$(dirname -- "$1")"
  base="$(basename -- "$1")"
  ( cd -- "$dir" && dir="$(pwd)" && printf '%s/%s\n' "${dir%/}" "${base}" )
}

check_dir() {
  if [ -z "$1" ]; then
    >&2 echo "${_R}error${_0}: directory cannot be empty string"
    exit 1
  fi
  if [ -z "${1##~*}" ]; then
    >&2 echo "${_R}error${_0}: please expand ${_U}~${_0} in directory name: ${_U}$1${_0}"
    exit 1
  fi
  if [ -z "${1##//*}" ]; then
    >&2 echo "${_R}error${_0}: directory cannot start with ${_U}//${_0}: ${_U}$1${_0}"
    exit 1
  fi
}

add_dir() {
  num_dirs=$((num_dirs + 1))
  dirs="$dirs${num_dirs}${1}${lf}"
  if [ -n "$1" ]; then
    dirs_c="$dirs_c  ${_B}($num_dirs)${_0}  ${_U}${1}${_0}${lf}"
  else
    dirs_c="$dirs_c  ${_B}($num_dirs)${_0}  Custom directory.${lf}"
  fi
}

check_sudo() {
  local dir="$1"
  sudo=
  while true; do
    if [ -e "$dir" ]; then
      if [ ! -d "$dir" ]; then
        >&2 echo "${_R}error${_0}: not a directory: ${_U}$dir${_0}"
        return 1
      fi
      if [ ! -w "$dir" ]; then
        if [ "$euid" = 0 ]; then
          >&2 echo "${_R}error${_0}: directory not writable: ${_U}$dir${_0}"
          return 1
        else
          if [ -z "$quiet" ]; then
            echo "${_Y}===>${_0} using ${_U}${_G}sudo${_0} for installation"
          fi
          sudo=sudo
        fi
      fi
      break
    fi
    if [ "$dir" = / ] || [ "$dir" = . ]; then
      break
    fi
    dir="$(dirname -- "$dir")"
  done
}

dirs=
dirs_c=
num_dirs=0
quiet=
algos=
url=
file=
sudo=
fd=
etc_shells=
asked=

command -v sudo >/dev/null 2>&1 && euid="$(id -u 2>/dev/null)" || euid=0

while getopts ':hqd:e:s:a:f:u:' opt "$@"; do
  case "$opt" in
    h)
      printf '%s\n' "$usage"
      exit
    ;;
    q)
      if [ -n "$quiet" ]; then
        >&2 echo "${_R}error${_0} duplicate option: ${_B}-${opt}${_0}"
        exit 1
      fi
      quiet=1
    ;;
    d)
      if printf "%s" "$dirs" | cut -b 2- | grep -qxF -- "$OPTARG"; then
        >&2 echo "${_R}error${_0}: duplicate option: ${_B}-${opt} ${OPTARG}${_0}"
        exit 1
      fi
      if [ "$(printf "%s" "$OPTARG" | wc -l | tr -Cd '0-9')" != 0 ]; then
        >&2 echo "${_R}error${_0}: incorrect value of ${_B}-${opt}${_0}: ${_B}${OPTARG}${_0}"
        exit 1
      fi
      if [ "$num_dirs" = 9 ]; then
        >&2 echo "${_R}error${_0}: too many options: ${_B}-${opt}${_0}"
        exit 1
      fi
      if [ -n "$OPTARG" ]; then
        check_dir "$OPTARG"
      fi
      add_dir "$OPTARG"
    ;;
    e)
      if [ -n "$etc_shells" ]; then
        >&2 echo "${_R}error${_0} duplicate option: ${_B}-${opt}${_0}"
        exit 1
      fi
      case "$OPTARG" in
        yes|no|ask);;
        *)
          >&2 echo "${_R}error${_0}: incorrect value of ${_B}-${opt}${_0}: ${_B}${OPTARG}${_0}"
          exit 1
        ;;
      esac
      etc_shells="$OPTARG"
    ;;
    s)
      if [ -n "$fd" ]; then
        >&2 echo "${_R}error${_0} duplicate option: ${_B}-${opt}${_0}"
        exit 1
      fi
      if ! printf '%s' "$OPTARG" | tr '\n' x | grep -qxE '[1-9][0-9]*'; then
        >&2 echo "${_R}error${_0}: incorrect value of ${_B}-${opt}${_0}: ${_B}${OPTARG}${_0}"
        exit 1
      fi
      fd="$OPTARG"
    ;;
    a)
      case "$OPTARG" in
        sha256|md5)
          if [ -n "$algos" -a -z "${algos##*<$OPTARG>*}" ]; then
            >&2 echo "${_R}error${_0}: duplicate option: ${_B}-${opt} ${OPTARG}${_0}"
            exit 1
          fi
          algos="$algos<$OPTARG>"
        ;;
        *)
          >&2 echo "${_R}error${_0}: incorrect value of ${_B}-${opt}${_0}: ${_B}${OPTARG}${_0}"
          exit 1
        ;;
      esac
    ;;
    f)
      if [ -n "$file" ]; then
        >&2 echo "${_R}error${_0}: duplicate option: ${_B}-${opt}${_0}"
        exit 1
      fi
      if [ -z "$OPTARG" ]; then
        >&2 echo "${_R}error${_0}: incorrect value of ${_B}-${opt}${_0}: ${_B}${OPTARG}${_0}"
        exit 1
      fi
      file="$(absfile "$OPTARG")"
    ;;
    u)
      if [ -n "$url" ]; then
        >&2 echo "${_R}error${_0}: duplicate option: ${_B}-${opt}${_0}"
        exit 1
      fi
      if [ -z "$OPTARG" ]; then
        >&2 echo "${_R}error${_0}: incorrect value of ${_B}-${opt}${_0}: ${_B}${OPTARG}${_0}"
        exit 1
      fi
      url="$OPTARG"
    ;;
    \?) >&2 echo "${_R}error${_0}: invalid option: ${_B}-${OPTARG}${_0}"           ; exit 1;;
    :)  >&2 echo "${_R}error${_0}: missing required argument: ${_B}-${OPTARG}${_0}"; exit 1;;
    *)  >&2 echo "${_R}internal error${_0}: unhandled option: ${_B}-${opt}${_0}"   ; exit 1;;
  esac
done

if [ "$OPTIND" -le $# ]; then
  >&2 echo "${_R}error${_0}: unexpected positional argument"
  return 1
fi

if [ -n "$algos" ]; then
  if [ -n "$file" ]; then
    >&2 echo "${_R}error${_0}: incompatible options: ${_B}-f${_0} and ${_B}-a${_0}"
    exit 1
  fi
  if [ -n "$url" ]; then
    >&2 echo "${_R}error${_0}: incompatible options: ${_B}-u${_0} and ${_B}-a${_0}"
    exit 1
  fi
fi

if [ "$num_dirs" = 0 ]; then
  if [ "$(uname -s 2>/dev/null)" = Linux ] &&
     [ "$(uname -o 2>/dev/null)" = Android ] &&
     [ -d "${PREFIX:-/data/data/com.termux/files/usr}" ]; then
    usr_local="${PREFIX:-/data/data/com.termux/files/usr}"/local
    usr_local_d="${_U}\$PREFIX/local${_0}"
  else
    usr_local=/usr/local
    usr_local_d="${_U}/usr/local${_0}   "
  fi

  add_dir "$usr_local"
  add_dir ~/.local
  add_dir ""

  dirs_c="  ${_B}(1)${_0}  $usr_local_d     ${_Y}<=${_0}"
  if check_sudo "$usr_local" >/dev/null 2>/dev/null; then
    if [ -n "$sudo" ]; then
      dirs_c="${dirs_c} uses ${_U}${_G}sudo${_0}"
    else
      dirs_c="${dirs_c} does not need ${_U}${_G}sudo${_0}"
    fi
    if [ -d "$usr_local" ]; then
      dirs_c="${dirs_c} (${_B}recommended${_0})"
    fi
  else
    dirs_c="${dirs_c} ${_R}not writable${_0}"
  fi

  dirs_c="${dirs_c}${lf}  ${_B}(2)${_0}  ${_U}~/.local${_0}          ${_Y}<=${_0}"
  if check_sudo ~/.local >/dev/null 2>/dev/null; then
    if [ -n "$sudo" ]; then
      dirs_c="${dirs_c} uses ${_U}${_G}sudo${_0}"
    else
      dirs_c="${dirs_c} does not need ${_U}${_G}sudo${_0}"
    fi
  else
    dirs_c="${dirs_c} ${_R}not writable${_0}"
  fi

  dirs_c="${dirs_c}${lf}  ${_B}(3)${_0}  Custom directory.${lf}"
fi

if [ "$num_dirs" = 1 ]; then
  choice=1
else
  echo "Choose installation directory for ${_G}Zsh 5.8${_0}:"
  echo ""
  printf "%s" "$dirs_c"
  printf '  \033[1m(q)\033[0m  Quit and do nothing.\n'
  echo ""
  while true; do
    printf "%sChoice [%sq]:%s " "$_B" "$(printf "%s" "$dirs" | cut -b 1 | tr -d '\n')" "$_0"
    read_choice
    if printf "%s" "$dirs" | cut -b 1 | grep -qxF -- "$choice"; then
      break
    fi
    if [ "$choice" = q ] || [ "$choice" = Q ]; then
      exit 1
    fi
    if [ -n "$choice" -a "$choice" != "$lf" ]; then
      >&2 echo "Invalid choice: ${_R}$choice${_0}"
    fi
  done
  asked=1
fi

dir="$(printf "%s" "$dirs" | sed "${choice}!d" | cut -b 2-)"
if [ -z "$dir" ]; then
  while :; do
    echo -n "${_B}Custom directory:${_0} "
    read -r dir
    [ -z "$dir" ] || break
  done
  check_dir "$dir"
  if [ -z "$quiet" ]; then
    echo
  fi
  asked=1
elif [ -z "$quiet" ] && [ "$num_dirs" != 1 ]; then
  echo
fi

if [ -z "$quiet" ]; then
  printf "%s\n" "${_Y}===>${_0} installing ${_G}Zsh 5.8${_0} to ${_U}$dir${_0}"
fi

check_sudo "$dir"

$sudo mkdir -p -- "$dir"
cd -- "$dir"
dir="$(pwd)"

if [ -z "$file" -a -z "$url" ]; then
  kernel="$(uname -s | tr '[A-Z]' '[a-z]')"
  arch="$(uname -m | tr '[A-Z]' '[a-z]')"

  case "$kernel" in
    linux-armv8l)    kernel=linux-aarch64;;
    msys_nt-6.*)     kernel=msys_nt-10.0;;
    msys_nt-10.*)    kernel=msys_nt-10.0;;
    mingw32_nt-6.*)  kernel=msys_nt-10.0;;
    mingw32_nt-10.*) kernel=msys_nt-10.0;;
    mingw64_nt-6.*)  kernel=msys_nt-10.0;;
    mingw64_nt-10.*) kernel=msys_nt-10.0;;
    cygwin_nt-6.*)   kernel=cygwin_nt-10.0;;
    cygwin_nt-10.*)  kernel=cygwin_nt-10.0;;
  esac

  filename="zsh-5.8-${kernel}-${arch}.tar.gz"
  url="$url_base/$filename"

  if [ -n "${archives##*file:$filename;*}" ]; then
    >&2 echo "${_R}error${_0}: there is no prebuilt binary for your architecture"
    >&2 echo
    >&2 echo "See ${_U}https://github.com/romkatv/zsh-bin#compiling${_0} for building one."
    exit 1
  fi

  check_sig=1
else
  check_sig=0
fi

if [ -n "$url" ]; then
  file="${TMPDIR:-/tmp}"/zsh-bin.tmp.$$.tar.gz
  cleanup2() { cleanup1; rm -f -- "$file"; }
  trap cleanup2 INT TERM EXIT

  if [ -z "$quiet" ]; then
    echo "${_Y}===>${_0} fetching ${_U}${url##*/}${_0}"
  fi

  (
    cd -- "${file%/*}"
    file="${file##*/}"

    set +e

    if command -v curl >/dev/null 2>&1; then
      err="$(command curl -fsSLo "$file" -- "$url" 2>&1)"
    elif command -v wget >/dev/null 2>&1; then
      err="$(command wget -O "$file" -- "$url" 2>&1)"
    elif command -v fetch >/dev/null 2>&1; then
      err="$(command fetch -q -o "$file" -- "$url" 2>&1)"
    else
      >&2 echo "${_R}error${_0}: please install ${_G}curl${_0} or ${_G}wget${_0} and retry"
      exit 1
    fi

    if [ $? != 0 ]; then
      >&2 printf "%s\n" "$err"
      >&2 echo "${_R}error${_0}: failed to download ${_U}$url${_0}"
      exit 1
    fi
  )
fi

if [ "$check_sig" = 1 ]; then
  if [ -z "$quiet" ]; then
    echo "${_Y}===>${_0} verifying archive integrity"
  fi

  for algo in sha256 md5; do
    hash=none
    case "$algo" in
      sha256)
        {
          command -v shasum >/dev/null 2>/dev/null                       &&
            hash="$(shasum -b -a 256 -- "$file" </dev/null 2>/dev/null)" &&
            hash="${hash%% *}"                                           &&
            [ ${#hash} -eq 64 ]
        } || {
          command -v sha256sum >/dev/null 2>/dev/null                    &&
            hash="$(sha256sum -b -- "$file" </dev/null 2>/dev/null)"     &&
            hash="${hash%% *}"                                           &&
            [ ${#hash} -eq 64 ]
        } || {
          # Note: sha256 can be from hashalot. It's incompatible.
          # Thankfully, it produces shorter output.
          command -v sha256 >/dev/null 2>/dev/null                      &&
            hash="$(sha256 -- "$file" </dev/null 2>/dev/null)"          &&
            hash="${hash##* }"                                          &&
            [ ${#hash} -eq 64 ]
        } || {
          hash=none
        }
      ;;
      md5)
        {
          command -v md5sum >/dev/null 2>/dev/null                      &&
            hash="$(md5sum -b -- "$file" </dev/null 2>/dev/null)"       &&
            hash="${hash%% *}"                                          &&
            [ ${#hash} -eq 32 ]
        } || {
          command -v md5 >/dev/null 2>/dev/null                         &&
          hash="$(md5 -- "$file" </dev/null 2>/dev/null)"               &&
          hash="${hash##* }"                                            &&
          [ ${#hash} -eq 32 ]
        } || {
          hash=none
        }
      ;;
      *)
        >&2 echo "${_R}internal error${_0}: unhandled algorithm: ${_B}$algo${_0}"
        exit 1
      ;;
    esac
    if [ "$hash" != none ]; then
      if [ -n "${archives##* $algo:$hash;*}" ]; then
        >&2 echo "${_R}error${_0}: ${_B}$algo${_0} signature mismatch"
        >&2 echo ""
        >&2 echo "Expected:"
        >&2 echo ""
        >&2 echo "  ${_G}$(printf "%s" "$archives" | grep -F -- "${url##*/}" | sed 's/  */ /g')${_0}"
        >&2 echo ""
        >&2 echo "Found:"
        >&2 echo ""
        >&2 echo "  ${_R}$algo:$hash${_0}"
        exit 1
      fi
      if [ -z "$quiet" ]; then
        echo "${_Y}===>${_0} ${_B}$algo${_0} signature matches"
      fi
      algos="${algos##*<$algo>*}"
    else
      if [ -z "$quiet" ]; then
        echo "${_Y}===>${_0} no tools to verify ${_B}$algo${_0} signature"
      fi
    fi
  done
fi

if [ -n "$algos" ]; then
  >&2 echo "${_R}error${_0}: no tools available to verify archive integrity"
  exit 1
fi

if [ -z "$quiet" ]; then
  echo "${_Y}===>${_0} extracting files"
fi

$sudo tar -xzf "$file"
$sudo ./share/zsh/5.8/scripts/relocate

$sudo ./bin/zsh -fc '
  setopt extended_glob                                 || exit
  zmodload -F zsh/files b:{zf_mv,zf_rm,zf_ln,zf_mkdir} || exit

  for pat in "[^[:upper:]]" "[[:upper:]]"; do
    for hex in ./share/terminfo/[[:xdigit:]][[:xdigit:]].zsh-bin(:t:r); do
      printf -v char "\\x$hex"
      [[ $char == $~pat ]] || continue
      [[ -e ./share/terminfo/$hex ]] || zf_mkdir -- ./share/terminfo/$hex || exit
      cp -- ./share/terminfo/$hex.zsh-bin/* ./share/terminfo/$hex/ || exit
      if [[ $char != [[:upper:]] ||
            ! ./share/terminfo/$char -ef ./share/terminfo/${(L)char} ]]; then
        [[ -e ./share/terminfo/$char ]] ||
          zf_ln -s -- $hex ./share/terminfo/$char 2>/dev/null ||
          zf_mkdir -- ./share/terminfo/$char || exit
        cp -- ./share/terminfo/$hex.zsh-bin/* ./share/terminfo/$char/ || exit
      fi
      zf_rm -rf -- ./share/terminfo/$hex.zsh-bin || exit
    done
  done

  for file in ./share/zsh/5.8/functions/*~*.[^/]#(.); do
    tmp=$file.tmp.$$.zwc
    zf_rm -f -- $tmp $file.zwc || exit
    zcompile -R -- $tmp $file  || exit
    zf_mv -f -- $tmp $file.zwc || exit
  done'

if [ ! -f /etc/shells ] || grep -qxF "$dir/bin/zsh" /etc/shells 2>/dev/null; then
  etc_shells=no
fi

if [ "${etc_shells:-ask}" = ask ]; then
  if [ -z "$quiet" -o -n "$asked" ]; then
    echo
  fi
  printf "%s\n" "Add ${_G}$dir/bin/zsh${_0} to ${_U}/etc/shells${_0}?"
  echo
  echo "This will allow you to use it as a login shell."
  echo ""
  echo "  ${_B}(y)${_0}  Yes. ${_B}Recommended.${_0}"
  echo "  ${_B}(n)${_0}  No."
  echo "  ${_B}(q)${_0}  Quit and do nothing."
  echo ""
  while true; do
    echo -n "${_B}Choice [ynq]:${_0} "
    read_choice
    case "$choice" in
      y|Y)
        if [ -z "$quiet" ]; then
          echo
        fi
        etc_shells=yes
        break
      ;;
      n|N)
        etc_shells=no
        break
      ;;
      q|Q)
        exit 1
      ;;
    esac
    if [ -n "$choice" -a "$choice" != "$lf" ]; then
      >&2 echo "Invalid choice: ${_R}$choice${_0}"
    fi
  done
fi

if [ "$etc_shells" = yes ]; then
  if [ -z "$quiet" ]; then
    printf "%s\n" "${_Y}===>${_0} adding ${_G}$dir/bin/zsh${_0} to ${_U}/etc/shells${_0}"
  fi
  if [ -z "$sudo" ]; then
    if [ ! -w /etc/shells ]; then
      if [ "$euid" = 0 ]; then
        >&2 echo "${_R}error${_0}: file not writable: ${_U}/etc/passwd${_0}"
        exit 1
      else
        if [ -z "$quiet" ]; then
          echo "${_Y}===>${_0} using ${_U}${_G}sudo${_0} for modifications"
        fi
        sudo=sudo
      fi
    fi
  fi
  printf "%s\n" "$dir/bin/zsh" | $sudo tee -a /etc/shells >/dev/null
fi

if [ -z "$quiet" ]; then
  echo ""
  echo "Installed ${_G}Zsh 5.8${_0} to ${_U}$dir${_0}"
  echo ""
  echo "To start Zsh, type:"
  echo ""
  if ! zsh="$(command -v zsh 2>/dev/null)" || [ "$zsh" != "$dir/bin/zsh" ]; then
    echo "  ${_Y}export${_0} PATH=${_Y}\"$dir/bin:\$PATH\"${_0}"
  fi
  echo "  ${_G}zsh${_0}"
  echo
fi

if [ -n "$fd" ]; then
  printf "%s\n" "$dir" >&"$fd"
fi

}
