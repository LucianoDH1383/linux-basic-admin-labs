# Permissions and Ownership in Linux / Permisos y Propiedad en Linux

## Viewing permissions / Ver permisos
Shows file and directory permissions.
Muestra los permisos de archivos y directorios.
```bash
ls -l
```

## Changing permissions with chmod / Cambiar permisos con chmod
Modifies file permissions using symbolic or numeric modes.
Modifica los permisos usando modos simbólicos o numéricos.
```bash
chmod u+x script.sh
chmod 755 script.sh
```

## Changing ownership with chown / Cambiar propietario con chown
Changes file owner and group.
Cambia el usuario propietario y el grupo de un archivo.
```bash
sudo chown user:user file.txt
```

## Changing group ownership / Cambiar grupo propietario
Changes the group associated with a file.
Cambia el grupo asociado a un archivo.
```bash
sudo chgrp group file.txt
```
