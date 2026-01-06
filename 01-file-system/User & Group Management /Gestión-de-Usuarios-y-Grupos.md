 # User & Group Management / Gestión de Usuarios y Grupos
📋 Índice / Table of Contents

    Ver usuarios del sistema / View system users

    Crear usuarios / Create users

    Modificar usuarios / Modify users

    Eliminar usuarios / Delete users

    Gestión de grupos / Group management

    Permisos de usuarios / User permissions

    Escenarios comunes / Common scenarios

## 1. Ver usuarios del sistema / View system users
Listar usuarios conectados / List logged in users
```bash

who                    # Usuarios conectados actualmente
w                      # Más detalles (qué están haciendo)
last                   # Historial de conexiones
lastlog                # Último login de cada usuario
```
## Ver información de usuario específico / View specific user info
```bash

id                     # Mi información de usuario
id username            # Información de otro usuario
whoami                 # ¿Quién soy yo?
groups username        # Grupos de un usuario
finger username        # Información detallada (si está instalado)
```
## Ver todos los usuarios del sistema / View all system users
```bash

cat /etc/passwd        # Lista completa de usuarios
getent passwd          # Similar, usa la base de datos del sistema

# Formateado mejor:
cat /etc/passwd | cut -d: -f1  # Solo nombres de usuario
getent passwd | grep /bin/bash # Usuarios con shell bash
```
Estructura de /etc/passwd / Structure of /etc/passwd
```text

usuario:x:1000:1000:Nombre Completo:/home/usuario:/bin/bash
↑        ↑ ↑    ↑   ↑               ↑             ↑
│        │ │    │   │               │             └─ Shell por defecto
│        │ │    │   │               └─ Directorio home
│        │ │    │   └─ Información del usuario (GECOS)
│        │ │    └─ GID (ID del grupo primario)
│        │ └─ UID (ID del usuario)
│        └─ Contraseña (x = en /etc/shadow)
└─ Nombre de usuario
```
# 2. Crear usuarios / Create users
Crear usuario básico / Create basic user
```bash

# Debian/Ubuntu (más amigable)
sudo adduser nuevo_usuario      # Interactivo, crea home, pregunta datos

# RedHat/CentOS/Debian alternativo
sudo useradd nuevo_usuario      # Básico, sin home por defecto
sudo useradd -m nuevo_usuario   # Con directorio home (-m)
```
## Opciones comunes al crear usuarios / Common options when creating users
```bash

# Crear usuario con configuración específica
sudo useradd -m -s /bin/bash -c "Nombre Completo" -d /home/usuario usuario

# Opciones:
# -m, --create-home          Crea directorio home
# -s, --shell SHELL         Shell por defecto (/bin/bash, /bin/sh, /sbin/nologin)
# -c, --comment COMMENT     Descripción/comentario
# -d, --home-dir HOME_DIR   Directorio home personalizado
# -g, --gid GROUP           Grupo primario
# -G, --groups GROUPS       Grupos secundarios (separados por coma)
# -u, --uid UID             UID específico
```
## Crear usuario con configuración completa / Create user with full configuration
```bash

# Ejemplo práctico - Desarrollador
sudo useradd -m \
  -s /bin/bash \
  -c "Juan Pérez - Desarrollador" \
  -d /home/juan \
  -g developers \
  -G sudo,docker,www-data \
  -u 2001 \
  juan

# Verificar creación
sudo tail -1 /etc/passwd
ls -la /home/juan/

Establecer contraseña / Set password
bash

# Para tu propio usuario
passwd

# Para otro usuario (requiere sudo)
sudo passwd nombre_usuario

# Opciones de passwd
sudo passwd -l usuario    # Bloquear cuenta
sudo passwd -u usuario    # Desbloquear cuenta
sudo passwd -e usuario    # Expirar contraseña (forzar cambio)
sudo passwd -S usuario    # Estado de la contraseña
```
## Crear usuario sin shell de login / Create user without login shell
```bash

# Para servicios/daemons (no puede hacer login)
sudo useradd -r -s /usr/sbin/nologin servicio_usuario
sudo useradd -r -s /bin/false mysql_user

# -r, --system            Crea usuario de sistema (UID < 1000)
```
# 3. Modificar usuarios / Modify users
Cambiar propiedades de usuario / Change user properties
```bash

# Comando principal
sudo usermod [opciones] nombre_usuario

# Cambiar shell
sudo usermod -s /bin/bash usuario      # Cambiar a bash
sudo usermod -s /sbin/nologin usuario  # Deshabilitar login

# Cambiar directorio home
sudo usermod -d /nuevo/home -m usuario  # -m mueve los archivos

# Cambiar nombre de usuario
sudo usermod -l nuevo_nombre viejo_nombre

# Cambiar UID
sudo usermod -u 1500 usuario

# Bloquear/desbloquear cuenta
sudo usermod -L usuario    # Bloquear (Lock)
sudo usermod -U usuario    # Desbloquear (Unlock)

# Expirar cuenta
sudo usermod -e 2024-12-31 usuario  # Expira en fecha específica
sudo usermod -e "" usuario          # Quitar expiración
```
# Gestionar grupos de usuario / Manage user groups
```bash

# Agregar a grupos secundarios
sudo usermod -aG sudo,docker,developers usuario

# -a, --append          Agrega a los grupos existentes (IMPORTANTE: sin -a sobreescribe)
# -G, --groups          Lista de grupos secundarios

# Cambiar grupo primario
sudo usermod -g developers usuario

# Ver grupos actuales
groups usuario
id -Gn usuario
```
## Ejemplos prácticos de modificación / Practical modification examples
```bash

# 1. Dar permisos de sudo a un usuario
sudo usermod -aG sudo nombre_usuario

# 2. Agregar usuario al grupo de Docker
sudo usermod -aG docker nombre_usuario

# 3. Deshabilitar cuenta de usuario temporalmente
sudo usermod -s /sbin/nologin usuario
# o cambiar shell a /bin/false

# 4. Cambiar todo el nombre de un usuario
sudo usermod -c "Nuevo Nombre Completo" -d /home/nuevo -m -l nuevo viejo
```
## 4. Eliminar usuarios / Delete users
Eliminar usuario / Delete user
```bash

# Debian/Ubuntu (más seguro)
sudo deluser nombre_usuario           # Elimina usuario, pregunta por home

# RedHat/CentOS/Debian alternativo
sudo userdel nombre_usuario           # Elimina usuario, no elimina home

# Eliminar usuario y su directorio home
sudo userdel -r nombre_usuario        # -r, --remove

# Forzar eliminación (si el usuario está logeado)
sudo userdel -f -r nombre_usuario     # -f, --force
```
## Proceso seguro de eliminación / Safe deletion process
```bash

# 1. Verificar si el usuario existe
id nombre_usuario

# 2. Cerrar sesiones del usuario
sudo pkill -9 -u nombre_usuario       # Terminar procesos del usuario
sudo loginctl terminate-user nombre_usuario  # Cerrar sesión

# 3. Hacer backup del home (opcional pero recomendado)
sudo tar -czf /backup/usuario_backup.tar.gz /home/nombre_usuario

# 4. Eliminar usuario
sudo userdel -r nombre_usuario

# 5. Verificar eliminación
getent passwd nombre_usuario
ls -la /home/  # Verificar que se eliminó el directorio
```
## Eliminar solo ciertos archivos / Delete only certain files
```bash

# Si no quieres eliminar todo el home, eliminar manualmente:
sudo rm -rf /home/nombre_usuario
sudo userdel nombre_usuario  # Sin -r

# Eliminar usuario pero mantener home
sudo userdel nombre_usuario
# Luego renombrar el home si es necesario
sudo mv /home/nombre_usuario /home/nombre_usuario_old
```
## 5. Gestión de grupos / Group management
Ver grupos del sistema / View system groups
```bash

cat /etc/group         # Todos los grupos
getent group           # Similar, desde base de datos del sistema

# Grupos de un usuario específico
groups usuario
id -Gn usuario

# Ver miembros de un grupo
getent group nombre_grupo
grep nombre_grupo /etc/group

Estructura de /etc/group / Structure of /etc/group
```text

nombre_grupo:x:1001:usuario1,usuario2,usuario3
↑              ↑ ↑    ↑
│              │ │    └─ Lista de miembros (separados por coma)
│              │ └─ GID (ID del grupo)
│              └─ Contraseña (x = en /etc/gshadow)
└─ Nombre del grupo
```

## Crear grupos / Create groups
```bash

# Grupo normal
sudo groupadd nombre_grupo

# Con GID específico
sudo groupadd -g 2000 nombre_grupo

# Grupo de sistema
sudo groupadd -r sistema_grupo    # -r, --system (GID < 1000)

# Verificar creación
getent group nombre_grupo
```
## Modificar grupos / Modify groups
```bash

# Cambiar nombre de grupo
sudo groupmod -n nuevo_nombre viejo_nombre

# Cambiar GID
sudo groupmod -g 3000 nombre_grupo

# Agregar contraseña a grupo (para newgrp)
sudo gpasswd nombre_grupo
```
## Eliminar grupos / Delete groups
```bash

# Eliminar grupo (debe estar vacío)
sudo groupdel nombre_grupo

# Eliminar grupo forzadamente
sudo groupdel -f nombre_grupo    # -f, --force
```
## Gestionar miembros de grupos / Manage group members
```bash

# Agregar usuario a grupo
sudo gpasswd -a usuario nombre_grupo    # -a, --add

# Remover usuario de grupo
sudo gpasswd -d usuario nombre_grupo    # -d, --delete

# Establecer administradores del grupo
sudo gpasswd -A usuario1,usuario2 nombre_grupo

# Listar miembros del grupo
sudo gpasswd -M usuario1,usuario2 nombre_grupo  # -M, --members (reemplaza lista)
```
## 6. Permisos de usuarios / User permissions
Permisos de archivos y directorios / File and directory permissions
```bash

# Ver permisos actuales
ls -la
stat archivo.txt

# Cambiar dueño (chown)
sudo chown usuario:grupo archivo.txt
sudo chown usuario archivo.txt          # Solo usuario
sudo chown :grupo archivo.txt           # Solo grupo

# Recursivo (para directorios)
sudo chown -R usuario:grupo directorio/

# Cambiar grupo (chgrp)
sudo chgrp grupo archivo.txt
sudo chgrp -R grupo directorio/
```
## Permisos especiales / Special permissions
```bash

# SetUID - Ejecutar como propietario
sudo chmod u+s /usr/bin/programa      # s en permisos de usuario
sudo chmod 4755 /usr/bin/programa     # Numérico

# SetGID - Archivos heredan grupo del directorio
sudo chmod g+s /directorio_compartido/
sudo chmod 2775 /directorio_compartido/

# Sticky bit - Solo dueño puede borrar
sudo chmod +t /tmp
sudo chmod 1777 /tmp
```
## Máscara de permisos por defecto (umask) / Default permission mask
```bash

# Ver umask actual
umask
umask -S  # Formato simbólico

# Cambiar umask (solo para esta sesión)
umask 022  # Archivos: 644, Directorios: 755
umask 027  # Archivos: 640, Directorios: 750

# Para hacerlo permanente (agregar a ~/.bashrc)
echo "umask 022" >> ~/.bashrc
```
## ACLs - Permisos avanzados / Advanced permissions
```bash

# Ver ACLs actuales
getfacl archivo.txt

# Agregar permiso a usuario específico
setfacl -m u:usuario:rwx archivo.txt
setfacl -m g:grupo:rx directorio/

# Permisos por defecto para nuevos archivos
setfacl -d -m u:usuario:rwx directorio/👁️ 1. Ver usuarios del sistema / View system users
```
##Listar usuarios conectados / List logged in users
```bash

who                    # Usuarios conectados actualmente
w                      # Más detalles (qué están haciendo)
last                   # Historial de conexiones
lastlog                # Último login de cada usuario
```
##Ver información de usuario específico / View specific user info
```bash

id                     # Mi información de usuario
id username            # Información de otro usuario
whoami                 # ¿Quién soy yo?
groups username        # Grupos de un usuario
finger username        # Información detallada (si está instalado)
```
## Ver todos los usuarios del sistema / View all system users
```bash

cat /etc/passwd        # Lista completa de usuarios
getent passwd          # Similar, usa la base de datos del sistema

# Formateado mejor:
cat /etc/passwd | cut -d: -f1  # Solo nombres de usuario
getent passwd | grep /bin/bash # Usuarios con shell bash

Estructura de /etc/passwd / Structure of /etc/passwd
```text

usuario:x:1000:1000:Nombre Completo:/home/usuario:/bin/bash
↑        ↑ ↑    ↑   ↑               ↑             ↑
│        │ │    │   │               │             └─ Shell por defecto
│        │ │    │   │               └─ Directorio home
│        │ │    │   └─ Información del usuario (GECOS)
│        │ │    └─ GID (ID del grupo primario)
│        │ └─ UID (ID del usuario)
│        └─ Contraseña (x = en /etc/shadow)
└─ Nombre de usuario
```
## 2. Crear usuarios / Create users
Crear usuario básico / Create basic user
```bash

# Debian/Ubuntu (más amigable)
sudo adduser nuevo_usuario      # Interactivo, crea home, pregunta datos

# RedHat/CentOS/Debian alternativo
sudo useradd nuevo_usuario      # Básico, sin home por defecto
sudo useradd -m nuevo_usuario   # Con directorio home (-m)
```
## Opciones comunes al crear usuarios / Common options when creating users
```bash

# Crear usuario con configuración específica
sudo useradd -m -s /bin/bash -c "Nombre Completo" -d /home/usuario usuario

# Opciones:
# -m, --create-home          Crea directorio home
# -s, --shell SHELL         Shell por defecto (/bin/bash, /bin/sh, /sbin/nologin)
# -c, --comment COMMENT     Descripción/comentario
# -d, --home-dir HOME_DIR   Directorio home personalizado
# -g, --gid GROUP           Grupo primario
# -G, --groups GROUPS       Grupos secundarios (separados por coma)
# -u, --uid UID             UID específico
```
## Crear usuario con configuración completa / Create user with full configuration
```bash

# Ejemplo práctico - Desarrollador
sudo useradd -m \
  -s /bin/bash \
  -c "Juan Pérez - Desarrollador" \
  -d /home/juan \
  -g developers \
  -G sudo,docker,www-data \
  -u 2001 \
  juan

# Verificar creación
sudo tail -1 /etc/passwd
ls -la /home/juan/
```
##Establecer contraseña / Set password
```bash

# Para tu propio usuario
passwd

# Para otro usuario (requiere sudo)
sudo passwd nombre_usuario

# Opciones de passwd
sudo passwd -l usuario    # Bloquear cuenta
sudo passwd -u usuario    # Desbloquear cuenta
sudo passwd -e usuario    # Expirar contraseña (forzar cambio)
sudo passwd -S usuario    # Estado de la contraseña
```
## Crear usuario sin shell de login / Create user without login shell
```bash

# Para servicios/daemons (no puede hacer login)
sudo useradd -r -s /usr/sbin/nologin servicio_usuario
sudo useradd -r -s /bin/false mysql_user

# -r, --system            Crea usuario de sistema (UID < 1000)
```
## 3. Modificar usuarios / Modify users
Cambiar propiedades de usuario / Change user properties
```bash

# Comando principal
sudo usermod [opciones] nombre_usuario

# Cambiar shell
sudo usermod -s /bin/bash usuario      # Cambiar a bash
sudo usermod -s /sbin/nologin usuario  # Deshabilitar login

# Cambiar directorio home
sudo usermod -d /nuevo/home -m usuario  # -m mueve los archivos

# Cambiar nombre de usuario
sudo usermod -l nuevo_nombre viejo_nombre

# Cambiar UID
sudo usermod -u 1500 usuario

# Bloquear/desbloquear cuenta
sudo usermod -L usuario    # Bloquear (Lock)
sudo usermod -U usuario    # Desbloquear (Unlock)

# Expirar cuenta
sudo usermod -e 2024-12-31 usuario  # Expira en fecha específica
sudo usermod -e "" usuario          # Quitar expiración
```
## Gestionar grupos de usuario / Manage user groups
```bash

# Agregar a grupos secundarios
sudo usermod -aG sudo,docker,developers usuario

# -a, --append          Agrega a los grupos existentes (IMPORTANTE: sin -a sobreescribe)
# -G, --groups          Lista de grupos secundarios

# Cambiar grupo primario
sudo usermod -g developers usuario

# Ver grupos actuales
groups usuario
id -Gn usuario
```
## Ejemplos prácticos de modificación / Practical modification examples
```bash

# 1. Dar permisos de sudo a un usuario
sudo usermod -aG sudo nombre_usuario

# 2. Agregar usuario al grupo de Docker
sudo usermod -aG docker nombre_usuario

# 3. Deshabilitar cuenta de usuario temporalmente
sudo usermod -s /sbin/nologin usuario
# o cambiar shell a /bin/false

# 4. Cambiar todo el nombre de un usuario
sudo usermod -c "Nuevo Nombre Completo" -d /home/nuevo -m -l nuevo viejo
```
## 4. Eliminar usuarios / Delete users
Eliminar usuario / Delete user
```bash

# Debian/Ubuntu (más seguro)
sudo deluser nombre_usuario           # Elimina usuario, pregunta por home

# RedHat/CentOS/Debian alternativo
sudo userdel nombre_usuario           # Elimina usuario, no elimina home

# Eliminar usuario y su directorio home
sudo userdel -r nombre_usuario        # -r, --remove

# Forzar eliminación (si el usuario está logeado)
sudo userdel -f -r nombre_usuario     # -f, --force
```
## Proceso seguro de eliminación / Safe deletion process
```bash

# 1. Verificar si el usuario existe
id nombre_usuario

# 2. Cerrar sesiones del usuario
sudo pkill -9 -u nombre_usuario       # Terminar procesos del usuario
sudo loginctl terminate-user nombre_usuario  # Cerrar sesión

# 3. Hacer backup del home (opcional pero recomendado)
sudo tar -czf /backup/usuario_backup.tar.gz /home/nombre_usuario

# 4. Eliminar usuario
sudo userdel -r nombre_usuario

# 5. Verificar eliminación
getent passwd nombre_usuario
ls -la /home/  # Verificar que se eliminó el directorio
```
## Eliminar solo ciertos archivos / Delete only certain files
```bash

# Si no quieres eliminar todo el home, eliminar manualmente:
sudo rm -rf /home/nombre_usuario
sudo userdel nombre_usuario  # Sin -r

# Eliminar usuario pero mantener home
sudo userdel nombre_usuario
# Luego renombrar el home si es necesario
sudo mv /home/nombre_usuario /home/nombre_usuario_old

👥 5. Gestión de grupos / Group management
Ver grupos del sistema / View system groups
bash

cat /etc/group         # Todos los grupos
getent group           # Similar, desde base de datos del sistema

# Grupos de un usuario específico
groups usuario
id -Gn usuario

# Ver miembros de un grupo
getent group nombre_grupo
grep nombre_grupo /etc/group

Estructura de /etc/group / Structure of /etc/group
text

nombre_grupo:x:1001:usuario1,usuario2,usuario3
↑              ↑ ↑    ↑
│              │ │    └─ Lista de miembros (separados por coma)
│              │ └─ GID (ID del grupo)
│              └─ Contraseña (x = en /etc/gshadow)
└─ Nombre del grupo
```
## Crear grupos / Create groups
```bash

# Grupo normal
sudo groupadd nombre_grupo

# Con GID específico
sudo groupadd -g 2000 nombre_grupo

# Grupo de sistema
sudo groupadd -r sistema_grupo    # -r, --system (GID < 1000)

# Verificar creación
getent group nombre_grupo
```
## Modificar grupos / Modify groups
```bash

# Cambiar nombre de grupo
sudo groupmod -n nuevo_nombre viejo_nombre

# Cambiar GID
sudo groupmod -g 3000 nombre_grupo

# Agregar contraseña a grupo (para newgrp)
sudo gpasswd nombre_grupo
```
## Eliminar grupos / Delete groups
```bash

# Eliminar grupo (debe estar vacío)
sudo groupdel nombre_grupo

# Eliminar grupo forzadamente
sudo groupdel -f nombre_grupo    # -f, --force

Gestionar miembros de grupos / Manage group members
bash

# Agregar usuario a grupo
sudo gpasswd -a usuario nombre_grupo    # -a, --add

# Remover usuario de grupo
sudo gpasswd -d usuario nombre_grupo    # -d, --delete

# Establecer administradores del grupo
sudo gpasswd -A usuario1,usuario2 nombre_grupo

# Listar miembros del grupo
sudo gpasswd -M usuario1,usuario2 nombre_grupo  # -M, --members (reemplaza lista)
```
## 6. Permisos de usuarios / User permissions
Permisos de archivos y directorios / File and directory permissions
```bash

# Ver permisos actuales
ls -la
stat archivo.txt

# Cambiar dueño (chown)
sudo chown usuario:grupo archivo.txt
sudo chown usuario archivo.txt          # Solo usuario
sudo chown :grupo archivo.txt           # Solo grupo

# Recursivo (para directorios)
sudo chown -R usuario:grupo directorio/

# Cambiar grupo (chgrp)
sudo chgrp grupo archivo.txt
sudo chgrp -R grupo directorio/
```
## Permisos especiales / Special permissions
```bash

# SetUID - Ejecutar como propietario
sudo chmod u+s /usr/bin/programa      # s en permisos de usuario
sudo chmod 4755 /usr/bin/programa     # Numérico

# SetGID - Archivos heredan grupo del directorio
sudo chmod g+s /directorio_compartido/
sudo chmod 2775 /directorio_compartido/

# Sticky bit - Solo dueño puede borrar
sudo chmod +t /tmp
sudo chmod 1777 /tmp
```
## Máscara de permisos por defecto (umask) / Default permission mask
```bash

# Ver umask actual
umask
umask -S  # Formato simbólico

# Cambiar umask (solo para esta sesión)
umask 022  # Archivos: 644, Directorios: 755
umask 027  # Archivos: 640, Directorios: 750

# Para hacerlo permanente (agregar a ~/.bashrc)
echo "umask 022" >> ~/.bashrc
```
## ACLs - Permisos avanzados / Advanced permissions
```bash

# Ver ACLs actuales
getfacl archivo.txt

# Agregar permiso a usuario específico
setfacl -m u:usuario:rwx archivo.txt
setfacl -m g:grupo:rx directorio/

# Permisos por defecto para nuevos archivos
setfacl -d -m u:usuario:rwx directorio/

# Eliminar entrada específica
setfacl -x u:usuario archivo.txt

# Eliminar todas las ACLs
setfacl -b archivo.txt
```
## 7. Escenarios comunes / Common scenarios
Escenario 1: Nuevo desarrollador en el equipo
```bash

# 1. Crear usuario
sudo useradd -m -s /bin/bash -c "Carlos López - Dev" -d /home/carlos -g developers carlos

# 2. Establecer contraseña
sudo passwd carlos

# 3. Agregar a grupos necesarios
sudo usermod -aG sudo,docker,www-data carlos

# 4. Crear estructura de proyecto
sudo mkdir -p /projects/carlos
sudo chown -R carlos:developers /projects/carlos
sudo chmod -R 775 /projects/carlos

# 5. Configurar SSH keys
sudo mkdir /home/carlos/.ssh
sudo chmod 700 /home/carlos/.ssh
# Copiar clave pública...
sudo chown -R carlos:carlos /home/carlos/.ssh
```
## Escenario 2: Usuario de solo lectura para auditoría
```bash

# 1. Crear usuario sin shell de login
sudo useradd -r -s /sbin/nologin -c "Usuario Auditoría" auditor

# 2. Crear directorio de logs compartido
sudo mkdir -p /var/log/auditoria
sudo chown root:auditor /var/log/auditoria
sudo chmod 750 /var/log/auditoria

# 3. Dar acceso de solo lectura
sudo setfacl -m u:auditor:rx /var/log/auditoria

Escenario 3: Compartir carpeta entre varios usuarios
bash

# 1. Crear grupo para el proyecto
sudo groupadd proyecto_alpha

# 2. Crear directorio compartido
sudo mkdir -p /shared/proyecto_alpha
sudo chown root:proyecto_alpha /shared/proyecto_alpha
sudo chmod 2775 /shared/proyecto_alpha  # SetGID activado

# 3. Agregar usuarios al grupo
sudo usermod -aG proyecto_alpha usuario1
sudo usermod -aG proyecto_alpha usuario2
sudo usermod -aG proyecto_alpha usuario3

# 4. Configurar ACLs para permisos específicos
sudo setfacl -d -m g:proyecto_alpha:rwx /shared/proyecto_alpha
```
## Escenario 4: Limitar acceso de usuario
```bash

# 1. Cambiar shell a restrictiva
sudo usermod -s /bin/rbash usuario_restrictivo

# 2. Crear directorio home con estructura controlada
sudo mkdir -p /home/usuario_restrictivo/{bin,documents}
sudo chown -R usuario_restrictivo:usuario_restrictivo /home/usuario_restrictivo

# 3. Copiar solo comandos permitidos
sudo cp /bin/ls /home/usuario_restrictivo/bin/
sudo cp /bin/cat /home/usuario_restrictivo/bin/

# 4. Configurar PATH restringido
echo 'PATH=$HOME/bin' >> /home/usuario_restrictivo/.bashrc
```
## Escenario 5: Eliminar usuario que se fue de la empresa
```bash

# 1. Bloquear cuenta inmediatamente
sudo usermod -L usuario_saliente      # Bloquear login
sudo usermod -s /sbin/nologin usuario_saliente  # Cambiar shell

# 2. Terminar sesiones activas
sudo pkill -9 -u usuario_saliente

# 3. Backup de datos importantes
sudo tar -czf /backup/usuario_saliente_$(date +%Y%m%d).tar.gz /home/usuario_saliente

# 4. Transferir propiedad de archivos (si es necesario)
sudo chown -R nuevo_responsable:nuevo_responsable /home/usuario_saliente/proyectos/

# 5. Eliminar usuario después de 30 días (política de retención)
# sudo userdel -r usuario_saliente
```
## Escenario 6: Crear usuario de servicio (para aplicaciones)
```bash

# 1. Crear usuario de sistema sin login
sudo useradd -r -s /usr/sbin/nologin -d /var/lib/miapp miapp_user

# 2. Crear directorios necesarios
sudo mkdir -p /var/lib/miapp /var/log/miapp /etc/miapp
sudo chown -R miapp_user:miapp_user /var/lib/miapp /var/log/miapp
sudo chmod 755 /etc/miapp

# 3. Configurar permisos de logs
sudo chmod 750 /var/log/miapp
sudo setfacl -m g:adm:rx /var/log/miapp  # Grupo adm puede leer logs

⚠️ Buenas prácticas / Best practices
Seguridad / Security
bash

# 1. UIDs/GIDs consistentes
# Usuarios normales: UID >= 1000
# Usuarios sistema: UID < 1000

# 2. Contraseñas seguras
sudo passwd -e usuario  # Forzar cambio en próximo login
sudo chage -M 90 usuario  # Máximo 90 días

# 3. Monitoreo de actividad
last usuario
faillock --user usuario  # Intentos fallidos de login

# 4. Deshabilitar cuentas innecesarias
sudo usermod -s /sbin/nologin usuario_viejo
sudo passwd -l usuario_viejo

Organización / Organization
bash

# 1. Convención de nombres
# usuarios: nombre.apellido o userXX
# grupos: departamento-funcion (dev-backend, ops-monitoring)

# 2. Documentación
# Mantener /etc/group y /etc/passwd comentados
# Documentar en wiki qué grupos existen y para qué sirven

# 3. Scripts para consistencia
# Crear scripts para crear usuarios con configuración estándar

Mantenimiento / Maintenance
bash

# 1. Revisión periódica
# Listar usuarios sin login reciente:
lastlog -b 90  # No han logeado en 90 días

# 2. Limpieza de cuentas temporales
# Script para eliminar usuarios expirados

# 3. Backup de configuraciones
sudo tar -czf /backup/etc_passwd_group_$(date +%Y%m%d).tar.gz /etc/passwd /etc/group /etc/shadow /etc/gshadow
```
## Herramientas útiles / Useful tools
Comandos de verificación / Verification commands
```bash

# Verificar consistencia de usuarios
pwck    # Verifica /etc/passwd
grpck   # Verifica /etc/group

# Información de envejecimiento de contraseñas
chage -l usuario
chage -M 60 -m 7 -W 7 usuario  # Configurar política

# Buscar archivos de usuarios
find / -user usuario 2>/dev/null
find / -group grupo 2>/dev/null

Herramientas gráficas (opcionales) / Graphical tools (optional)
bash

# Ubuntu/Debian
sudo apt install gnome-system-tools  # users-admin

# RedHat/CentOS
sudo yum install system-config-users

# Terminal interactivo
sudo vigr  # Editar /etc/group
sudo vipw  # Editar /etc/passwd

# Eliminar entrada específica
setfacl -x u:usuario archivo.txt

# Eliminar todas las ACLs
setfacl -b archivo.txt
```
## 7. Escenarios comunes / Common scenarios
Escenario 1: Nuevo desarrollador en el equipo
```bash

# 1. Crear usuario
sudo useradd -m -s /bin/bash -c "Carlos López - Dev" -d /home/carlos -g developers carlos

# 2. Establecer contraseña
sudo passwd carlos

# 3. Agregar a grupos necesarios
sudo usermod -aG sudo,docker,www-data carlos

# 4. Crear estructura de proyecto
sudo mkdir -p /projects/carlos
sudo chown -R carlos:developers /projects/carlos
sudo chmod -R 775 /projects/carlos

# 5. Configurar SSH keys
sudo mkdir /home/carlos/.ssh
sudo chmod 700 /home/carlos/.ssh
# Copiar clave pública...
sudo chown -R carlos:carlos /home/carlos/.ssh
```
## Escenario 2: Usuario de solo lectura para auditoría
```bash

# 1. Crear usuario sin shell de login
sudo useradd -r -s /sbin/nologin -c "Usuario Auditoría" auditor

# 2. Crear directorio de logs compartido
sudo mkdir -p /var/log/auditoria
sudo chown root:auditor /var/log/auditoria
sudo chmod 750 /var/log/auditoria

# 3. Dar acceso de solo lectura
sudo setfacl -m u:auditor:rx /var/log/auditoria
```
## Escenario 3: Compartir carpeta entre varios usuarios
```bash

# 1. Crear grupo para el proyecto
sudo groupadd proyecto_alpha

# 2. Crear directorio compartido
sudo mkdir -p /shared/proyecto_alpha
sudo chown root:proyecto_alpha /shared/proyecto_alpha
sudo chmod 2775 /shared/proyecto_alpha  # SetGID activado

# 3. Agregar usuarios al grupo
sudo usermod -aG proyecto_alpha usuario1
sudo usermod -aG proyecto_alpha usuario2
sudo usermod -aG proyecto_alpha usuario3

# 4. Configurar ACLs para permisos específicos
sudo setfacl -d -m g:proyecto_alpha:rwx /shared/proyecto_alpha
```
## Escenario 4: Limitar acceso de usuario
```bash

# 1. Cambiar shell a restrictiva
sudo usermod -s /bin/rbash usuario_restrictivo

# 2. Crear directorio home con estructura controlada
sudo mkdir -p /home/usuario_restrictivo/{bin,documents}
sudo chown -R usuario_restrictivo:usuario_restrictivo /home/usuario_restrictivo

# 3. Copiar solo comandos permitidos
sudo cp /bin/ls /home/usuario_restrictivo/bin/
sudo cp /bin/cat /home/usuario_restrictivo/bin/

# 4. Configurar PATH restringido
echo 'PATH=$HOME/bin' >> /home/usuario_restrictivo/.bashrc
```
## Escenario 5: Eliminar usuario que se fue de la empresa
```bash

# 1. Bloquear cuenta inmediatamente
sudo usermod -L usuario_saliente      # Bloquear login
sudo usermod -s /sbin/nologin usuario_saliente  # Cambiar shell

# 2. Terminar sesiones activas
sudo pkill -9 -u usuario_saliente

# 3. Backup de datos importantes
sudo tar -czf /backup/usuario_saliente_$(date +%Y%m%d).tar.gz /home/usuario_saliente

# 4. Transferir propiedad de archivos (si es necesario)
sudo chown -R nuevo_responsable:nuevo_responsable /home/usuario_saliente/proyectos/

# 5. Eliminar usuario después de 30 días (política de retención)
# sudo userdel -r usuario_saliente
```
## Escenario 6: Crear usuario de servicio (para aplicaciones)
```bash

# 1. Crear usuario de sistema sin login
sudo useradd -r -s /usr/sbin/nologin -d /var/lib/miapp miapp_user

# 2. Crear directorios necesarios
sudo mkdir -p /var/lib/miapp /var/log/miapp /etc/miapp
sudo chown -R miapp_user:miapp_user /var/lib/miapp /var/log/miapp
sudo chmod 755 /etc/miapp

# 3. Configurar permisos de logs
sudo chmod 750 /var/log/miapp
sudo setfacl -m g:adm:rx /var/log/miapp  # Grupo adm puede leer logs
```
## Buenas prácticas / Best practices
Seguridad / Security
```bash

# 1. UIDs/GIDs consistentes
# Usuarios normales: UID >= 1000
# Usuarios sistema: UID < 1000

# 2. Contraseñas seguras
sudo passwd -e usuario  # Forzar cambio en próximo login
sudo chage -M 90 usuario  # Máximo 90 días

# 3. Monitoreo de actividad
last usuario
faillock --user usuario  # Intentos fallidos de login

# 4. Deshabilitar cuentas innecesarias
sudo usermod -s /sbin/nologin usuario_viejo
sudo passwd -l usuario_viejo
```
##Organización / Organization
```bash

# 1. Convención de nombres
# usuarios: nombre.apellido o userXX
# grupos: departamento-funcion (dev-backend, ops-monitoring)

# 2. Documentación
# Mantener /etc/group y /etc/passwd comentados
# Documentar en wiki qué grupos existen y para qué sirven

# 3. Scripts para consistencia
# Crear scripts para crear usuarios con configuración estándar
```
## Mantenimiento / Maintenance
```bash

# 1. Revisión periódica
# Listar usuarios sin login reciente:
lastlog -b 90  # No han logeado en 90 días

# 2. Limpieza de cuentas temporales
# Script para eliminar usuarios expirados

# 3. Backup de configuraciones
sudo tar -czf /backup/etc_passwd_group_$(date +%Y%m%d).tar.gz /etc/passwd /etc/group /etc/shadow /etc/gshadow
```
## Herramientas útiles / Useful tools
Comandos de verificación / Verification commands
```bash

# Verificar consistencia de usuarios
pwck    # Verifica /etc/passwd
grpck   # Verifica /etc/group

# Información de envejecimiento de contraseñas
chage -l usuario
chage -M 60 -m 7 -W 7 usuario  # Configurar política

# Buscar archivos de usuarios
find / -user usuario 2>/dev/null
find / -group grupo 2>/dev/null
```
## Herramientas gráficas (opcionales) / Graphical tools (optional)
```bash

# Ubuntu/Debian
sudo apt install gnome-system-tools  # users-admin

# RedHat/CentOS
sudo yum install system-config-users

# Terminal interactivo
sudo vigr  # Editar /etc/group
sudo vipw  # Editar /etc/passwd
```
