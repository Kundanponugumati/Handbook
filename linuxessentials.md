# Linux Essentials

Basic commands for daily use. These work in the Mac terminal and inside Linux Docker containers.

---

## 1. Move around folders

| Command | What it does |
|---|---|
| `pwd` | Shows the folder you are in now (print working directory). |
| `ls` | Lists files and folders in the current folder. |
| `ls -l` | Lists with details (size, date, permissions). |
| `ls -a` | Lists all files, also hidden files (names that start with `.`). |
| `ls -la` | Both of the above. |
| `cd folder` | Goes into `folder`. |
| `cd ..` | Goes one folder up (back). |
| `cd ../..` | Goes two folders up. |
| `cd ~` or `cd` | Goes to your **home** folder (for example `/Users/kundan`). |
| `cd /` | Goes to the **root** folder (the top of the whole system). |
| `cd -` | Goes back to the previous folder. |

> Note: `~` is the **home** folder, not the root folder. The root folder is `/`.

---

## 2. Create files and folders

| Command | What it does |
|---|---|
| `mkdir name` | Creates a folder. |
| `mkdir -p a/b/c` | Creates nested folders. Also creates missing parent folders. |
| `touch file1 file2` | Creates empty files. If a file exists, it updates its time. |

---

## 3. Delete files and folders

| Command | What it does |
|---|---|
| `rm file` | Deletes a file. |
| `rm file1 file2` | Deletes many files. |
| `rmdir folder` | Deletes a folder. Works only if the folder is empty. |
| `rm -r folder` | Deletes a folder and everything inside it. |
| `rm -rf folder` | Same, but does not ask for confirmation. |
| `rm -i file` | Asks "are you sure?" before it deletes. |

> **Warning:** `rm` does not use the Trash. Deleted files are gone. Be careful with `rm -rf`. Always check the path first.

---

## 4. Copy, move, rename

| Command | What it does |
|---|---|
| `cp a.txt b.txt` | Copies `a.txt` to `b.txt`. |
| `cp -r folder1 folder2` | Copies a folder and everything inside it. |
| `mv a.txt folder/` | Moves `a.txt` into `folder`. |
| `mv old.txt new.txt` | Renames a file. (Linux uses `mv` to rename.) |

---

## 5. Write inside a file

**Option A: from the command line**

| Command | What it does |
|---|---|
| `echo "hello" > file.txt` | Writes `hello` into the file. **Replaces** the old content. |
| `echo "world" >> file.txt` | Adds `world` at the end. **Keeps** the old content. |

- `>` = overwrite.
- `>>` = append (add to the end).

**Option B: many lines at once (heredoc)**

```bash
cat > file.txt << EOF
line 1
line 2
line 3
EOF
```

**Option C: open a text editor in the terminal**

| Command | What it does |
|---|---|
| `nano file.txt` | Opens a simple editor. Save: `Ctrl+O`, then `Enter`. Exit: `Ctrl+X`. |
| `open -e  file.txt` | Opens a text editor.|
| `vim file.txt` | Opens vim. Press `i` to type. Press `Esc`, then type `:wq` and `Enter` to save and exit. Type `:q!` to exit without saving. |
| `code file.txt` | Opens the file in VS Code (on your Mac, not in a container). |

> Tip: start with `nano`. It is the easiest.

---

## 6. Read files

| Command | What it does |
|---|---|
| `cat file.txt` | Prints the whole file. |
| `less file.txt` | Opens the file page by page. Arrow keys to scroll. `q` to exit. |
| `head file.txt` | Shows the first 10 lines. |
| `head -n 20 file.txt` | Shows the first 20 lines. |
| `tail file.txt` | Shows the last 10 lines. |
| `tail -f app.log` | Shows new lines live as the file grows. Good for logs. `Ctrl+C` to stop. |
| `wc -l file.txt` | Counts the lines in the file. |

---

## 7. Search

| Command | What it does |
|---|---|
| `grep "word" file.txt` | Shows the lines that contain `word`. |
| `grep -i "word" file.txt` | Same, but ignores upper/lower case. |
| `grep -r "word" folder/` | Searches in all files inside a folder. |
| `grep -n "word" file.txt` | Also shows line numbers. |
| `find . -name "*.py"` | Finds all files that end with `.py` in this folder and subfolders. |

---

## 8. Pipes: connect commands

The `|` symbol sends the output of one command into the next command.

```bash
cat app.log | grep "ERROR"            # only error lines
docker logs myapp | grep "user=A"     # only logs of user A
ls -l | wc -l                         # count files
```

---

## 9. Permissions

| Command | What it does |
|---|---|
| `chmod +x script.sh` | Makes a script runnable. |
| `./script.sh` | Runs a script in the current folder. |
| `sudo command` | Runs a command as admin (root user). Asks for your password. |

---

## 10. Processes and system

| Command | What it does |
|---|---|
| `ps aux` | Lists all running programs (processes). |
| `top` | Shows live CPU and memory use. `q` to exit. |
| `kill 1234` | Stops the process with ID `1234`. |
| `kill -9 1234` | Forces the process to stop. |
| `df -h` | Shows free disk space. |
| `du -sh folder` | Shows the size of a folder. |

---

## 11. Network

| Command | What it does |
|---|---|
| `curl http://localhost:8000` | Sends a request to a URL and prints the reply. Good to test APIs. |
| `ping google.com` | Checks if a host is reachable. `Ctrl+C` to stop. |
| `lsof -i :8000` | Shows which program uses port 8000 (Mac). |

---

## 12. Environment variables

| Command | What it does |
|---|---|
| `echo $HOME` | Prints the value of the variable `HOME`. |
| `export NAME=value` | Sets a variable for this terminal session. |
| `env` | Lists all variables. |

---

## 13. Useful shortcuts

| Shortcut | What it does |
|---|---|
| `Tab` | Completes file and folder names. Press it often. |
| `↑` / `↓` | Goes through old commands. |
| `Ctrl+C` | Stops the running command. |
| `Ctrl+L` or `clear` | Clears the screen. |
| `Ctrl+R` | Searches old commands. Type part of the command. |
| `history` | Lists old commands. |
| `man ls` | Shows the manual for `ls`. `q` to exit. |
| `ls --help` | Shows short help for a command (Linux). |

---

## 14. Use these inside a Docker container

```bash
docker exec -it <container-name> bash    # open a terminal inside the container
docker exec -it <container-name> sh      # use this if bash is not installed
exit                                     # leave the container terminal
```

Inside the container, all the commands above work the same way.

> Note: small images (for example `alpine` or `slim`) may not have `nano`, `vim`, or `curl`. Install them first if needed:
> - Debian/Ubuntu images: `apt update && apt install -y nano curl`
> - Alpine images: `apk add nano curl`
