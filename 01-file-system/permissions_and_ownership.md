# Permissions and Ownership in Linux

## Viewing permissions
Shows file and directory permissions.
```bash
ls -l
```

## Changing permissions with chmod
Modifies file permissions using symbolic or numeric modes.
```bash
chmod u+x script.sh
chmod 755 script.sh
```

## Changing ownership with chown
Changes file owner and group.
```bash
sudo chown user:user file.txt
```

## Changing group ownership
Changes the group associated with a file.
```bash
sudo chgrp group file.txt
```
