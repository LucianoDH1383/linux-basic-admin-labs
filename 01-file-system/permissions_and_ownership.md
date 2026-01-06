#🔍 Checking permissions / Verificar permisos

## How to check current permissions of files and directories.
Cómo verificar los permisos actuales de archivos y directorios.
```bash

ls -l                  # Basic list with permissions / Lista básica con permisos
ls -la                 # Include hidden files / Incluir archivos ocultos
ls -lh                 # Human readable sizes / Tamaños legibles para humanos
ls -l /etc/passwd      # Check specific file / Verificar archivo específico

📝 Understanding the output / Entender la salida

Decoding the permission string (like drwxr-xr-x).
Decodificando la cadena de permisos (como drwxr-xr-x).
text

-rw-r--r-- 1 user group 1024 Jan 1 12:00 file.txt
↑ ↑↑↑ ↑↑↑ ↑↑↑
│ │││ │││ ││└─ Others: r-- (read only) / Otros: r-- (solo lectura)
│ │││ │││ │└── Group: r-- (read only) / Grupo: r-- (solo lectura)
│ │││ │││ └─── Owner: rw- (read & write) / Propietario: rw- (lectura y escritura)
│ │││ ││└───── Execute bit for directories / Bit de ejecución para directorios
│ │││ │└────── Directory indicator (d=directory, -=file, l=link) / Indicador de directorio
│ └┴┴┴┴─────── File type and permissions / Tipo de archivo y permisos
```
## Changing permissions with chmod / Cambiar permisos con chmod

Most common permissions you'll use:
Permisos más comunes que usarás:
```bash

# For scripts and programs / Para scripts y programas:
chmod 755 script.sh           # Owner: rwx, Group: r-x, Others: r-x
chmod +x script.sh            # Add execute permission to all / Agregar permiso de ejecución a todos

# For configuration files / Para archivos de configuración:
chmod 644 config.conf         # Owner: rw-, Group: r--, Others: r--
chmod 600 private.key         # Owner: rw-, Group: ---, Others: ---

# For directories / Para directorios:
chmod 755 /var/www/           # Standard web directory / Directorio web estándar
chmod 777 /tmp/backup/        # Temporary (be careful!) / Temporal (¡ten cuidado!)

# Recursive changes (entire directory) / Cambios recursivos (directorio completo):
chmod -R 755 /opt/myapp/      # Apply to all files and subdirectories / Aplicar a todos los archivos y subdirectorios
chmod -R u=rwX,g=rX,o=rX dir/ # Smart recursive (X adds execute only to dirs) / Recursivo inteligente (X agrega ejecución solo a directorios)
```
## Changing ownership with chown / Cambiar propiedad con chown

Common scenarios you'll encounter:
Escenarios comunes que encontrarás:
```bash

# Fix web server permissions (very common) / Corregir permisos del servidor web (muy común):
sudo chown -R www-data:www-data /var/www/html/

# Give a user access to a directory / Dar acceso a un usuario a un directorio:
sudo chown -R john:developers /opt/project/

# Change only the group / Cambiar solo el grupo:
sudo chown :apache /var/log/httpd/

# Copy ownership from another file / Copiar propiedad de otro archivo:
sudo chown --reference=/etc/nginx/nginx.conf /etc/nginx/sites-available/
```
##Changing group with chgrp / Cambiar grupo con chgrp
```bash

sudo chgrp developers /shared/folder/
sudo chgrp -R apache /var/www/
```
## Common permission issues and fixes / Problemas comunes de permisos y soluciones

Problem: "Permission denied" when running script
Problema: "Permiso denegado" al ejecutar script
```bash
chmod +x /path/to/script.sh

Problem: Web server can't write to directory
Problema: El servidor web no puede escribir en el directorio
bash

sudo chown -R www-data:www-data /var/www/uploads/
sudo chmod -R 775 /var/www/uploads/
```
## Problem: Can't read a file as regular user
Problema: No puedo leer un archivo como usuario regular
```bash

sudo chmod 644 /etc/config/file.conf
# OR give your user access / O dar acceso a tu usuario:
sudo chown youruser:youruser /etc/config/file.conf

Problem: Directory not accessible
Problema: Directorio no accesible
```bash

chmod 755 /path/to/directory/
```
## Security best practices / Mejores prácticas de seguridad
bash
```
# NEVER do this (security risk) / NUNCA hagas esto (riesgo de seguridad):
chmod 777 /any/path/           # ❌ Anyone can do anything / ❌ Cualquiera puede hacer cualquier cosa
chmod -R 777 /                 # ❌❌❌ NEVER DO THIS! / ❌❌❌ ¡NUNCA HAGAS ESTO!

# Instead, do this / En su lugar, haz esto:
chmod 755 directories/         # ✅ Safe for directories / ✅ Seguro para directorios
chmod 644 files.txt           # ✅ Safe for regular files / ✅ Seguro para archivos regulares
chmod 600 secrets.txt         # ✅ Private files / ✅ Archivos privados

📋 Quick reference table / Tabla de referencia rápida
Permission / Permiso	Numeric / Numérico	Description / Descripción
Read + Write + Execute / Lectura + Escritura + Ejecución	7	rwx - Full access / Acceso completo
Read + Write / Lectura + Escritura	6	rw- - Can read and write / Puede leer y escribir
Read + Execute / Lectura + Ejecución	5	r-x - Can read and execute / Puede leer y ejecutar
Read only / Solo lectura	4	r-- - Read only / Solo lectura
Write only / Solo escritura	2	-w- - Write only (rare) / Solo escritura (raro)
Execute only / Solo ejecución	1	--x - Execute only / Solo ejecución
```
## Common combinations / Combinaciones comunes:
```
    755 - Scripts, programs, directories / Scripts, programas, directorios

    644 - Configuration files, documents / Archivos de configuración, documentos

    600 - Private keys, sensitive data / Claves privadas, datos sensibles

    775 - Shared directories with group write / Directorios compartidos con escritura de grupo
```
## Real-world examples / Ejemplos del mundo real
```bash

# 1. Deploying a website / Desplegando un sitio web:
sudo chown -R www-data:www-data /var/www/mywebsite/
sudo find /var/www/mywebsite/ -type f -exec chmod 644 {} \;
sudo find /var/www/mywebsite/ -type d -exec chmod 755 {} \;

# 2. Setting up a shared project folder / Configurando una carpeta de proyecto compartida:
sudo mkdir /shared/project
sudo chown -R :developers /shared/project
sudo chmod -R 2775 /shared/project  # SetGID for group inheritance / SetGID para herencia de grupo

# 3. Securing SSH keys / Asegurando claves SSH:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 644 ~/.ssh/known_hosts

# 4. Log files (rotation ready) / Archivos de log (listos para rotación):
sudo chown root:adm /var/log/
sudo chmod 755 /var/log/
sudo chmod 640 /var/log/syslog
```
## Quick diagnostic commands / Comandos de diagnóstico rápido
```bash

# What permissions do I have here? / ¿Qué permisos tengo aquí?
ls -la

# Who owns this? / ¿Quién es el dueño de esto?
ls -l /path/to/file

# Can I run this? / ¿Puedo ejecutar esto?
ls -l /usr/bin/command

# Check specific permission / Verificar permiso específico:
test -r /file.txt && echo "Readable" || echo "Not readable"
test -w /file.txt && echo "Writable" || echo "Not writable"
test -x /file.txt && echo "Executable" || echo "Not executable"
```
## When to use sudo / Cuándo usar sudo
```bash

# Need sudo when / Necesitas sudo cuando:
# 1. Changing files not owned by you / Cambias archivos que no te pertenecen
sudo chown user:user /etc/config/file

# 2. Changing system directories / Cambias directorios del sistema
sudo chmod 755 /usr/local/bin/

# 3. Working with services / Trabajas con servicios
sudo chown -R mysql:mysql /var/lib/mysql/

# DON'T use sudo for / NO uses sudo para:
# Your own home directory files / Archivos de tu propio directorio home
chmod 600 ~/.bashrc  # ✅ No sudo needed / ✅ No se necesita sudo

## Permission numeric values / Valores numéricos de permisos
```bash

# Each permission has a value / Cada permiso tiene un valor:
# Read (r) = 4
# Write (w) = 2  
# Execute (x) = 1
# No permission (-) = 0

# Calculate like this / Calcula así:
# Owner: rwx = 4+2+1 = 7
# Group: r-x = 4+0+1 = 5
# Others: r-x = 4+0+1 = 5
# Result: 755
```
## Special permissions / Permisos especiales
```bash

# SetUID - Runs as file owner / Ejecuta como propietario del archivo
chmod u+s /usr/bin/passwd      # 4755
chmod 4755 /usr/bin/sudo

# SetGID - Files inherit directory group / Archivos heredan grupo del directorio
chmod g+s /shared/              # 2775
chmod 2775 /var/www/html/

# Sticky bit - Only owner can delete / Solo el dueño puede borrar
chmod +t /tmp                   # 1777
chmod 1777 /var/tmp/
```
## Default permissions (umask) / Permisos por defecto (umask)
```bash

# Show current umask / Mostrar umask actual
umask
umask -S  # Symbolic form / Forma simbólica

# Set umask for session / Establecer umask para la sesión
umask 022  # Files: 644, Dirs: 755 / Archivos: 644, Directorios: 755
umask 027  # Files: 640, Dirs: 750 / Archivos: 640, Directorios: 750

# Permanent umask in ~/.bashrc / Umask permanente en ~/.bashrc
echo "umask 022" >> ~/.bashrc
```
##Viewing effective permissions / Ver permisos efectivos
```bash

# Check if you can actually access / Verificar si realmente puedes acceder
namei -l /path/to/file
getfacl /path/to/directory  # For ACLs / Para ACLs

# Test permissions directly / Probar permisos directamente
[ -r /file.txt ] && echo "Can read" || echo "Cannot read"
[ -w /file.txt ] && echo "Can write" || echo "Cannot write"
[ -x /file.txt ] && echo "Can execute" || echo "Cannot execute"
```
