When you gain initial access to a target system, the shell you get is often limited — lacking features like tab completion, proper signal handling (e.g., Ctrl+C), or full TTY (interactive terminal) support. **Upgrading a shell** means turning this basic shell into a **fully interactive, stable shell** so you can work more effectively on the system.

**Check Available Shells**

```shell
cat /etc/shells
# Lists all valid login shells available on the system
```

**Start an Interactive Bash Session**

```shell
/bin/bash -i
# Launches an interactive instance of Bash
```

**Spawn a TTY Shell Using Python**

```shell
python --version
python -c 'import pty; pty.spawn("/bin/bash")'
# Uses Python to spawn a pseudo-terminal with Bash.

# If Python 2 isn’t available, try:
python3 --version
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

**Alternative with Perl**

```shell
perl -e 'exec "/bin/bash";'
# Uses Perl to execute a new Bash shell
```

**Alternative with Ruby**

```shell
ruby -e 'exec "/bin/bash"'
# Runs Bash through Ruby if available
```

