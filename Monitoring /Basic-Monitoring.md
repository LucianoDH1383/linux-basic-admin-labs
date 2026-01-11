## Basic Monitoring for SysAdmin Jr / Monitoreo Básico para SysAdmin Jr
## Table of Contents / Índice
```
Why Monitoring Matters / Por qué es Importante el Monitoreo

What to Monitor First / Qué Monitorear Primero

Built-in Linux Tools / Herramientas Nativas de Linux

Monitoring Commands Explained / Comandos de Monitoreo Explicados

Alerting Basics / Conceptos Básicos de Alertas

Practice Exercises / Ejercicios Prácticos
```
## 1. Why Monitoring Matters / Por qué es Importante el Monitoreo

The Monitoring Mindset / La Mentalidad del Monitoreo

```text
❌ REACTIVE: "The server is down!" → "Let me check..."
✅ PROACTIVE: "Disk at 85%, might fail soon" → "Let me clean it before it breaks"
What Monitoring Gives You / Lo que el Monitoreo te Da:
Visibility / Visibilidad: Sabes qué está pasando

Alerting / Alertas: Te avisa antes de que falle

Trends / Tendencias: Ves patrones (ej: "Cada lunes a las 9 AM hay pico de CPU")

Data for Decisions / Datos para Decisiones: "¿Necesitamos más RAM? Veamos los gráficos"
```
## 2. What to Monitor First / Qué Monitorear Primero
# Priority 1: Survival Metrics / Métricas de Supervivencia
```bash
# 1. DISK SPACE / ESPACIO EN DISCO
# Why / Por qué: Disco lleno = TODO se detiene
# Command / Comando: df -h
# Threshold / Umbral: >80% = Warning, >90% = Critical

# 2. MEMORY / MEMORIA
# Why / Por qué: Sin memoria = procesos mueren
# Command / Comando: free -h
# Threshold / Umbral: >80% = Warning, >90% = Critical

# 3. CPU / CPU
# Why / Por qué: CPU al 100% = sistema lento
# Command / Comando: top, mpstat
# Threshold / Umbral: >70% por 5 minutos = Warning
Priority 2: Service Health / Salud de Servicios
bash
# 4. SERVICE STATUS / ESTADO DE SERVICIOS
# Why / Por qué: Servicio caído = usuarios no pueden trabajar
# Command / Comando: systemctl status
# Check / Verificar: sshd, nginx, mysql, postgresql

# 5. NETWORK / RED
# Why / Por qué: Sin red = servidor aislado
# Command / Comando: ping, ss -tulpn
# Check / Verificar: Conexión a internet, puertos abiertos

# 6. LOGS / REGISTROS
# Why / Por qué: Errores = problemas futuros
# Command / Comando: journalctl, grep error
# Check / Verificar: Errores, intentos fallidos de login
```
# Priority 3: Performance / Rendimiento
```bash
# 7. RESPONSE TIME / TIEMPO DE RESPUESTA
# Why / Por qué: Lento = usuarios frustrados
# Command / Comando: time curl, ab (Apache Benchmark)

# 8. USER COUNT / CANTIDAD DE USUARIOS
# Why / Por qué: Saber cuánta carga soporta
# Command / Comando: who, w
```
## 3. Built-in Linux Tools / Herramientas Nativas de Linux
Real-time Monitoring / Monitoreo en Tiempo Real
```bash
# 1. top - Basic process monitor / Monitor básico de procesos
top
# Keys / Teclas:
# '1' = Show all CPUs / Mostrar todas las CPUs
# 'M' = Sort by memory / Ordenar por memoria
# 'P' = Sort by CPU / Ordenar por CPU
# 'q' = Quit / Salir

# 2. htop - Better version of top / Versión mejorada de top
htop
```
# Install / Instalar:
```
sudo apt install htop    # Debian/Ubuntu
sudo yum install htop    # RHEL/CentOS
```
# 3. glances - All-in-one monitor / Monitor todo-en-uno
```
glances
```
# Install / Instalar:
```
sudo apt install glances
Disk Monitoring / Monitoreo de Disco
```
# 1. df - Disk free / Espacio libre en disco
```
df -h                    # Human readable / Legible para humanos
df -i                    # Inode usage / Uso de inodos
df -Th                   # Show filesystem type / Mostrar tipo de filesystem
```
# Options / Opciones:
```
# -h = Human readable (KB, MB, GB)
# -i = Show inodes instead of space
# -T = Show filesystem type
```
# 2. du - Disk usage / Uso de disco
```
du -sh /var/log         # Size of directory / Tamaño del directorio
du -ah /home | sort -rh | head -20  # Top 20 largest / Top 20 más grandes
```
# Options / Opciones:
```
# -s = Summary (solo total)
# -h = Human readable
# -a = All files (incluye archivos individuales)
```
# 3. iostat - Disk I/O statistics / Estadísticas de I/O de disco
```
iostat -x 1             # Update every second / Actualizar cada segundo
# Install / Instalar: sudo apt install sysstat
```
## Memory Monitoring / Monitoreo de Memoria
```bash
# 1. free - Memory usage / Uso de memoria
free -h                 # Human readable / Legible para humanos
free -m                 # In megabytes / En megabytes

# Understanding / Entendiendo:
# total = Total RAM
# used = RAM en uso
# free = RAM libre
# buff/cache = RAM usada para caché (se puede liberar)
# available = RAM realmente disponible para programas nuevos

# 2. vmstat - Virtual memory stats / Estadísticas de memoria virtual
vmstat 1 5              # 5 reports, 1 second apart / 5 reportes, cada 1 segundo

# Important columns / Columnas importantes:
# r = Processes waiting for CPU
# b = Processes sleeping
# swpd = Virtual memory used
# free = Free memory
# si = Memory swapped in from disk
# so = Memory swapped out to disk

# 3. /proc/meminfo - Detailed memory info / Info detallada de memoria
cat /proc/meminfo       # Raw memory data / Datos crudos de memoria
Network Monitoring / Monitoreo de Red
bash
# 1. ss / netstat - Network connections / Conexiones de red
ss -tulpn               # All listening ports / Todos los puertos escuchando
# Options / Opciones:
# -t = TCP connections
# -u = UDP connections
# -l = Listening ports
# -p = Show process
# -n = Don't resolve names (más rápido)

# 2. iftop - Bandwidth usage / Uso de ancho de banda
sudo iftop              # Show real-time bandwidth / Mostrar ancho de banda en tiempo real
# Install / Instalar: sudo apt install iftop

# 3. nload - Network load / Carga de red
nload                   # Simple bandwidth monitor / Monitor simple de ancho de banda
# Install / Instalar: sudo apt install nload
```
Log Monitoring / Monitoreo de Logs
```bash
# 1. tail - Follow logs / Seguir logs
sudo tail -f /var/log/syslog          # Follow system log / Seguir log del sistema
sudo tail -f /var/log/nginx/access.log # Follow web server / Seguir servidor web

# Options / Opciones:
# -f = Follow (continuously show new lines)
# -n 50 = Show last 50 lines
# -F = Follow, retry if file rotates

# 2. journalctl - Systemd journal / Journal de Systemd
sudo journalctl -f                    # Follow / Seguir
sudo journalctl --since "1 hour ago"  # Last hour / Última hora
sudo journalctl -p err                # Only errors / Solo errores
sudo journalctl -u nginx              # Only nginx service / Solo servicio nginx

# 3. grep for errors / Buscar errores con grep
sudo grep -i error /var/log/syslog                    # Find errors / Encontrar errores
sudo grep "Failed password" /var/log/auth.log         # Failed logins / Logins fallidos
sudo grep -c "error" /var/log/syslog                  # Count errors / Contar errores
```
 ## 4. Monitoring Commands Explained / Comandos de Monitoreo Explicados
Understanding top Output / Entendiendo la Salida de top
```text
top - 14:30:30 up 10 days,  3:15,  1 user,  load average: 0.05, 0.10, 0.15
↑          ↑          ↑         ↑        ↑              ↑
│          │          │         │        │              └─ Load average (1, 5, 15 min)
│          │          │         │        └─ Users logged in
│          │          │         └─ System uptime
│          │          └─ Time
│          └─ Command name
└─ Always first line

Tasks: 120 total,   1 running, 119 sleeping,   0 stopped,   0 zombie
↑                     ↑           ↑             ↑            ↑
│                     │           │             │            └─ Zombie processes
│                     │           │             └─ Stopped processes
│                     │           └─ Sleeping processes
│                     └─ Running processes
└─ Process summary

%Cpu(s):  2.3 us,  1.0 sy,  0.0 ni, 96.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
↑          ↑        ↑       ↑       ↑       ↑       ↑       ↑       ↑
│          │        │       │       │       │       │       │       └─ Stolen (virtualization)
│          │        │       │       │       │       │       └─ Software interrupts
│          │        │       │       │       │       └─ Hardware interrupts
│          │        │       │       │       └─ I/O wait
│          │        │       │       └─ Idle
│          │        │       └─ Nice (low priority)
│          │        └─ System (kernel)
│          └─ User (programs)
└─ CPU usage breakdown
Understanding df Output / Entendiendo la Salida de df
text
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   40G   10G  80% /
↑                ↑     ↑     ↑    ↑    ↑
│                │     │     │    │    └─ Where it's mounted
│                │     │     │    └─ Percentage used
│                │     │     └─ Available space
│                │     └─ Used space
│                └─ Total size
└─ Disk partition
Understanding free Output / Entendiendo la Salida de free
text
              total        used        free      shared  buff/cache   available
Mem:           7.7G        2.1G        1.2G        123M        4.4G        5.2G
Swap:          2.0G          0B        2.0G
Key Points / Puntos Clave:

buff/cache is memory used for caching (can be freed if needed)

available is what's really free for new applications

Swap is disk space used as "extra RAM" (slow)
```
## 5. Alerting Basics / Conceptos Básicos de Alertas
Simple Manual Checks / Chequeos Manuales Simples
```bash
# Check disk space and warn / Verificar espacio y advertir
DISK_USAGE=$(df / | awk 'END{print $5}' | sed 's/%//')
if [ $DISK_USAGE -gt 80 ]; then
    echo "⚠️  WARNING: Disk is ${DISK_USAGE}% full!"
fi

# Check memory and warn / Verificar memoria y advertir
MEM_USAGE=$(free | awk '/Mem:/ {printf("%.0f"), $3/$2*100}')
if [ $MEM_USAGE -gt 80 ]; then
    echo "⚠️  WARNING: Memory is ${MEM_USAGE}% used!"
fi

# Check if service is running / Verificar si servicio está corriendo
if ! systemctl is-active --quiet nginx; then
    echo "❌ ALERT: nginx is not running!"
fi
```
# What to Alert On / Sobre Qué Alertar
```bash
# CRITICAL (Wake me up at 3 AM) / (Despiértame a las 3 AM)
- Disk > 95%
- Service down for > 5 minutes
- Can't ping gateway
- High error rate in logs

# WARNING (Check tomorrow) / (Revisar mañana)
- Disk > 80%
- Memory > 85%
- CPU > 90% for 10 minutes
- Backup failed

# INFO (Just for records) / (Solo para registros)
- User logged in/out
- Package updated
- Configuration changed
```
# Alert Channels / Canales de Alerta
```bash
# 1. Email - Simple but might be ignored
# 2. SMS - Good for critical alerts
# 3. Slack/Teams - Good for team alerts
# 4. PagerDuty/Opsgenie - Professional on-call
# 5. Dashboard - Always visible
```
## 6. Practice Exercises / Ejercicios Prácticos
Exercise 1: Daily Health Check / Ejercicio 1: Chequeo Diario de Salud
```bash
# Task / Tarea: Create a daily monitoring routine
# Time / Tiempo: 5 minutes every morning

# Steps / Pasos:
1. Check disk space: df -h | grep -E "^/dev|Use%"
2. Check memory: free -h
3. Check CPU load: uptime
4. Check critical services: systemctl status nginx mysql sshd
5. Check for errors: sudo tail -20 /var/log/syslog | grep -i error

# Record in a log / Registrar en un log:
echo "$(date): Disk $(df / | awk 'END{print $5}'), Mem $(free | awk '/Mem:/ {printf("%.0f%%"), $3/$2*100}')" >> /var/log/daily-check.log
```
# Exercise 2: Find the Problem / Ejercicio 2: Encontrar el Problema
```bash
# Scenario / Escenario: Users report "server is slow"

# Investigation steps / Pasos de investigación:
1. Check load average: uptime
2. Check top processes: top -b -n 1 | head -20
3. Check memory: free -h
4. Check disk I/O: iostat -x 1 3
5. Check network: ss -s
6. Check logs: sudo journalctl --since "10 minutes ago" -p err

# Expected output / Salida esperada:
# You should be able to identify:
# - Is it CPU, memory, disk, or network?
# - Which process is causing it?
# - When did it start?
```
## Exercise 3: Create Monitoring Dashboard / Ejercicio 3: Crear Dashboard de Monitoreo
```bash
# Create a simple text-based dashboard
echo "========================================"
echo "        SYSTEM DASHBOARD"
echo "========================================"
echo "Time: $(date)"
echo "Uptime: $(uptime -p)"
echo ""
echo "💿 DISK: $(df -h / | tail -1 | awk '{print $5}') used"
echo "💾 MEMORY: $(free -h | grep Mem | awk '{print $3"/"$2}')"
echo "⚡ LOAD: $(cat /proc/loadavg | awk '{print $1,$2,$3}')"
echo ""
echo "🛠️ SERVICES:"
systemctl is-active nginx && echo "  nginx: ✅" || echo "  nginx: ❌"
systemctl is-active sshd && echo "  sshd: ✅" || echo "  sshd: ❌"
echo "========================================"
Exercise 4: Log Analysis / Ejercicio 4: Análisis de Logs
bash
# Analyze /var/log/auth.log for security
echo "🔒 SECURITY REPORT - $(date)"
echo "============================="
echo "Failed logins: $(sudo grep "Failed password" /var/log/auth.log | wc -l)"
echo "Successful logins: $(sudo grep "Accepted password" /var/log/auth.log | wc -l)"
echo "Users currently logged in: $(who | wc -l)"
echo ""

# Show suspicious activity
echo "Suspicious activity (last hour):"
sudo grep -i "fail\|invalid\|refused" /var/log/auth.log --since="1 hour ago"
```
Exercise 5: Trend Analysis / Ejercicio 5: Análisis de Tendencias
```bash
# Track disk usage over time
echo "Date, Disk Usage (%)" > disk-trend.csv
echo "$(date), $(df / | awk 'END{print $5}' | sed 's/%//')" >> disk-trend.csv

# After a week, check growth
echo "Disk usage growth per day:"
awk -F, '{print $2}' disk-trend.csv | tail -7
```
