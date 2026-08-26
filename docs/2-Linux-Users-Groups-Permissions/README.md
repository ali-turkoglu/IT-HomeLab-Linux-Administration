# Phase 2 – Linux Users, Groups & Permissions

## Purpose & Objectives

Linux uses a user, group, and permission model to control access to files and directories.

In this phase, I configured local users, groups, and permissions to simulate a small corporate environment and practice access control. These local accounts are temporary for testing purposes. In a later phase, they will be replaced by centralized Active Directory users when the server is integrated into my `homelab.local` domain.

The main objectives of this phase were:

- Understand Linux users, primary groups, and secondary groups.
- Manage file and directory ownership with `chown`.
- Apply `rwx` permissions using `chmod`, including numeric permissions such as `770`.
- Configure group-based project directories.
- Implement `setgid` for shared directory group inheritance.
- Understand the role of `umask`.
- Manage administrative access using `sudo`.
- Apply the Principle of Least Privilege (PoLP) using `/etc/sudoers.d/`.

---

## Environment

- **Operating System:** Ubuntu Server 26.04 LTS
- **Hostname:** `ubuntu01`
- **Domain:** `homelab.local` *(planned for a later phase)*
- **Access:** SSH from macOS workstation
- **Shell:** Bash

---

## Architecture & Implementation

### 1. User and Group Roles

Linux identifies users with a unique **UID (User ID)**. Each user has a primary group and can also belong to additional secondary groups.

I organized the lab users into simple business roles to practice group-based access control:

    Users
    ├── Developers
    │   ├── alice
    │   └── bob
    │
    ├── Operations
    │   ├── charlie
    │   └── david
    │
    ├── Manager
    │   └── eric
    │
    └── Security/Admin
        └── frank

`Developers` and `Operations` are Linux groups used for access control.

`Manager` and `Security/Admin` are role descriptions for the lab users and are not separate Linux groups.

The actual Linux group structure is:

    developers
    ├── alice
    └── bob

    operations
    ├── charlie
    └── david

This is a simple RBAC-style simulation for learning purposes. The final centralized identity and access model will be implemented later using Active Directory.

### 2. User Management

Users were created with `adduser`.

Example:

    sudo adduser alice

A user's UID, primary group, and secondary groups can be checked with:

    id alice

Example output:

    uid=1001(alice) gid=1001(alice) groups=1001(alice),100(users),1002(developers)

The `groups` command can also be used to display group membership:

    groups alice

### 3. Group Management

Linux groups were created with `addgroup`:

    sudo addgroup developers
    sudo addgroup operations

Users were then added to the appropriate groups:

    sudo adduser alice developers
    sudo adduser bob developers
    sudo adduser charlie operations
    sudo adduser david operations

Group information can be checked with `getent`:

    getent group developers
    getent group operations

This provides a simple group-based access model:

    developers
    ├── alice
    └── bob

    operations
    ├── charlie
    └── david

---

## 4. Linux Ownership and Permissions

Every Linux file and directory has an owner and a group.

Permissions are divided into three categories:

    Owner | Group | Others

For example:

    -rwxr-x---

means:

    Owner  → rwx
    Group  → r-x
    Others → ---

### Permission Symbols

    r = read
    w = write
    x = execute

The meaning of `x` depends on the object:

- For a file: execute the file
- For a directory: enter or access the directory

### Numeric Permissions

The numeric values are:

    r = 4
    w = 2
    x = 1

They can be combined:

    --- = 0
    --x = 1
    -w- = 2
    -wx = 3
    r-- = 4
    r-x = 5
    rw- = 6
    rwx = 7

The three digits represent:

    User | Group | Others

For example, `770` means:

    Owner  → rwx (7)
    Group  → rwx (7)
    Others → --- (0)

---

## 5. Project Directories

Two project directories were created:

    /srv/projects/
    ├── development/
    └── operations/

The Development directory was configured as:

    Owner  → alice
    Group  → developers
    Mode   → 770

The Operations directory was configured as:

    Owner  → charlie
    Group  → operations
    Mode   → 770
    Setgid → enabled

### Changing Ownership

The `chown` command changes the owner and group:

    sudo chown charlie:operations /srv/projects/operations

The result can be checked with:

    ls -ld /srv/projects/operations

### Changing Permissions

The `chmod` command changes permissions:

    sudo chmod 770 /srv/projects/development
    sudo chmod 770 /srv/projects/operations

This allows the owner and members of the project group to access the directory while other users are denied.

---

## 6. Group-Based Access Testing

The permission model was tested using users from different groups.

Alice is a member of the `developers` group:

    sudo -u alice ls /srv/projects/development

Access was successful.

Bob is also a member of `developers` and can access the same directory.

Eric is not a member of `developers`:

    sudo -u eric ls /srv/projects/development

The result was:

    Permission denied

The same access model was tested with the Operations directory:

    Operations Directory
    │
    ├── Owner  → charlie
    ├── Group  → operations
    └── Others → ---

Therefore:

    charlie → Owner → Access
    david   → Group → Access
    alice   → Other → Denied
    eric    → Other → Denied

This demonstrated how Linux uses the Owner, Group, and Others permission categories.

---

## 7. Setgid for Shared Directories

### What is setgid?

`setgid` means **Set Group ID**.

When setgid is enabled on a directory, newly created files and directories inherit the group ownership of the parent directory.

This is useful for shared project directories.

The Operations directory was configured with:

    sudo chmod g+s /srv/projects/operations

Here:

    g = group
    + = add
    s = setgid

The directory then showed:

    drwxrws---

The `s` appears in the group execute position.

Before setgid, a file created by David could have:

    david:david

After enabling setgid, a new file created by David inherited:

    david:operations

This was verified by creating and checking a new file in the Operations directory.

Setgid was used only on the Operations directory in this phase to demonstrate group inheritance.

---

## 8. umask

`umask` is a **user file-creation mode mask**.

It controls which permissions are restricted when new files and directories are created.

The current value can be checked with:

    umask

For example:

    0022

The basic permission values are:

    1 → execute
    2 → write
    4 → read

The values can be combined to define which permissions are masked.

For example:

    0007

would mask all permissions for `Others`.

`umask` was studied in this phase to understand how default permissions are created. It was not permanently changed for the HomeLab users.

For shared directories, `setgid` and an appropriate `umask` can be used together to create more consistent group-based file access.

---

## 9. Sudo and Administrative Access

Linux file permissions control normal user access to files and directories.

`sudo` allows an authorized user to run commands with administrative privileges.

Ubuntu commonly uses the `sudo` group for administrative access.

The default sudo group rule can be checked with:

    sudo grep '^%sudo' /etc/sudoers

The result was:

    %sudo ALL=(ALL:ALL) ALL

This means members of the `sudo` group can use `sudo` to run commands with administrative privileges.

The basic model is:

    User
    │
    ├── Primary Group
    ├── Secondary Groups
    │      ├── developers
    │      └── operations
    │
    └── sudo
           ↓
       Administrative Access

File permissions and `sudo` provide different types of access control.

A user can have access to a project directory through group membership without having administrative access to the server.

---

## 10. Limited Administrative Access with `sudoers.d`

Full sudo access is not always necessary.

More specific administrative permissions can be configured using:

    /etc/sudoers
    /etc/sudoers.d/

A separate sudoers file was created for Frank:

    sudo visudo -f /etc/sudoers.d/frank

The following rule was added:

    frank ALL=(root) /usr/bin/systemctl status ssh

This allows Frank to run:

    sudo systemctl status ssh

but does not allow him to run other administrative commands such as:

    sudo systemctl restart ssh

The allowed command was successful, while the restart command was denied.

This demonstrates the **Principle of Least Privilege (PoLP)**: users should receive only the administrative permissions they need.

`visudo` was used instead of editing sudoers files directly because it checks the configuration syntax before saving.

---

## 11. Final User and Access Model

Alice was initially added to the `sudo` group for testing. After the test, the administrative access was removed:

    sudo deluser alice sudo

The final access model is:

| User | Role | Project Access | Admin Access |
| :--- | :--- | :--- | :--- |
| **alice** | Developer | Development | ❌ None |
| **bob** | Developer | Development | ❌ None |
| **charlie** | Operations | Operations | ❌ None |
| **david** | Operations | Operations | ❌ None |
| **eric** | Manager | None | ❌ None |
| **frank** | Security/Admin | None | ⚠️ Limited |
| **aliturkoglu** | Lab Administrator | — | 👑 Full sudo |

### Project Directory Configuration

    /srv/projects/development
    Owner  → alice
    Group  → developers
    Mode   → 770
    Setgid → disabled

    /srv/projects/operations
    Owner  → charlie
    Group  → operations
    Mode   → 770
    Setgid → enabled

---

## Testing & Verification

The following tests were performed during the phase:

- [x] Created and verified local users and groups.
- [x] Verified primary and secondary group membership.
- [x] Tested file and directory ownership.
- [x] Tested `770` permissions and Owner, Group, and Others access.
- [x] Verified group-based access to project directories.
- [x] Tested `setgid` group inheritance.
- [x] Tested full `sudo` administrative access.
- [x] Tested limited sudo access with `sudoers.d`.
- [x] Removed unnecessary administrative access from Alice.
- [x] Removed temporary test files after verification.

---

## Lessons Learned

This phase demonstrated how Linux controls access using users, groups, ownership, and permissions.

The main lessons were:

- Users and groups are separate concepts; a user can belong to multiple groups.
- `rwx` permissions have different meanings for files and directories.
- `chown` changes ownership, while `chmod` changes permissions.
- `setgid` helps shared directories keep consistent group ownership.
- `umask` controls default permissions for newly created files and directories.
- `/etc/sudoers.d/` can provide more limited administrative permissions compared to the broad `sudo` group.
- File permissions and administrative privileges are separate concepts.

---

## Result

The Ubuntu Server now has a basic local Linux user and permission structure suitable for further administration practice.

The Development and Operations directories use group-based access control, while the Operations directory uses setgid for group inheritance.

These local accounts are intentionally part of the learning environment and are not the final identity solution for the HomeLab.

In a later phase, the Ubuntu Server will be integrated into the existing `homelab.local` Active Directory domain. At that stage, domain users and groups will be used for centralized authentication and access control instead of relying on these temporary local accounts.

---

## Useful Commands

The following commands were used during this phase:

    adduser
    addgroup
    id
    groups
    getent
    ls -l
    ls -ld
    chown
    chmod
    umask
    sudo
    visudo

Useful configuration files and directories:

    /etc/passwd
    /etc/group
    /etc/sudoers
    /etc/sudoers.d/

    ---

## Navigation

| Previous | Home | Next |
|:--------:|:----:|:----:|
| ⬅️ [1-Ubuntu-Server-Installation-Base-Configuration](../1-Ubuntu-Server-Installation-Base-Configuration/README.md) | 🏠 [Home](../../README.md) | ➡️ [Phase 3 – Linux Filesystem & Storage Basics](../3-Linux-Filesystem-Storage-Basics/README.md) |
