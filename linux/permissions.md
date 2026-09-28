# Linux Permissions Lab

## Objective

The goal of this lab was to understand how Linux handles:

* File and directory ownership
* Owner, group, and others permissions
* Directory traversal
* `umask`
* Shared group access
* setgid directories
* File deletion behavior
* Sticky-bit protected directories

The lab used multiple local users to test how permissions behave in practice.

---

## Test Users and Groups

Two test users were created:

* `alice`
* `bob`

A shared group named `developers` was also created, and both users were added to it.

Verification:

```bash
getent group developers
groups alice
groups bob
```

The group membership was:

```text
developers:x:1003:alice,bob
```

This allowed Alice and Bob to share access through group permissions.

---

## Directory Traversal

A file was created under the original user's home directory.

While logged in as Alice, attempting to access the file resulted in:

```text
Permission denied
```

The original user's home directory had permissions similar to:

```text
drwxr-x---
```

This means:

```text
Owner:  rwx
Group:  r-x
Others: ---
```

Alice was neither the owner nor a member of the directory's group, so she received the `others` permissions:

```text
---
```

As a result, she could not traverse the directory.

### Lesson

A user must have sufficient permissions on every directory in a file's path.

For directories:

* `r` allows directory contents to be listed
* `w` allows directory entries to be created or removed
* `x` allows the directory to be traversed

Even if a file itself is readable, a user cannot access it if they cannot traverse one of its parent directories.

---

## File Permissions

The test file originally had permissions similar to:

```text
-rw-rw-r--
```

This represents:

```text
Owner:  rw-
Group:  rw-
Others: r--
```

Once Alice was allowed to traverse the directory path, she could read the file because she matched the `others` permission class and `others` had read permission.

After removing read permission from `others`:

```bash
chmod o-r secret.txt
```

the file became:

```text
-rw-rw----
```

Alice could still reach the file, but could no longer read it.

### Lesson

Directory permissions determine whether a user can reach a file.

File permissions determine what the user can do with the file after reaching it.

---

## umask

The system's umask was:

```text
0002
```

The first digit represents special permission bits, while the remaining digits correspond to:

```text
Owner
Group
Others
```

A `0002` umask removes write permission from `others`.

New directories normally begin with a maximum mode of:

```text
777
```

Applying `0002` results in:

```text
775
rwxrwxr-x
```

New regular files normally begin with:

```text
666
```

Applying `0002` results in:

```text
664
rw-rw-r--
```

This matched the permissions observed during the lab.

### Lesson

New files and directories do not simply inherit their parent's permissions.

Their initial permissions are primarily determined by the creation mode requested by the application and the user's `umask`.

---

## Shared Directory

A shared directory was created for the `developers` group.

Its ownership was configured as:

```text
Owner: <original-user>
Group: developers
```

Permissions were set to:

```bash
chmod 770 shared
```

This resulted in:

```text
drwxrwx---
```

Meaning:

```text
Owner:  rwx
Group:  rwx
Others: ---
```

Alice and Bob could both enter and modify the shared directory because they were members of `developers`.

---

## Group Ownership Problem

Alice created a file inside the shared directory.

The new file appeared as:

```text
-rw-rw-r-- alice alice alice.txt
```

Even though the parent directory belonged to `developers`, the new file used Alice's primary group.

This meant Bob did not receive the group permissions on the file.

Instead, Bob matched the `others` permissions:

```text
r--
```

Bob could read the file but could not modify it.

### Lesson

A normal directory does not automatically force new files to inherit its group ownership.

---

## setgid Directory

The shared directory was changed to:

```bash
sudo chmod 2770 shared
```

The leading `2` enables the setgid bit.

The directory then appeared as:

```text
drwxrws---
```

The `s` in the group execute position indicates setgid is enabled.

A new file created by Alice now appeared as:

```text
-rw-rw-r-- alice developers alice2.txt
```

Because Bob was also a member of `developers`, he received the file's group permissions and could successfully append to it.

### Lesson

When setgid is enabled on a directory, newly created files and directories inherit the parent directory's group.

This is useful for shared project directories.

---

## setgid Permission Restriction

Initially, running:

```bash
chmod 2770 shared
```

did not preserve the setgid bit.

The original user owned the directory but was not a member of the `developers` group.

Running:

```bash
sudo chmod 2770 shared
```

successfully enabled setgid.

### Lesson

Owning a directory does not automatically give a user unrestricted authority over its group identity.

Linux applies additional security restrictions to special permission bits such as setgid.

---

## File Deletion Experiment

Alice created a file and restricted it to:

```bash
chmod 400 delete-test.txt
```

The resulting permissions were:

```text
-r--------
```

Bob could not:

* Read the file
* Modify the file

However, Bob could still delete it because he had write and execute permission on the containing shared directory.

The `rm` command warned that the file was write-protected, but after confirmation the deletion succeeded.

### Lesson

File write permission controls modification of the file's contents.

Deletion is primarily controlled by the permissions of the containing directory.

A user with sufficient write and execute permissions on a directory may be able to remove files they do not own.

---

## Sticky Bit

To prevent users from deleting each other's files, the shared directory was changed to:

```bash
sudo chmod 3770 shared
```

The leading `3` represents:

```text
2 = setgid
1 = sticky bit
```

The directory appeared as:

```text
drwxrws--T
```

The capital `T` indicated that the sticky bit was enabled while `others` did not have execute permission.

After enabling the sticky bit, Bob could no longer delete a file owned by Alice.

The kernel rejected the operation.

### Lesson

The sticky bit is useful on shared writable directories.

It allows users to create files while preventing them from deleting files owned by other users.

A common real-world example is `/tmp`.

---

## Key Takeaways

This lab demonstrated several important Linux permission concepts:

* File permissions and directory permissions serve different purposes.
* Directory execute permission controls traversal.
* Access to a file depends on every directory in its path.
* `umask` influences the permissions of newly created objects.
* Linux groups provide scalable shared access.
* setgid can force new files to inherit a shared group.
* Directory permissions control file creation, deletion, and renaming.
* The sticky bit protects files in shared writable directories.

The most important lesson was that Linux permissions should be understood as a system of interacting rules rather than as permissions on individual files alone.
