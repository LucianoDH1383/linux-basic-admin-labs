
## 🐛 Linux Troubleshooting Guide / Guía de Resolución de Problemas

## 📋 **Table of Contents / Índice**
1. [Troubleshooting Methodology](#methodology)
2. [Common Problems & Solutions](#common-problems)
3. [Diagnostic Tools](#tools)
4. [Troubleshooting Scripts](#scripts)
5. [Incident Documentation](#incidents)

---

## 🔍 **1. Troubleshooting Methodology / Metodología**

### **The 5 Whys Technique / Técnica de los 5 Porqués**
Problem: Website is slow

Why? → Database queries are slow

Why? → No indexes on large tables

Why? → Indexes weren't maintained

Why? → No monitoring on query performance

Why? → Lack of database maintenance procedures

text

### **Systematic Approach / Enfoque Sistemático**
```bash
# Step 1: Identify the scope / Paso 1: Identificar el alcance
who is affected?              # ¿Quién está afectado?
what exactly is broken?       # ¿Qué exactamente está roto?
when did it start?           # ¿Cuándo empezó?
where is it happening?       # ¿Dónde está pasando?

# Step 2: Gather data / Paso 2: Recolectar datos
collect_logs() {
    journalctl --since "1 hour ago"
    dmesg | tail -50
    systemctl list-units --failed
}

# Step 3: Hypothesize & test / Paso 3: Hipótesis y prueba
test_one_thing_at_a_time     # Probar una cosa a la vez
document_every_change        # Documentar cada cambio
```
# 2. Common Problems & Solutions / Problemas Comunes y Soluciones
Problem: "Permission denied" / Problema: "Permiso denegado"
```bash
# Symptoms / Síntomas:
# - Cannot run script / No puedo ejecutar script
# - Cannot access file / No puedo acceder a archivo
# - Operation not permitted / Operación no permitida
```
# Diagnosis / Diagnóstico:
```
ls -l /path/to/file          # Check permissions / Verificar permisos
id                           # Check user/group / Verificar usuario/grupo
namei -l /path/to/file      # Check all permissions in path / Verificar todos los permisos en la ruta
```
# Solutions / Soluciones:
```
sudo chmod +x script.sh     # Add execute permission / Agregar permiso de ejecución
sudo chown user:group file  # Change ownership / Cambiar propiedad
sudo setfacl -m u:user:rwx file  # Add ACL permission / Agregar permiso ACL
Problem: "No space left on device" / Problema: "No hay espacio en disco"
```bash
# Symptoms / Síntomas:
# - Cannot write to disk / No se puede escribir en disco
# - Database errors / Errores de base de datos
# - Service crashes / Servicios se caen
```
# Diagnosis / Diagnóstico:
```
df -h                        # Check disk usage / Verificar uso de disco
du -sh /* 2>/dev/null | sort -hr | head -20  # Find large directories / Encontrar directorios grandes
lsof +L1                    # Find deleted but open files / Encontrar archivos borrados pero abiertos
```
# Solutions / Soluciones:
# Clear old logs / Limpiar logs viejos
```
sudo journalctl --vacuum-time=7d
sudo find /var/log -name "*.log" -mtime +30 -delete
```
# Clear package cache / Limpiar caché de paquetes
```
sudo apt clean              # Debian/Ubuntu
sudo yum clean all         # RHEL/CentOS
```
# Find and delete large files / Encontrar y borrar archivos grandes
```
sudo find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null
Problem: "Service not starting" / Problema: "Servicio no inicia"
```
```bash
# Symptoms / Síntomas:
# - systemctl start fails / systemctl start falla
# - Port not listening / Puerto no escuchando
# - Service crashes immediately / Servicio se cae inmediatamente
```
# Diagnosis / Diagnóstico:
```
sudo systemctl status service_name    # Check service status / Verificar estado del servicio
sudo journalctl -u service_name -f    # Follow service logs / Seguir logs del servicio
sudo systemctl cat service_name       # View service file / Ver archivo del servicio
ss -tulpn | grep :port               # Check if port is listening / Verificar si puerto está escuchando
```
# Solutions / Soluciones:
# Check dependencies / Verificar dependencias
```
sudo systemctl list-dependencies service_name
```
# Check syntax / Verificar sintaxis
```
sudo nginx -t                        # For nginx / Para nginx
sudo apache2ctl configtest          # For Apache / Para Apache
```
# Test manually / Probar manualmente
```
sudo -u service_user /path/to/binary --config /path/to/config
Problem: "Network connectivity issues" / Problema: "Problemas de conectividad de red"
```
```bash
# Symptoms / Síntomas:
# - Cannot ping / No se puede hacer ping
# - Cannot resolve DNS / No se puede resolver DNS
# - Port not reachable / Puerto no alcanzable
```
# Diagnosis / Diagnóstico:
```
ping 8.8.8.8                         # Test basic connectivity / Probar conectividad básica
ping google.com                      # Test DNS resolution / Probar resolución DNS
traceroute google.com                # Trace network path / Rastrear ruta de red
curl -I https://google.com           # Test HTTP connectivity / Probar conectividad HTTP
nslookup google.com                  # Test DNS manually / Probar DNS manualmente
```
# Solutions / Soluciones:
# Check network config / Verificar configuración de red
```
ip addr show
ip route show
cat /etc/resolv.conf
```
# Check firewall / Verificar firewall
```
sudo iptables -L -n -v
sudo ufw status
```
# Restart network / Reiniciar red
```
sudo systemctl restart NetworkManager   # NetworkManager
sudo systemctl restart networking      # systemd-networkd
Problem: "High CPU usage" / Problema: "Alto uso de CPU"
```
```bash
# Symptoms / Síntomas:
# - System slow / Sistema lento
# - High load average / Carga promedio alta
# - Processes using 100% CPU / Procesos usando 100% CPU
```
# Diagnosis / Diagnóstico:
```
top                                 # Interactive process view / Vista interactiva de procesos
htop                                # Better interactive view / Mejor vista interactiva
ps aux --sort=-%cpu | head -10     # Top CPU processes / Top procesos de CPU
mpstat -P ALL 1 5                  # CPU statistics / Estadísticas de CPU
```
# Find CPU-hungry processes / Encontrar procesos que consumen CPU
```
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head -20
```
# Solutions / Soluciones:
# Kill problematic process / Matar proceso problemático
```
sudo kill -9 PID                   # Force kill / Forzar terminación
```
# Limit process CPU / Limitar CPU de proceso
```
sudo cpulimit -p PID -l 50        # Limit to 50% / Limitar al 50%
```
# Check for malware / Verificar malware
```
sudo chkrootkit
sudo rkhunter --check
Problem: "High memory usage" / Problema: "Alto uso de memoria"
```
```bash
# Symptoms / Síntemas:
# - System slow / Sistema lento
# - Swap being used / Swap siendo usado
# - Out of memory errors / Errores de memoria agotada
```
# Diagnosis / Diagnóstico:
```
free -h                            # Memory usage / Uso de memoria
vmstat 1 5                        # Virtual memory stats / Estadísticas de memoria virtual
ps aux --sort=-%mem | head -10    # Top memory processes / Top procesos de memoria
cat /proc/meminfo                 # Detailed memory info / Información detallada de memoria
```
# Check for memory leaks / Verificar fugas de memoria
```
valgrind --leak-check=yes /path/to/program
```
# Solutions / Soluciones:
# Clear page cache / Limpiar caché de página
```
sudo sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```
# Restart heavy services / Reiniciar servicios pesados
```
sudo systemctl restart memory_hungry_service
```
# Adjust swappiness / Ajustar swappiness
```
sudo sysctl vm.swappiness=10
Problem: "Cannot connect to database" / Problema: "No se puede conectar a base de datos"
```
```bash
# Symptoms / Síntomas:
# - Connection refused / Conexión rechazada
# - Authentication failed / Autenticación fallida
# - Too many connections / Demasiadas conexiones
```
# Diagnosis / Diagnóstico:
# MySQL/MariaDB
```
sudo systemctl status mysql
sudo tail -f /var/log/mysql/error.log
mysqladmin ping                   # Test connection / Probar conexión
```
# PostgreSQL
```
sudo systemctl status postgresql
sudo tail -f /var/log/postgresql/postgresql-*.log
pg_isready                       # Test connection / Probar conexión
```
# Check connections / Verificar conexiones
```
mysql -e "SHOW PROCESSLIST;"     # MySQL
psql -c "SELECT * FROM pg_stat_activity;"  # PostgreSQL
```
# Solutions / Soluciones:
# Restart database service / Reiniciar servicio de base de datos
```
sudo systemctl restart mysql
```
# Increase connections / Aumentar conexiones
# Edit /etc/mysql/mysql.conf.d/mysqld.cnf and set:
# max_connections = 500

# Kill idle connections / Matar conexiones inactivas
```
mysql -e "SHOW PROCESSLIST;" | grep "Sleep" | awk '{print $1}' | xargs -I {} mysql -e "KILL {};"
🛠️ 3. Diagnostic Tools / Herramientas de Diagnóstico
System Monitoring Tools / Herramientas de Monitoreo
```

# Basic monitoring / Monitoreo básico
```
top        # Process monitor / Monitor de procesos
htop       # Better top / Top mejorado
iotop      # Disk I/O monitor / Monitor de I/O de disco
nethogs    # Network traffic by process / Tráfico de red por proceso
```
# Disk monitoring / Monitoreo de disco
```
iostat -x 1  # Disk I/O statistics / Estadísticas de I/O de disco
df -i       # Inode usage / Uso de inodos
duf        # Better df / df mejorado (needs install / necesita instalación)
```
# Network monitoring / Monitoreo de red
```
iftop      # Network bandwidth / Ancho de banda de red
nload      # Network load / Carga de red
bmon       # Bandwidth monitor / Monitor de ancho de banda
```
# Log monitoring / Monitoreo de logs
```
lnav       # Log file navigator / Navegador de archivos de log
multitail  # Multiple tail / Tail múltiple
Network Troubleshooting Tools / Herramientas de Red
```
# Connectivity / Conectividad
```
ping        # Test reachability / Probar alcanzabilidad
traceroute  # Trace network path / Rastrear ruta de red
mtr         # Better traceroute / Traceroute mejorado
pathping    # Windows traceroute+ping / Traceroute+ping de Windows
```
# Port scanning / Escaneo de puertos
```
nmap        # Network mapper / Mapeador de red
netcat      # Swiss army knife for TCP/UDP / Navaja suiza para TCP/UDP
telnet      # Test TCP connections / Probar conexiones TCP
```
# DNS troubleshooting / Resolución de DNS
```
dig         # DNS lookup / Consulta DNS
nslookup    # DNS query / Consulta DNS
host        # Simple DNS lookup / Consulta DNS simple
```
# Packet analysis / Análisis de paquetes
```
tcpdump     # Packet capture / Captura de paquetes
tshark      # Wireshark CLI / Wireshark en línea de comandos
ngrep       # Network grep / Grep de red
```
## Performance Analysis Tools / Herramientas de Análisis de Rendimiento
```
# System performance / Rendimiento del sistema
sar         # System activity reporter / Reporteador de actividad del sistema
vmstat      # Virtual memory stats / Estadísticas de memoria virtual
mpstat      # CPU stats / Estadísticas de CPU
pidstat     # Process stats / Estadísticas de proceso
```
# I/O performance / Rendimiento de I/O
```
iostat      # I/O statistics / Estadísticas de I/O
iotop       # I/O by process / I/O por proceso
blktrace    # Block layer I/O tracing / Rastreo de I/O de capa de bloque
```
# Memory analysis / Análisis de memoria
```
smem        # Memory usage reporting / Reporte de uso de memoria
valgrind    # Memory debugger / Depurador de memoria
pmap        # Process memory map / Mapa de memoria de proceso
```
## Security Troubleshooting Tools / Herramientas de Seguridad
```bash
# File integrity / Integridad de archivos
aide        # Advanced Intrusion Detection Environment
tripwire    # File integrity checker / Verificador de integridad de archivos

# Rootkit detection / Detección de rootkits
chkrootkit  # Rootkit checker / Verificador de rootkits
rkhunter    # Rootkit hunter / Cazador de rootkits

# Log analysis / Análisis de logs
logwatch    # Log analyzer / Analizador de logs
fail2ban    # Ban IPs with failed attempts / Banear IPs con intentos fallidos

# Network security / Seguridad de red
nmap        # Port scanner / Escáner de puertos
netstat     # Network statistics / Estadísticas de red
ss          # Socket statistics / Estadísticas de sockets
📜 4. Troubleshooting Scripts / Scripts de Troubleshooting
Quick System Check Script / Script de Chequeo Rápido
bash
#!/bin/bash
# quick_check.sh - Quick system health check

echo "=== SYSTEM HEALTH CHECK ==="
echo "Timestamp: $(date)"
echo ""

echo "=== UPTIME AND LOAD ==="
uptime
echo ""

echo "=== MEMORY USAGE ==="
free -h
echo ""

echo "=== DISK USAGE ==="
df -h
echo ""

echo "=== TOP PROCESSES (CPU) ==="
ps aux --sort=-%cpu | head -5
echo ""

echo "=== TOP PROCESSES (MEMORY) ==="
ps aux --sort=-%mem | head -5
echo ""

echo "=== FAILED SERVICES ==="
systemctl list-units --failed
echo ""

echo "=== NETWORK CONNECTIONS ==="
ss -tulpn | head -10
echo ""
Log Analysis Script / Script de Análisis de Logs
bash
#!/bin/bash
# log_analyzer.sh - Analyze common log files

LOG_DIR="/var/log"
ERROR_KEYWORDS="error\|fail\|critical\|panic\|fatal"

echo "=== LOG ANALYSIS REPORT ==="
echo ""

# Check system logs
echo "=== SYSTEM LOG ERRORS (last hour) ==="
journalctl --since "1 hour ago" | grep -i "$ERROR_KEYWORDS" | tail -20
echo ""

# Check auth log for failed attempts
echo "=== FAILED LOGIN ATTEMPTS ==="
grep "Failed password" "$LOG_DIR/auth.log" 2>/dev/null | tail -10
echo ""

# Check for OOM kills
echo "=== OUT OF MEMORY KILLS ==="
grep -i "oom" "$LOG_DIR/syslog" 2>/dev/null | tail -10
echo ""

# Check for disk errors
echo "=== DISK ERRORS ==="
dmesg | grep -i "disk\|sda\|sdb\|error" | tail -10
echo ""
Network Diagnostic Script / Script de Diagnóstico de Red
bash
#!/bin/bash
# network_check.sh - Comprehensive network check

echo "=== NETWORK DIAGNOSTIC ==="
echo ""

echo "=== NETWORK INTERFACES ==="
ip addr show
echo ""

echo "=== ROUTING TABLE ==="
ip route show
echo ""

echo "=== DNS CONFIGURATION ==="
cat /etc/resolv.conf
echo ""

echo "=== CONNECTIVITY TESTS ==="
echo "Testing Google DNS..."
ping -c 3 8.8.8.8
echo ""

echo "Testing DNS resolution..."
nslookup google.com
echo ""

echo "=== OPEN PORTS ==="
ss -tulpn
echo ""

echo "=== FIREWALL STATUS ==="
sudo iptables -L -n -v 2>/dev/null || echo "iptables not available"
sudo ufw status 2>/dev/null || echo "ufw not available"
echo ""
```
## Service Health Check Script / Script de Chequeo de Salud de Servicios
```bash
#!/bin/bash
# service_check.sh - Check common services

SERVICES=("nginx" "apache2" "mysql" "postgresql" "ssh" "docker")

echo "=== SERVICE HEALTH CHECK ==="
echo ""

for service in "${SERVICES[@]}"; do
    if systemctl is-enabled "$service" 2>/dev/null | grep -q "enabled"; then
        status=$(systemctl is-active "$service")
        if [ "$status" = "active" ]; then
            echo "✅ $service: ACTIVE"
        else
            echo "❌ $service: INACTIVE"
            echo "   Status: $(systemctl status "$service" | grep "Active:")"
        fi
    else
        echo "⚪ $service: NOT ENABLED"
    fi
done

echo ""
echo "=== LISTENING PORTS ==="
ss -tulpn | grep -E ":80|:443|:3306|:5432|:22"
Backup Verification Script / Script de Verificación de Backups
bash
#!/bin/bash
# backup_check.sh - Verify backup integrity

BACKUP_DIR="/backup"
DAYS_OLD=7

echo "=== BACKUP VERIFICATION ==="
echo ""

echo "=== RECENT BACKUPS ==="
find "$BACKUP_DIR" -type f -mtime -$DAYS_OLD -exec ls -lh {} \;
echo ""

echo "=== BACKUP SIZE CHECK ==="
du -sh "$BACKUP_DIR"
echo ""

echo "=== TEST RESTORE (if applicable) ==="
# Example for MySQL backup
if [ -f "$BACKUP_DIR/latest.sql.gz" ]; then
    echo "Testing MySQL backup extraction..."
    gunzip -t "$BACKUP_DIR/latest.sql.gz" && echo "✅ Backup file is valid"
else
    echo "No MySQL backup found"
fi
echo ""
```
##5. Incident Documentation / Documentación de Incidentes
Incident Report Template / Plantilla de Reporte de Incidente
```
# Incident Report: [Brief Title]

## 1. Executive Summary
**Date:** [YYYY-MM-DD]  
**Time detected:** [HH:MM]  
**Time resolved:** [HH:MM]  
**Duration:** [X hours/minutes]  
**Impact:** [High/Medium/Low]  
**Affected services:** [List services]

## 2. Timeline
| Time | Event | Action Taken |
|------|-------|-------------|
| 14:30 | Alert received: High CPU on web-server-01 | Acknowledged alert |
| 14:35 | Investigation started | Logged into server |
| 14:40 | Identified runaway process | Collected process info |
| 14:45 | Determined root cause | Documented findings |
| 14:50 | Implemented fix | Killed process, added monitoring |
| 15:00 | Verified resolution | Confirmed normal operation |

## 3. Root Cause Analysis
**Primary cause:** [What directly caused the incident]
- Example: Unoptimized database query causing high CPU

**Contributing factors:** [What made it worse]
- Example: Lack of query monitoring
- Example: No alerting on slow queries

## 4. Impact Assessment
**Services affected:**
- Website (downtime: 30 minutes)
- API endpoints (partial degradation)

**Users affected:** Approximately 500 concurrent users

**Business impact:** Estimated $X in lost revenue

## 5. Resolution Steps
1. Identified problematic process using `top`
2. Collected diagnostics with `ps aux --sort=-%cpu`
3. Killed process with `kill -9 [PID]`
4. Verified system returned to normal with `htop`
5. Added monitoring rule for similar patterns

## 6. Lessons Learned
**What went well:**
- Quick detection by monitoring system
- Clear escalation process
- Effective communication

**What could be improved:**
- Need better query optimization
- Should implement automatic killing of runaway processes
- Need more comprehensive logging

## 7. Action Items
| Item | Owner | Due Date | Status |
|------|-------|----------|--------|
| Optimize problematic query | DBA Team | 2024-03-15 | Pending |
| Implement query monitoring | DevOps | 2024-03-20 | In progress |
| Update runbook with this incident | SRE Team | 2024-03-10 | Completed |

## 8. Evidence
- [Screenshot of monitoring dashboard](link)
- [Log excerpts](link)
- [Command output](link)
Post-Mortem Template / Plantilla de Post-Mortem
markdown
# Post-Mortem: [Incident Name]

## Metadata
- **Date:** [YYYY-MM-DD]
- **Author:** [Your Name]
- **Incident ID:** [INC-XXXX]
- **Status:** [Open/Closed]

## Incident Summary
Brief description of what happened, when, and who was affected.

## Timeline of Events
Detailed minute-by-minute account of the incident.

## Root Cause
The fundamental reason why the incident occurred.

## Detection
How the incident was detected (monitoring, user report, etc.)

## Resolution
Steps taken to resolve the incident.

## Impact
Quantitative and qualitative impact of the incident.

## Corrective Actions
Immediate fixes applied during the incident.

## Preventive Actions
Long-term solutions to prevent recurrence.

## Metrics
1. **MTTD (Mean Time to Detect):** [X minutes]
2. **MTTR (Mean Time to Resolve):** [X minutes]
3. **Affected Users:** [Number]
4. **Downtime:** [X minutes]

## Key Takeaways
1. What we learned
2. What we'll do differently
3. Process improvements needed

## Attachments
- Logs
- Graphs
- Config changes
- Chat transcripts
Runbook Template / Plantilla de Runbook
markdown
# Runbook: [Procedure Name]

## Purpose
Brief description of what this runbook is for.

## Prerequisites
- Required permissions
- Necessary tools
- Pre-checks

## Procedure

### Step 1: [Step Name]
**Command:** `command_to_run`
**Expected output:** [What you should see]
**Error handling:** [What to do if it fails]

### Step 2: [Step Name]
**Command:** `command_to_run`
**Expected output:** [What you should see]
**Error handling:** [What to do if it fails]

## Verification
How to verify the procedure was successful.

## Rollback
Steps to undo the procedure if something goes wrong.

## References
- Related documentation
- Previous incidents
- Team contacts

## Revision History
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2024-01-15 | 1.0 | John Doe | Initial version |
🎯 Quick Reference Cheatsheet / Hoja de Referencia Rápida
When X happens, check Y / Cuando X pasa, verifica Y
text
Website down? → Check: nginx status, port 80, disk space, logs
Can't SSH? → Check: sshd status, firewall, network, keys
Database slow? → Check: connections, queries, disk I/O, memory
High CPU? → Check: top processes, runaway scripts, malware
No disk space? → Check: logs, backups, cache, large files
Essential Commands Flowchart / Diagrama de Comandos Esenciales
text
Start: Problem detected
  ↓
Check: systemctl status [service]
  ↓
Check: journalctl -u [service] --since "10 min ago"
  ↓
Check: df -h (disk space)
  ↓
Check: free -h (memory)
  ↓
Check: top/htop (processes)
  ↓
Check: ss -tulpn (network)
  ↓
Implement fix
  ↓
Document everything
📚 Recommended Resources / Recursos Recomendados
Books / Libros
"The Practice of System and Network Administration" by Thomas Limoncelli

"Site Reliability Engineering" by Google

"Linux Troubleshooting Bible" by Christopher Negus

Websites / Sitios Web
Linux Documentation Project

Server Fault

DigitalOcean Community

Tools to Learn / Herramientas para Aprender
Prometheus + Grafana (monitoring)

ELK Stack (logging)

Ansible (automation)

Docker (containers)
```
# Remember / Recuerda: Good troubleshooting is 80% methodology and 20% technical knowledge. Stay calm, be systematic, and document everything.

# El buen troubleshooting es 80% metodología y 20% conocimiento técnico. Mantén la calma, sé sistemático y documenta todo.
