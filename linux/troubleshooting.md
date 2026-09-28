# Troubleshooting Notes

## Alice Could Not Access a File

### Problem

While logged in as `alice`, I attempted to access a file under:

`/home/snowbear/linux-lab/permissions/secret.txt`

and received:

`Permission denied`

### Investigation

I checked:

- the current user with `whoami`
- the home directory permissions with `ls -ld`
- file ownership with `ls -l`

The original user's home directory had permissions similar to:

`drwxr-x---`

Alice was neither the owner nor a member of the directory's group, so she had no traverse permission on that directory.

### Resolution

I temporarily added execute permission for others on the home directory:

`chmod o+x /home/snowbear`

This allowed Alice to traverse the directory path.

### Lesson

File permissions alone do not determine access. A user must also have sufficient permissions on every parent directory in the path.

For directories, the execute (`x`) bit controls traversal.
