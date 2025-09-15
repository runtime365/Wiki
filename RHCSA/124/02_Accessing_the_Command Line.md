# 2.1. Accessing the Command Line in a Text Interface

## Objectives
Log in to a Linux system and run simple commands by using the shell.

## Introduction to the Bash Shell

A **command line** is a text-based interface that is used to input instructions to a computer system. The Linux command line is provided by a program named the **shell**. Many shell program variants have been developed over the years. Every user can use a different shell, but Red Hat recommends using the default shell for system administration.

The default user shell in Red Hat Enterprise Linux (RHEL) is the **GNU Bourne-Again Shell (Bash)**. The Bash shell is an improved version of the original Bourne shell (sh) on UNIX systems.

### Shell Prompts

The shell displays a string when it is waiting for user input, named the **shell prompt**.

**Regular user prompt:**
```bash
user@host:~$
```

**Superuser (root) prompt:**
```bash
root@host:~#
```

> **Note:** The number sign (#) indicates the superuser shell. Identifying a superuser shell helps you to avoid mistakes that can affect the whole system.

### Shell Capabilities

Using Bash to execute commands can be powerful. The Bash shell provides:
- A scripting language that can support task automation
- Capabilities that can enable or simplify operations that are hard to accomplish at scale with graphical tools

> **Comparison Note:** The Bash shell is conceptually similar to the Microsoft Windows cmd.exe command-line interpreter. However, Bash has a sophisticated scripting language, and is more similar to Windows PowerShell.

## Shell Basics

Commands that are entered at the shell prompt have three basic parts:

1. **Command** to run
2. **Options** to adjust the behavior of the command
3. **Arguments**, which are typically the targets of the command

### Command Structure

```bash
command [options] [arguments]
```

**Example:**
```bash
usermod -L user01
```
- `usermod` is the command
- `-L` is the option
- `user01` is the argument

This command locks the password of the user01 user account.

## Logging in to a Local System

### Physical Console

A **terminal** is a text-based interface to enter commands into and print output from a computer system. To run the shell, you must log in to the computer on a terminal.

- Hardware keyboard and display for input and output might be directly connected to the computer
- This is the **physical console** from the Linux machine
- The physical console supports multiple **virtual consoles**, which can run on separate terminals
- Each virtual console supports an independent login session

### Virtual Console Navigation

Switch between virtual consoles by pressing:
```
Ctrl + Alt + F1 through F6
```

### Virtual Console Layout in RHEL 10

- **tty1**: Graphical login screen (if available)
- **tty2-tty6**: Text login prompts
- **tty2**: Typically where the graphical session runs (if no active text session)

### Headless Servers

A **headless server** does not have a keyboard and display that are permanently connected to it:
- Common in data centers to save space and expense
- Login provided by **serial console** on a serial port
- Connected to a networked console server for remote access
- **Examples of networked console servers:**
  - **HP iLO** (Integrated Lights-Out)
  - **Dell iDRAC** (Integrated Dell Remote Access Controller)
  - **IBM IMM** (Integrated Management Module)
  - **Supermicro IPMI** (Intelligent Platform Management Interface)
- Used when network card becomes misconfigured or network is unreachable
- Provides out-of-band management independent of the operating system
- Often accessed via Remote Desktop Protocol (RDP) for graphical interface

## Logging in to a Remote System

### Secure Shell (SSH)

The most common way to get a shell prompt on a remote system is to use **Secure Shell (SSH)**. Most Linux systems (including RHEL) and macOS provide the OpenSSH command-line program `ssh`.

### Password Authentication

```bash
user@host:~$ ssh remoteuser@remotehost
remoteuser@remotehost's password: password
remoteuser@remotehost:~$
```

The ssh command encrypts the connection to secure communication against eavesdropping or hijacking.

### Public Key Authentication

For increased security, some systems use **public key authentication** instead of passwords:

```bash
user@host:~$ ssh -i mylab.pem remoteuser@remotehost
remoteuser@remotehost:~$
```

**Key points:**
- Users have a special identity file with a **private key** (equivalent to a password)
- Account on server is configured with a matching **public key**
- Private key file must have correct permissions: `chmod 600 mylab.pem`

### Initial Login on Remote System

When connecting to a remote host for the first time:

```bash
user@host:~$ ssh -i mylab.pem remoteuser@remotehost
The authenticity of host 'remotehost (192.0.2.42)' can't be established.
ECDSA key fingerprint is 47:bf:82:cd:fa:68:06:ee:d8:83:03:1a:bb:29:14:a3.
Are you sure you want to continue connecting (yes/no)? yes
remoteuser@remotehost:~$
```

**Host Key Security:**
- Each server has unique host keys
- SSH compares host keys against saved list
- Warning appears if host key changed (possible security threat)
- Host keys protect against interceptor attacks

### Logging out of Remote System

End SSH session with:
```bash
remoteuser@remotehost:~$ exit
# OR
# Press Ctrl + D
```

---

# 2.2. Quiz - Accessing the Command Line in a Text Interface

## Questions

1. **Which term describes the interpreter that executes commands that are typed as strings?**
   - a. Command
   - b. Console
   - c. Shell ✓
   - d. Terminal

2. **Which term describes the visual cue that indicates that an interactive shell is waiting for the user to type a command?**
   - a. Argument
   - b. Command
   - c. Option
   - d. Prompt ✓

3. **Which term describes the name of a program to run?**
   - a. Argument
   - b. Command ✓
   - c. Option
   - d. Prompt

4. **Which term describes the part of the command that adjusts the behavior of a command?**
   - a. Argument
   - b. Command
   - c. Option ✓
   - d. Prompt

5. **Which term describes the part of the command that specifies the target that the command should operate on?**
   - a. Argument ✓
   - b. Command
   - c. Option
   - d. Prompt

6. **Which term describes the hardware display and keyboard to interact with a system?**
   - a. Physical Console ✓
   - b. Virtual Console
   - c. Shell
   - d. Terminal

7. **Which term describes one of multiple logical consoles that can each support an independent login session?**
   - a. Physical Console
   - b. Virtual Console ✓
   - c. Shell
   - d. Terminal

8. **Which term describes an interface that provides a display for output and a keyboard for input to a shell session?**
   - a. Console
   - b. Virtual Console
   - c. Shell
   - d. Terminal ✓

---

# 2.3. Accessing the Command Line with the Desktop Environment

## Objectives
Log in to a Linux system with the GNOME desktop environment and run commands from a shell prompt in a terminal program.

## The GNOME Desktop Environment

The **desktop environment** is the graphical user interface on a Linux system. **GNOME 47** is the default desktop environment in Red Hat Enterprise Linux 10.

### GNOME Features
- Integrated desktop for users
- Unified development platform
- Powered by Wayland graphical framework
- Highly customizable GNOME Shell

### Theme Options
- **Default**: Standard GNOME theme
- **GNOME Classic**: Closer to earlier GNOME versions
- Set theme at login via Settings icon

## Parts of the GNOME Shell

### Top Bar
- Runs along the top of the screen
- Always displayed
- Provides access to:
  - Activities Overview
  - Message tray
  - System menu

### Activities Overview
- Organize windows and start applications
- Access by:
  - Clicking Red Hat logo (upper-left corner)
  - Pressing **Super key** (Windows/Command key)

**Three main areas:**
1. **Workspace selector** (right side)
2. **Windows overview** (center)
3. **Dash** (bottom)

### Message Tray
- Displays notifications from applications/system
- Access by:
  - Clicking clock on top bar
  - Pressing **Super + M**
- Shows calendar and events

### Windows Overview
- Center area of Activities Overview
- Displays thumbnails of active windows
- Manage windows in current workspace

### Dash (Dock)
- Configurable list of application icons
- Shows:
  - Favorite applications
  - Running applications
  - Show Applications icon

### Workspace Selector
- Right side of Activities Overview
- Displays active workspaces
- Switch between workspaces

### System Menu
- Upper-right corner of top bar
- Functions:
  - Screen recording
  - Account settings
  - Lock screen
  - Log out/shut down
  - Brightness adjustment
  - Network connections
  - Dark Style toggle

## Keyboard Shortcuts

Access keyboard shortcuts via:
**System menu → Settings → Keyboard → View and Customize Shortcuts**

**Important shortcuts:**
- **Super**: Open Activities Overview
- **Super + M**: Open/close message tray
- **Super + L**: Lock screen
- **Alt + F2**: Run command dialog
- **Ctrl + Alt + LeftArrow/RightArrow**: Switch workspaces

> **Training Environment Note:** In some web-based training environments, certain keys (like Super or Ctrl + Alt combinations) may not pass through to the virtual machine. Use the on-screen keyboard available in the browser interface.

## Accessing Workspaces

**Workspaces** are separate desktop screens for organizing application windows.

### Switching Methods

**Method 1:** Keyboard shortcuts
```
Ctrl + Alt + LeftArrow    # Previous workspace
Ctrl + Alt + RightArrow   # Next workspace
```

**Method 2:** Activities Overview
1. Click Red Hat logo (upper-left)
2. Click desired workspace in workspace selector

### Workspace Configuration

Access via: **System menu → Settings → Multitasking**

**Options:**
- **Dynamic Workspaces** (default): Created/removed automatically
- **Fixed Number of Workspaces**: Always display set number

## Starting a Terminal

In RHEL 10, the **Ptyxis terminal emulator** replaces GNOME Terminal.

### Methods to start terminal:

1. **Activities Overview**: Click Terminal icon in dash
2. **Search**: Type "terminal" in Activities Overview search
3. **Run Command**: Press **Alt + F2**, type `ptyxis`

## Locking and Logging Out

### Lock Screen
- **System menu → Lock icon**
- **Super + L** shortcut
- Auto-lock after idle time

**Unlock:**
- Press Enter, Space, or click mouse
- Enter password

### Log Out
- **System menu → Power icon → Log Out**
- Confirmation dialog with cancel option

### Power Management

**Shut Down:**
- **System menu → Power icon → Power Off**
- **Ctrl + Alt + Del**
- Auto-shutdown after 60 seconds if no choice

**Reboot:**
- **System menu → Power icon → Restart**
- Auto-restart after 60 seconds if no choice

> **Update Option:** RHEL 10 may offer "Install pending software updates" option during shutdown/reboot if updates are available.

---

# 2.4. Guided Exercise - Access the Command Line with the Desktop Environment

## Outcomes
- Log in to a Linux system using GNOME desktop environment
- Run commands from shell prompt in terminal program

## Instructions

### 1. Log out of GNOME desktop
1. Click system menu (upper-right corner)
2. Click Power icon → Log Out
3. Click Log Out in confirmation dialog

### 2. Log in as student user
1. Click student user account on login screen
2. Enter password: `student`
3. Press Enter

### 3. Change password using terminal
1. Open terminal:
   - Press Super key twice OR click Red Hat logo
   - Type "terminal" and press Enter
2. Execute password change:
   ```bash
   student@workstation:~$ passwd
   Changing password for user student.
   Current password: student
   New password: redhat!23
   Retype new password: redhat!23
   passwd: all authentication tokens updated successfully.
   ```

### 4. Verify password change
1. Log out: System menu → Power → Log Out
2. Log back in with new password: `redhat!23`

### 5. Lock and unlock screen
1. Lock: System menu → Lock icon
2. Unlock: Press Enter, enter password `redhat!23`

### 6. Practice shutdown (without actually shutting down)
1. System menu → Power → Power Off
2. Click Cancel to abort

### 7. Finish exercise
```bash
student@workstation:~$ lab finish cli-desktop
```

---

# 2.5. Executing Commands with the Bash Shell

## Objectives
Save time when running commands from a Bash shell prompt by using productivity features.

## Basic Command Syntax

The GNU Bourne-Again Shell (Bash) interprets commands with up to three parts:

1. **Command** to run
2. **Options** (usually begin with `-` or `--`)
3. **Arguments** (typically targets of the command)

### Command Execution
- Each word separated by spaces
- Each command on separate line
- Press Enter to execute
- Use semicolon (`;`) to separate multiple commands on one line

**Example:**
```bash
user@host:~$ whoami
user
user@host:~$

user@host:~$ command1 ; command2
command1 output
command2 output
user@host:~$
```

## Write Simple Commands

### Date Command
```bash
user@host:~$ date
Tue Apr 29 17:58:22 UTC 2025

user@host:~$ date +%R      # 24-hour time format
17:58

user@host:~$ date +%x      # Date in MM/DD/YY format
04/29/25
```

### Password Command
```bash
user@host:~$ passwd
Current password: old_password
New password: new_password
Retype new password: new_password
passwd: password updated successfully
```

**Password Requirements:**
- Strong password required
- Mix of lowercase, uppercase, numbers, symbols
- Not based on dictionary words

### File Type Command
```bash
user@host:~$ file /etc/passwd
/etc/passwd: ASCII text

user@host:~$ file /bin/passwd
/bin/passwd: setuid ELF 64-bit LSB pie executable, x86-64...

user@host:~$ file /home
/home: directory
```

## View the Contents of Files

### Cat Command
Display file contents:
```bash
user@host:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
...

user@host:~$ cat file1 file2    # Multiple files
Hello World!!
Introduction to Linux commands.
```

### Less Command
Page through long files:
```bash
user@host:~$ less /etc/passwd
# Use arrow keys to scroll
# Press Q to exit
```

### Head and Tail Commands
```bash
user@host:~$ head /etc/passwd           # First 10 lines
user@host:~$ head -n 5 /etc/passwd      # First 5 lines
user@host:~$ tail /etc/passwd           # Last 10 lines
user@host:~$ tail -n 3 /etc/passwd      # Last 3 lines
```

### Word Count Command
```bash
user@host:~$ wc /etc/passwd
45 113 2738 /etc/passwd          # lines words characters

user@host:~$ wc -l /etc/passwd   # Lines only
45 /etc/passwd

user@host:~$ wc -w /etc/passwd   # Words only
113 /etc/passwd

user@host:~$ wc -c /etc/passwd   # Characters only
2738 /etc/passwd
```

## Tab Completion

**Tab completion** helps quickly complete commands and file names:

```bash
user@host:~$ pasTab+Tab
passt passt.avx2 pasta paste
passt-repair passwd pasta.avx2

user@host:~$ passTab+Tab
passt passt-repair passt.avx2 passwd

user@host:~$ passwTab
# Completes to passwd

user@host:~$ ls /etc/pasTab
# Completes partial file names

user@host:~$ useradd --Tab+Tab
--badnames --gid --no-log-init --shell
--base-dir --groups --non-unique --skel
...
```

## Write Long Commands on Multiple Lines

Use backslash (`\`) to continue commands on multiple lines:

```bash
user@host:~$ head -n 3 \
/usr/share/dict/words \
/usr/share/dict/linux.words
==> /usr/share/dict/words <==
1080
10-point
10th
==> /usr/share/dict/linux.words <==
1080
10-point
10th
```

## Command History

### History Command
```bash
user@host:~$ history
...
23 clear
24 who
25 pwd
26 ls /etc
27 uptime
28 ls -l
29 date
30 history
```

### History Expansion
```bash
user@host:~$ !ls           # Run most recent ls command
ls -l

user@host:~$ !26           # Run command number 26
ls /etc
```

### Navigation Keys
- **UpArrow**: Previous command
- **DownArrow**: Next command
- **LeftArrow/RightArrow**: Move cursor in current command
- **Esc + .** or **Alt + .**: Insert last word of previous command

## Command Line Editing

### Useful Editing Shortcuts

| Shortcut | Description |
|----------|-------------|
| **Ctrl + A** | Jump to beginning of command line |
| **Ctrl + E** | Jump to end of command line |
| **Ctrl + U** | Clear from cursor to beginning |
| **Ctrl + K** | Clear from cursor to end |
| **Ctrl + LeftArrow** | Jump to beginning of previous word |
| **Ctrl + RightArrow** | Jump to end of next word |
| **Ctrl + R** | Search command history |

---

# 2.6. Quiz - Executing Commands with the Bash Shell

## Questions and Answers

1. **Which Bash command displays the last five lines of the /var/log/messages file?**
   - c. `tail -n 5 /var/log/messages` ✓

2. **Which Bash shortcut or command separates commands on the same line?**
   - c. `;` ✓

3. **Which Bash command changes a user's password?**
   - c. `passwd` ✓

4. **Which Bash command displays the file type?**
   - a. `file` ✓

5. **Which Bash shortcut or command is used to complete commands, file names, and options?**
   - d. Pressing Tab ✓

6. **Which Bash shortcut or command re-executes a specific command in the history list?**
   - b. `!number` ✓

7. **Which Bash shortcut or command jumps to the beginning of the command line?**
   - e. Pressing Ctrl + A ✓

8. **Which Bash shortcut or command displays the list of previously executed commands?**
   - d. `history` ✓

9. **Which Bash shortcut or command copies the last argument of previous commands?**
   - d. Pressing Esc + . ✓

---

# 2.7. Lab - Access the Command Line

## Objectives
- Successfully run simple programs from the Bash shell command line
- Execute commands to identify file types and display parts of text files
- Practice using Bash command history shortcuts

## Lab Instructions

### Prerequisites
```bash
student@workstation:~$ lab start cli-review
```

### Tasks

1. **Display current time and date**
   ```bash
   student@workstation:~$ date
   ```

2. **Display current time in 24-hour format**
   ```bash
   student@workstation:~$ date +%R
   ```

3. **Determine file type of /home/student/zcat**
   ```bash
   student@workstation:~$ file zcat
   zcat: a /usr/bin/sh script, ASCII text executable
   ```
   *Answer: ASCII text files are human readable.*

4. **Display size of zcat file using wc command**
   ```bash
   student@workstation:~$ wc zcat    # Use Esc + . for filename
   51 298 1977 zcat
   ```

5. **Display first 10 lines**
   ```bash
   student@workstation:~$ head zcat   # Use Esc + . again
   ```

6. **Display last 10 lines**
   ```bash
   student@workstation:~$ tail zcat   # Use Esc + . again
   ```

7. **Repeat previous command with ≤4 keystrokes**
   ```bash
   # Method 1: UpArrow + Enter (2 keystrokes)
   # Method 2: !! + Enter (4 keystrokes)
   student@workstation:~$ !!
   ```

8. **Display last 20 lines using command-line editing**
   ```bash
   # Use UpArrow to get previous command
   # Use Ctrl + A to go to beginning
   # Use Ctrl + RightArrow to jump to next word
   # Add -n 20 option
   student@workstation:~$ tail -n 20 zcat
   ```

9. **Re-run date +%R command using history**
   ```bash
   student@workstation:~$ history
   # Identify command number for date +%R
   student@workstation:~$ !2    # Replace 2 with actual number
   ```

### Evaluation
```bash
student@workstation:~$ lab grade cli-review
```

### Finish
```bash
student@workstation:~$ lab finish cli-review
```

---

# 2.8. Summary

## Key Learnings

### Bash Shell Fundamentals
- **Bash shell** is a command interpreter for interactive Linux command execution
- Provides powerful scripting language and task automation capabilities
- More sophisticated than Windows cmd.exe, similar to PowerShell

### Basic Commands and Navigation
- **Command structure**: `command [options] [arguments]`
- **File operations**: `cat`, `less`, `head`, `tail`, `wc`, `file`
- **System commands**: `date`, `passwd`, `whoami`

### Productivity Features
- **Tab completion** for commands, filenames, and options
- **Command history** with `!number` and `!string` expansion
- **Command-line editing** shortcuts for efficient navigation
- **Multi-line commands** using backslash continuation

### GNOME Desktop Environment
- **Activities Overview** for window and application management
- **Workspaces** for organizing multiple application windows
- **Terminal access** via Ptyxis terminal emulator
- **System management** through graphical interface

### Login Methods
- **Local login**: Physical console with virtual consoles (tty1-tty6)
- **Remote login**: SSH with password or public key authentication
- **Security**: Host key verification and encrypted communication

### Best Practices
- Use graphical interface for many administrative tasks
- Command line provides powerful automation capabilities
- Disable graphical interface on servers to preserve resources
- Combine GUI and CLI approaches based on task requirements

## References
- bash(1), date(1), file(1), cat(1), less(1), head(1), tail(1), wc(1), passwd(1) man pages
- Using Secure Communications between Two Systems with OpenSSH guide
- Using the GNOME Desktop Environment guide