# Shell Upgrades

### Most reliable

```bash
/usr/bin/script -qc /bin/bash /dev/null
```

### Python

```python
python -c 'import pty; pty.spawn("/bin/bash")'
/usr/bin/python -c 'import pty; pty.spawn("/bin/bash")'

# Python 3
python3 -c 'import pty; pty.spawn("/bin/bash")'
/usr/bin/python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### Spawn TTY Shell

```bash
# Spawn Python shell
python -c 'import pty; pty.spawn("/bin/bash")'

# Background the shell
Ctrl + Z

# Get current Rows and Columns
stty -a | head -n1 | cut -d ';' -f 2-3 | cut -b2- | sed 's/; /\n/'

# Bring shell back to the foreground
stty raw -echo; fg

# Set size for the remote shell 
# (where ROWS and COLS are the values from the 3rd command)
stty rows ROWS cols COLS

# Add some colours
export TERM=xterm-256color

# Reload bash to apply the TERM variable
exec /bin/bash
```

### Spawn TTY Shell #2

Similar to above, slightly less function but easier to implement.

```python
# Spawn Python shell
python -c 'import pty;pty.spawn("/bin/bash")'

# Give terminal functions
export TERM=xterm

# Background the shell
Ctrl + Z

# Bring shell back to the foreground 
stty raw -echo; fg
```

