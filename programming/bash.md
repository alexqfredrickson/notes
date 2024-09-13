# Bash

## Basics

Bash stands for `Bourne Again SHell`. Pressing tab autocompletes text.

## IO Redirection

Programs produce either output (via `stdout`, or "standard output") or errors (via `stderr`, or "standard error").

| Operator       | Description                                             |
| -----------    | ---------------------------------------------------     |
| `>`            | Redirects `stdout` (1) to a file; overwrites the file   |
| `>>`           | Redirects `stdout` (1) to a file; appends to the file   |
| `2>`           | Redirects `stderr` (2) to a file; overwrites the file   |
| `>&`           | Combines `stdout` (1) and `stderr` (2) redirection      |
| `2>&1`         | Redirects `stderr` (2) to `stdout` (1)                  |
| `\|`           | Redirects `stdout` (1) to `stdin` (0)                   |

Redirecting to `/dev/null` will suppress output.

## Strings

Double quotes (`"`) represent strings. Backquotes (\`), backslashes (`\`), and `$` signs are preserved.

Single quotes (`'`) suppress all parameter, arithmetic, and command/substitution expressions.

## Commands

### `history`

Gets Bash history.

#### History Expansion

| Operator    | Description                                             |
| ----------- | ---------------------------------------------------     |
| `!!`        | Repeats the last command.  |
| `!8`        | Invokes the `8`th command in history.  |
| `!string`   | Invokes a history command *starting with* the specified string.  |
| `!?string`  | Invokes a history command *containing* the specified string.  |

### `id`

Gets current user identity/group.

User accounts are found in `/etc/passwd`. Groups are in `/etc/group`. Passwords are in `/etc/shadow`.

### `chmod`

Changes file permissions.

| Mode    | Binary    | Linux Notation |
| -----   | -------   |--------------- |
| `0`     |  `000`    | `---`          |
| `1`     |  `001`    | `--x`          |
| `2`     |  `010`    | `-w-`          |
| `3`     |  `011`    | `-wx`          |
| `4`     |  `100`    | `r--`          |
| `5`     |  `101`    | `r-x`          |
| `6`     |  `110`    | `rw-`          |
| `7`     |  `111`    | `rwx`          |

`r` is "read". `w` is "write". `x` is "execute".

File permissions look something like `drwxrw-r--`.

 * The `d` is the *file type*. 
   * `-` is a regular file. 
   * `d` is a directory. 
   * `l` is a symbolic link. 
   * `c` is a character file. 
   * `b` is a block file.
 * The next three letters (`rwx`) are *user privileges*.
 * The middle three letters (`rw-`) are *group privileges*.
 * The final three letters (`r--`) are *world privileges*.

### `umask`

Sets default file attributes and permissions.

`/etc/sudoers` defines users that are allowed to run `sudo` commands.

### `su`

Runs commands as a different user ... i.e. whoever, including `root`.

### `sudo`

Runs a command as root.

### `ps`

Displays current runnning processes.

### `top`

Displays tasks.

### `jobs`

Lists active jobs.

### `shutdown`

Shuts the system down.

### `bg`

Places a job in the *background*, prohibiting it from accepting user input. Running a program with `&` makes it run in the background automatically.

### `fg`

Places a job in the *foreground*, allowing it to accept user input.

### `kill`

Kills a process.

### `printenv`

Prints the *environment* (i.e. environmental variables and shell variables).

### `set`

Sets shell options.

### `export` 

Exports an environment to subsequently executed programs.

### `alias`

Aliases a command.

### `ls`

Lists files.

### `touch`

Creates an empty file.

### `mv`

Moves a file.

### `cat`

Concatenates files; prints `stdout.`

### `traceroute`

Traces the route of a network packet through each system out to a host, returning hostnames, IP addresses, performance data, etc.

### `netstat`

Examines network settings and statistics.

## Processes

Processes are instances of programs being executed; i.e. steps being performed by a *processor*. Processes have execution contexts, images, memory, files, etc. A process can contain multiple threads.

## Jobs

Jobs are programs that start and do not detach.

## Login Shells

A login shell is the first executing process under your user ID in an interactive session. `su` and `ssh`-bases sessions use interactive login shells. It's rare to run a non-interactive login shell. When you start a shell in a terminal in an existing session, you get an interactive, non-login shell. When a shell runs a script or a command passed on its command line, it's a non-interactive, non-login shell.

On system login, Bash starts and reads startup-related configuration files which contain environmental variables shared by all users - generally located in the home directory.

In a **login** shell session: `/etc/profile` is read, then `~/.bash_profile`. User-defined variables go into the bash profile. `~/.bash_login` and `~/.profile` are read as backups. 

In a **non-login** shell session: `/etc/bash.bashrc` is read, then `~/.bashrc`. 

## Package Management

`.deb` files are for Debian Linux. `.rpm` files are for Red Hat Linux.

Package maintainers compile source code, create package metadata, create installation scripts, and modify source code to suit the particular distribution.

Low-level package management tools like `dpkg` and `rpm` install and remove packages from the system.

Higher-level package management tools like `yum`, `apt`, and `aptitude` will do the same, but perform dependency resolution and search metadata.

## The Filesystem Hierarchy Standard


| Path    | Description               |  
| -----   | -------                   |
| /       |  The root directory.                     |
| /bin    |  Contains command binaries (e.g. `cat`, `ls`, etc.)                     |
| /boot   |  Contains boot loaders (e.g. kernels)                     |
| /dev    |  Contains device files (e.g. `/dev/null`)                     |
| /etc    |  Contains system-wide configuration files                     |
| /home   |  A user's home directory                     |
| /lib    |  Libraries for binaries in `/bin` and `/sbin`                     |
| /media  |  Mount points for removable media such as CD-ROM                     |
| /mnt    |  Temporarily-mounted filesystems                     |
| /opt    |  Optional application software packages that are not managed by distribution managers.                     |
| /proc   | A virtual file system that provides process data as files. This seems like a bit of a hack.                     |
| /root   |  Home directory for the root user                     |
| /run    |  Runtime variable data (e.g. current logged-in user, running daemons, etc.)                     |
| /sbin   |  System binaries (e.g. `fsck`, `init`, etc.)                     |
| /srv    |  Contains site-specific, *served* data, like if you're running a file-share server.                     |
| /sys    |  Device, kernel, and driver feature information                     |
| /tmp    |  Temporary files                     |
| /usr    |  System-wide read-only files; multi-user applications (e.g. `tail`, `awk`, etc.). Contains non-essential command binaries, language-specific header files, etc.                     |
