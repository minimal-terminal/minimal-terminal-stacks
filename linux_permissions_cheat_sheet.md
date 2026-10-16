# Linux File Permissions & Ownership Cheat Sheet

Understanding Linux file permissions and ownership is critical for system security, administration, and troubleshooting. This cheat sheet covers the foundational concepts and essential commands.

## 1. Permission Types

Each file and directory has three types of permissions:

- **Read (r):** Allows viewing the content of a file or listing the contents of a directory.
  - Octal Value: `4`
- **Write (w):** Allows modifying the content of a file or creating/deleting files within a directory.
  - Octal Value: `2`
- **Execute (x):** Allows running a file as a program or accessing a directory (e.g., `cd` into it).
  - Octal Value: `1`

## 2. Ownership Categories

Permissions are applied to three categories of users:

- **User (u):** The owner of the file or directory.
- **Group (g):** The group associated with the file or directory.
- **Others (o):** All other users on the system.

## 3. Viewing Permissions

Use the `ls -l` command to view detailed file and directory information, including permissions.

```bash
ls -l /path/to/file_or_directory
```

**Example Output Breakdown:**

```
-rwxr-xr-- 1 user group 1024 Jan 1 10:00 myfile.txt
```

- **`-`**: File type (`-` for regular file, `d` for directory, `l` for symbolic link, etc.)
- **`rwx`**: Permissions for the **User** (owner)
- **`r-x`**: Permissions for the **Group**
- **`r--`**: Permissions for **Others**

## 4. Changing Permissions (`chmod`)

The `chmod` command is used to change file and directory permissions. It can be used with symbolic modes (u, g, o, a for all; +, -, = for add, remove, set) or octal (numeric) modes.

### Symbolic Mode

- `chmod u+x script.sh`: Add execute permission for the user.
- `chmod go-w directory/`: Remove write permission for group and others.
- `chmod a=rwx file.txt`: Set read, write, execute permissions for all.

### Octal (Numeric) Mode

Octal mode combines the numeric values of permissions for each category (User, Group, Others).

- `chmod 755 script.sh`: `rwxr-xr-x` (User: Read, Write, Execute; Group: Read, Execute; Others: Read, Execute)
  - `7` (user) = 4+2+1 (rwx)
  - `5` (group) = 4+0+1 (r-x)
  - `5` (others) = 4+0+1 (r-x)

**Common Octal Modes:**

- **`777`**: `rwxrwxrwx` (Full permissions for all - generally discouraged)
- **`755`**: `rwxr-xr-x` (Executable for owner, readable/executable for group/others - common for scripts/directories)
- **`644`**: `rw-r--r--` (Readable/writable for owner, readable for group/others - common for files)
- **`600`**: `rw-------` (Readable/writable only for owner - common for sensitive files)

## 5. Changing Ownership (`chown`)

The `chown` command changes the user owner and/or group owner of a file or directory.

- `chown user file.txt`: Change only the user owner.
- `chown :group file.txt`: Change only the group owner.
- `chown user:group file.txt`: Change both user and group owner.
- `chown -R user:group directory/`: Recursively change ownership for a directory and its contents.

## 6. Changing Group (`chgrp`)

The `chgrp` command is a shortcut to change only the group owner.

- `chgrp newgroup file.txt`: Change the group owner to `newgroup`.

## 7. Default Permissions (`umask`)

`umask` sets the default file permissions for newly created files and directories. It's a mask that *removes* permissions from the maximum default (666 for files, 777 for directories).

- **View current umask:** `umask` (e.g., `0022`)
  - `0022` means no special permissions, and `022` implies: `w` for group, `w` for others are removed from default.
  - Default file permissions: `666 - 022 = 644` (rw-r--r--)
  - Default directory permissions: `777 - 022 = 755` (rwxr-xr-x)

- **Set umask:** `umask 002`: Set default permissions to `664` for files and `775` for directories.

## 8. Special Permissions

These add extra security or functionality:

- **SUID (Set User ID):** When an executable file with SUID is run, it executes with the permissions of the file owner, not the user running it. (Octal: `4000`)
  - `chmod u+s program` or `chmod 4755 program`
- **SGID (Set Group ID):** Similar to SUID, but for group permissions. For directories, new files/subdirectories inherit the parent's group. (Octal: `2000`)
  - `chmod g+s directory` or `chmod 2775 directory`
- **Sticky Bit:** For directories, only the owner of a file (or root) can delete or rename files within that directory, even if others have write permission. (Octal: `1000`)
  - `chmod +t /tmp` or `chmod 1777 /tmp`

**Example with Special Permissions:**

```
-rwsr-xr-x 1 root root 50K Jan 1 10:00 /usr/bin/passwd
```

- `s` in the user's execute field indicates SUID is set.