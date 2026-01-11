## Scripts de Automatización para Administradores de Sistemas / Automation Scripts for System Administrators
# Índice / Table of Contents
```
Filosofía de la Automatización / Automation Philosophy

Estructura Profesional de Scripts / Professional Script Structure

Backups Automatizados / Automated Backups

Monitoreo y Alertas / Monitoring & Alerts

Gestión de Usuarios / User Management

Mantenimiento de Sistemas / System Maintenance

Seguridad y Auditoría / Security & Audit

Despliegue y Configuración / Deployment & Configuration

Programación con Cron / Scheduling with Cron
```
## 1. Filosofía de la Automatización / Automation Philosophy
Principios de Automatización Efectiva / Effective Automation Principles
```bash
# REGLA 1: Si lo haces más de 2 veces, automatízalo
# RULE 1: If you do it more than 2 times, automate it

# REGLA 2: Documenta antes de automatizar
# RULE 2: Document before automating

# REGLA 3: Comienza simple, mejora después
# RULE 3: Start simple, improve later

# REGLA 4: Incluye logging y alertas
# RULE 4: Include logging and alerts

# REGLA 5: Prueba en ambiente no productivo primero
# RULE 5: Test in non-production first
```
# Jerarquía de Automatización / Automation Hierarchy
```bash
Nivel 1: Comandos manuales documentados
         # Documented manual commands

Nivel 2: Scripts básicos con parámetros
         # Basic scripts with parameters

Nivel 3: Scripts con logging y errores controlados
         # Scripts with logging and error handling

Nivel 4: Automatización programada (Cron, Systemd timers)
         # Scheduled automation

Nivel 5: Orquestación (Ansible, Puppet, Chef)
         # Orchestration tools
```
## 2. Estructura Profesional de Scripts / Professional Script Structure
Plantilla Base para Scripts Bash / Bash Script Template
```bash
#!/usr/bin/env bash
# ==============================================================================
# Nombre: nombre-script.sh
# Descripción: Descripción clara del propósito del script
# Autor: Tu Nombre
# Versión: 1.0.0
# Fecha: $(date +%Y-%m-%d)
# Licencia: MIT
# ==============================================================================

# ------------------------------------------------------------------------------
# CONFIGURACIÓN
# ------------------------------------------------------------------------------
set -o errexit          # Exit on error
set -o nounset          # Exit on undefined variables
set -o pipefail         # Capture pipe failures

readonly SCRIPT_NAME="$(basename "${0}")"
readonly SCRIPT_DIR="$(cd "$(dirname "${0}")" && pwd)"
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.*}.log"
readonly LOCK_FILE="/tmp/${SCRIPT_NAME%.*}.lock"

# ------------------------------------------------------------------------------
# CONSTANTES Y CONFIGURACIONES
# ------------------------------------------------------------------------------
readonly BACKUP_DIR="/backup"
readonly RETENTION_DAYS=30
readonly MAX_DISK_USAGE=90

# ------------------------------------------------------------------------------
# FUNCIONES DE UTILIDAD
# ------------------------------------------------------------------------------
log_info() {
    echo "[INFO] $(date '+%Y-%m-%d %H:%M:%S') - $*" | tee -a "${LOG_FILE}"
}

log_error() {
    echo "[ERROR] $(date '+%Y-%m-%d %H:%M:%S') - $*" | tee -a "${LOG_FILE}" >&2
}

check_root() {
    if [[ ${EUID} -ne 0 ]]; then
        log_error "Este script requiere privilegios de root"
        exit 1
    fi
}

send_alert() {
    local subject="$1"
    local message="$2"
    # Implementar envío de alerta (email, Slack, etc.)
    echo "ALERTA: ${subject}" | tee -a "${LOG_FILE}"
}

# ------------------------------------------------------------------------------
# FUNCIONES PRINCIPALES
# ------------------------------------------------------------------------------
backup_database() {
    local db_name="$1"
    local backup_file="${BACKUP_DIR}/${db_name}_$(date +%Y%m%d_%H%M%S).sql.gz"
    
    log_info "Iniciando backup de ${db_name}"
    
    if mysqldump "${db_name}" | gzip > "${backup_file}"; then
        log_info "Backup completado: ${backup_file}"
        echo "${backup_file}"
    else
        log_error "Falló el backup de ${db_name}"
        send_alert "Backup Fallido" "Base de datos: ${db_name}"
        return 1
    fi
}

# ------------------------------------------------------------------------------
# MANEJO DE ARGUMENTOS
# ------------------------------------------------------------------------------
parse_args() {
    while [[ $# -gt 0 ]]; do
        case $1 in
            -h|--help)
                show_help
                exit 0
                ;;
            -v|--verbose)
                readonly VERBOSE=true
                ;;
            --db=*)
                readonly DATABASE="${1#*=}"
                ;;
            *)
                log_error "Argumento desconocido: $1"
                exit 1
                ;;
        esac
        shift
    done
}

show_help() {
    cat << EOF
Uso: ${SCRIPT_NAME} [OPCIONES]

Descripción:
  Realiza backup de bases de datos MySQL con rotación automática.

Opciones:
  -h, --help          Muestra este mensaje de ayuda
  -v, --verbose       Modo verboso
  --db=NOMBRE         Nombre de la base de datos (requerido)

Ejemplos:
  ${SCRIPT_NAME} --db=mi_base
  ${SCRIPT_NAME} --db=produccion --verbose
EOF
}

# ------------------------------------------------------------------------------
# FUNCIÓN PRINCIPAL
# ------------------------------------------------------------------------------
main() {
    # Verificar root
    check_root
    
    # Parsear argumentos
    parse_args "$@"
    
    # Verificar requerimientos
    if [[ -z "${DATABASE:-}" ]]; then
        log_error "Se requiere especificar una base de datos"
        show_help
        exit 1
    fi
    
    # Crear directorio de backups si no existe
    mkdir -p "${BACKUP_DIR}"
    
    # Realizar backup
    if backup_file=$(backup_database "${DATABASE}"); then
        # Rotar backups antiguos
        find "${BACKUP_DIR}" -name "${DATABASE}_*.sql.gz" -mtime +${RETENTION_DAYS} -delete
        
        log_info "Proceso completado exitosamente"
        exit 0
    else
        log_error "Proceso falló"
        exit 1
    fi
}

# ------------------------------------------------------------------------------
# EJECUCIÓN
# ------------------------------------------------------------------------------
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```
## 3. Backups Automatizados / Automated Backups
Backup de Directorios con Rotación / Directory Backup with Rotation
```bash
#!/usr/bin/env bash
# backup-directorio.sh - Backup de directorios con rotación y verificación

readonly SOURCE_DIRS=(
    "/etc"
    "/var/www"
    "/home/usuario/datos"
)

readonly BACKUP_ROOT="/backup"
readonly ENCRYPT_KEY="/etc/backup.key"
readonly RETENTION_DAYS=7

main() {
    local backup_date=$(date +%Y%m%d_%H%M%S)
    local backup_dir="${BACKUP_ROOT}/backup_${backup_date}"
    
    mkdir -p "${backup_dir}"
    
    # Backup de cada directorio
    for dir in "${SOURCE_DIRS[@]}"; do
        if [[ -d "${dir}" ]]; then
            local dir_name=$(basename "${dir}")
            local backup_file="${backup_dir}/${dir_name}.tar.gz"
            
            echo "Backup de ${dir}..."
            tar -czf "${backup_file}" -C "$(dirname "${dir}")" "$(basename "${dir}")"
            
            # Verificar integridad
            if tar -tzf "${backup_file}" >/dev/null 2>&1; then
                echo "✓ Backup verificado: ${backup_file}"
            else
                echo "✗ Backup corrupto: ${backup_file}"
                return 1
            fi
        fi
    done
    
    # Encriptar backup sensible
    if [[ -f "${ENCRYPT_KEY}" ]]; then
        gpg --batch --yes --passphrase-file "${ENCRYPT_KEY}" \
            --output "${backup_dir}.gpg" --symmetric "${backup_dir}.tar.gz"
    fi
    
    # Rotación de backups antiguos
    find "${BACKUP_ROOT}" -name "backup_*" -type d -mtime +${RETENTION_DAYS} -exec rm -rf {} +
    
    echo "Backup completado: ${backup_dir}"
}
Backup de MySQL con Notificaciones / MySQL Backup with Notifications
bash
#!/usr/bin/env bash
# backup-mysql.sh - Backup completo de MySQL con notificaciones

readonly MYSQL_USER="backup_user"
readonly MYSQL_PASS="password_segura"
readonly BACKUP_DIR="/backup/mysql"
readonly NOTIFY_EMAIL="admin@empresa.com"

main() {
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local backup_file="${BACKUP_DIR}/full_backup_${timestamp}.sql.gz"
    
    # Obtener lista de bases de datos excluyendo las del sistema
    local databases=$(mysql -u"${MYSQL_USER}" -p"${MYSQL_PASS}" -e "SHOW DATABASES;" | \
        grep -Ev "(Database|information_schema|performance_schema|mysql|sys)")
    
    # Backup completo
    echo "Iniciando backup completo de MySQL..."
    mysqldump -u"${MYSQL_USER}" -p"${MYSQL_PASS}" \
        --all-databases \
        --single-transaction \
        --routines \
        --triggers \
        --events | gzip > "${backup_file}"
    
    # Verificar backup
    if [[ ${PIPESTATUS[0]} -eq 0 ]]; then
        local file_size=$(du -h "${backup_file}" | cut -f1)
        echo "✓ Backup exitoso: ${backup_file} (${file_size})"
        
        # Enviar notificación
        send_notification "success" "Backup MySQL completado" "Tamaño: ${file_size}"
    else
        echo "✗ Error en backup de MySQL"
        send_notification "error" "Fallo backup MySQL" "Verificar logs"
        return 1
    fi
}

send_notification() {
    local status="$1"
    local subject="$2"
    local message="$3"
    
    # Enviar email
    echo "Estado: ${status}" | mail -s "[${status}] ${subject}" "${NOTIFY_EMAIL}"
    
    # Registrar en log
    echo "$(date): ${subject} - ${message}" >> "/var/log/backup-mysql.log"
}
```
## 4. Monitoreo y Alertas / Monitoring & Alerts
Monitor de Recursos del Sistema / System Resource Monitor
```bash
#!/usr/bin/env bash
# monitor-recursos.sh - Monitoreo y alerta de recursos del sistema

readonly ALERT_THRESHOLDS=(
    "disk_usage:90"
    "memory_usage:85"
    "cpu_load:5.0"
    "swap_usage:50"
)

readonly ALERT_RECIPIENTS=("admin@empresa.com")

main() {
    local alerts=()
    
    # Verificar uso de disco
    local disk_usage=$(df / | awk 'END{print $5}' | sed 's/%//')
    if [[ ${disk_usage} -gt 90 ]]; then
        alerts+=("Uso de disco: ${disk_usage}%")
    fi
    
    # Verificar memoria
    local mem_usage=$(free | awk '/Mem:/ {printf "%.0f", $3/$2*100}')
    if [[ ${mem_usage} -gt 85 ]]; then
        alerts+=("Uso de memoria: ${mem_usage}%")
    fi
    
    # Verificar carga de CPU
    local cpu_load=$(awk '{print $1}' /proc/loadavg)
    local cpu_cores=$(nproc)
    local load_threshold=$(echo "${cpu_cores} * 1.5" | bc)
    
    if (( $(echo "${cpu_load} > ${load_threshold}" | bc -l) )); then
        alerts+=("Carga de CPU alta: ${cpu_load}")
    fi
    
    # Verificar procesos zombie
    local zombie_count=$(ps aux | awk '$8=="Z" {print $0}' | wc -l)
    if [[ ${zombie_count} -gt 0 ]]; then
        alerts+=("Procesos zombie: ${zombie_count}")
    fi
    
    # Si hay alertas, enviar notificación
    if [[ ${#alerts[@]} -gt 0 ]]; then
        send_alert "${alerts[@]}"
    else
        echo "✓ Sistema OK - $(date)" >> /var/log/monitor.log
    fi
}

send_alert() {
    local alert_message="ALERTA DEL SISTEMA - $(date)\n\n"
    alert_message+="Host: $(hostname)\n"
    alert_message+="IP: $(hostname -I)\n\n"
    
    for alert in "$@"; do
        alert_message+="• ${alert}\n"
    done
    
    alert_message+="\nAcciones recomendadas:\n"
    alert_message+="1. Verificar logs del sistema\n"
    alert_message+="2. Revisar procesos en ejecución\n"
    alert_message+="3. Limpiar espacio si es necesario\n"
    
    # Enviar por diferentes canales
    echo -e "${alert_message}" | mail -s "ALERTA: $(hostname)" "${ALERT_RECIPIENTS[@]}"
    
    # También registrar en log central
    echo -e "${alert_message}" >> /var/log/system-alerts.log
}
Monitor de Servicios Críticos / Critical Services Monitor
bash
#!/usr/bin/env bash
# monitor-servicios.sh - Verifica y reinicia servicios críticos

readonly CRITICAL_SERVICES=(
    "sshd"
    "nginx"
    "mysql"
    "postgresql"
    "docker"
)

readonly MAX_RESTART_ATTEMPTS=3
readonly RESTART_LOG="/var/log/service-restarts.log"

main() {
    for service in "${CRITICAL_SERVICES[@]}"; do
        if systemctl is-enabled "${service}" >/dev/null 2>&1; then
            if ! systemctl is-active --quiet "${service}"; then
                echo "⚠️  Servicio ${service} inactivo, intentando reiniciar..."
                
                if systemctl restart "${service}"; then
                    echo "✓ ${service} reiniciado exitosamente"
                    log_restart "${service}" "success"
                else
                    echo "✗ Falló el reinicio de ${service}"
                    log_restart "${service}" "failed"
                    
                    # Verificar intentos recientes
                    local recent_attempts=$(grep -c "${service}" "${RESTART_LOG}" 2>/dev/null)
                    if [[ ${recent_attempts} -ge ${MAX_RESTART_ATTEMPTS} ]]; then
                        send_critical_alert "${service}"
                    fi
                fi
            fi
        fi
    done
}

log_restart() {
    local service="$1"
    local status="$2"
    echo "$(date): ${service} - ${status}" >> "${RESTART_LOG}"
}

send_critical_alert() {
    local service="$1"
    local message="El servicio ${service} ha fallado ${MAX_RESTART_ATTEMPTS} veces consecutivas. Se requiere intervención manual."
    
    # Enviar alerta crítica
    echo "${message}" | mail -s "CRÍTICO: Servicio ${service} inestable" "soporte@empresa.com"
    
    # También podría enviar SMS o notificación push
    echo "ALERTA CRÍTICA ENVIADA: ${service}"
}
```
## 5. Gestión de Usuarios / User Management
Creación Automatizada de Usuarios / Automated User Creation
```bash
#!/usr/bin/env bash
# crear-usuario.sh - Crea usuarios con configuración consistente

readonly DEFAULT_SHELL="/bin/bash"
readonly SKEL_DIR="/etc/skel"
readonly USER_PREFIX="usr"
readonly USER_HOME_BASE="/home"

main() {
    local username="${1}"
    local full_name="${2}"
    local department="${3:-empleados}"
    
    # Validaciones
    if [[ -z "${username}" || -z "${full_name}" ]]; then
        echo "Uso: $0 <username> <nombre_completo> [departamento]"
        exit 1
    fi
    
    if id "${username}" &>/dev/null; then
        echo "Error: El usuario ${username} ya existe"
        exit 1
    fi
    
    # Generar password aleatorio
    local password=$(openssl rand -base64 12)
    
    # Crear usuario
    useradd -m -s "${DEFAULT_SHELL}" -c "${full_name}" "${username}"
    
    # Establecer password
    echo "${username}:${password}" | chpasswd
    
    # Forzar cambio en primer login
    chage -d 0 "${username}"
    
    # Configurar grupos según departamento
    case "${department}" in
        "dev")
            usermod -aG developers,sudo,docker "${username}"
            ;;
        "ops")
            usermod -aG adm,sudo,backup "${username}"
            ;;
        *)
            usermod -aG users "${username}"
            ;;
    esac
    
    # Configurar SSH
    local ssh_dir="${USER_HOME_BASE}/${username}/.ssh"
    mkdir -p "${ssh_dir}"
    chmod 700 "${ssh_dir}"
    touch "${ssh_dir}/authorized_keys"
    chmod 600 "${ssh_dir}/authorized_keys"
    chown -R "${username}:${username}" "${ssh_dir}"
    
    # Crear estructura de directorios
    mkdir -p "${USER_HOME_BASE}/${username}"/{proyectos,documentos,scripts}
    chown -R "${username}:${username}" "${USER_HOME_BASE}/${username}"
    
    # Registrar creación
    echo "$(date): ${username} - ${full_name} - ${department}" >> /var/log/user-creation.log
    
    # Enviar credenciales (en producción usar método seguro)
    echo "Usuario creado: ${username}"
    echo "Password temporal: ${password}"
    echo "Departamento: ${department}"
}
```
# Auditoría de Usuarios y Permisos / User & Permission Audit
```bash
#!/usr/bin/env bash
# auditoria-usuarios.sh - Auditoría de usuarios y permisos

readonly AUDIT_DATE=$(date +%Y%m%d)
readonly REPORT_DIR="/var/audit"
readonly REPORT_FILE="${REPORT_DIR}/user-audit-${AUDIT_DATE}.csv"

main() {
    mkdir -p "${REPORT_DIR}"
    
    # Cabecera del reporte
    echo "Usuario,Grupos,Ultimo Login,Home,Tamaño Home,Shell" > "${REPORT_FILE}"
    
    # Obtener todos los usuarios del sistema
    getent passwd | while IFS=: read -r username _ uid gid desc home shell; do
        # Solo usuarios regulares (UID >= 1000)
        if [[ ${uid} -ge 1000 ]]; then
            # Información del usuario
            local groups=$(id -Gn "${username}" | tr ' ' ',')
            local last_login=$(lastlog -u "${username}" | tail -1 | awk '{print $4,$5,$6,$7,$8,$9}')
            local home_size=$(du -sh "${home}" 2>/dev/null | cut -f1)
            
            # Agregar al reporte
            echo "\"${username}\",\"${groups}\",\"${last_login}\",\"${home}\",\"${home_size}\",\"${shell}\"" >> "${REPORT_FILE}"
            
            # Verificar problemas comunes
            check_user_issues "${username}" "${home}" "${shell}"
        fi
    done
    
    # Generar resumen
    generate_summary
}

check_user_issues() {
    local username="$1"
    local home="$2"
    local shell="$3"
    
    # Usuarios inactivos (sin login en 90 días)
    if [[ "${shell}" != "/sbin/nologin" && "${shell}" != "/bin/false" ]]; then
        local last_login_seconds=$(date -d "$(lastlog -u "${username}" | tail -1 | awk '{print $4,$5,$6,$7,$8,$9}')" +%s 2>/dev/null)
        local ninety_days_ago=$(date -d "90 days ago" +%s)
        
        if [[ -n "${last_login_seconds}" && ${last_login_seconds} -lt ${ninety_days_ago} ]]; then
            echo "ALERTA: Usuario ${username} inactivo por más de 90 días" >> "${REPORT_DIR}/issues.log"
        fi
    fi
    
    # Home directory con permisos incorrectos
    if [[ -d "${home}" ]]; then
        local home_perms=$(stat -c "%a" "${home}")
        if [[ ${home_perms} -gt 750 ]]; then
            echo "ALERTA: Permisos inseguros en ${home} (${home_perms})" >> "${REPORT_DIR}/issues.log"
        fi
    fi
}

generate_summary() {
    local total_users=$(grep -c '^"' "${REPORT_FILE}")
    local inactive_users=$(grep -c "inactivo" "${REPORT_DIR}/issues.log" 2>/dev/null || echo "0")
    
    cat > "${REPORT_DIR}/summary-${AUDIT_DATE}.txt" << EOF
REPORTE DE AUDITORÍA DE USUARIOS
Fecha: $(date)
================================

Estadísticas:
- Total usuarios: ${total_users}
- Usuarios inactivos: ${inety_days_ago}
- Problemas detectados: $(wc -l < "${REPORT_DIR}/issues.log" 2>/dev/null || echo "0")

Recomendaciones:
1. Deshabilitar usuarios inactivos
2. Revisar permisos de directorios home
3. Eliminar cuentas de empleados que ya no trabajan

El reporte completo está en: ${REPORT_FILE}
EOF
    
    echo "Auditoría completada. Reporte: ${REPORT_FILE}"
}
🛠️ 6. Mantenimiento de Sistemas / System Maintenance
Limpieza Automatizada del Sistema / Automated System Cleanup
bash
#!/usr/bin/env bash
# limpieza-sistema.sh - Mantenimiento y limpieza automatizada

readonly LOG_DIR="/var/log/cleanup"
readonly BACKUP_BEFORE_CLEANUP=true

main() {
    mkdir -p "${LOG_DIR}"
    local log_file="${LOG_DIR}/cleanup-$(date +%Y%m%d).log"
    
    echo "=== INICIANDO LIMPIEZA DEL SISTEMA - $(date) ===" | tee -a "${log_file}"
    
    # 1. Limpiar caché de paquetes
    cleanup_package_cache | tee -a "${log_file}"
    
    # 2. Rotar y limpiar logs
    cleanup_old_logs | tee -a "${log_file}"
    
    # 3. Limpiar archivos temporales
    cleanup_temp_files | tee -a "${log_file}"
    
    # 4. Limpiar descargas antiguas
    cleanup_old_downloads | tee -a "${log_file}"
    
    # 5. Reporte de espacio liberado
    generate_cleanup_report | tee -a "${log_file}"
    
    echo "=== LIMPIEZA COMPLETADA - $(date) ===" | tee -a "${log_file}"
}

cleanup_package_cache() {
    echo "🔧 Limpiando caché de paquetes..."
    
    local freed_space=0
    
    # Debian/Ubuntu
    if command -v apt-get &>/dev/null; then
        apt-get clean
        apt-get autoremove --purge -y
        freed_space=$(du -sh /var/cache/apt/ | cut -f1)
    fi
    
    # RHEL/CentOS
    if command -v yum &>/dev/null; then
        yum clean all
        yum autoremove -y
        freed_space=$(du -sh /var/cache/yum/ | cut -f1)
    fi
    
    echo "✓ Caché de paquetes limpiado (${freed_space} liberados)"
}

cleanup_old_logs() {
    echo " Limpiando logs antiguos..."
    
    # Rotar logs manualmente si es necesario
    logrotate -f /etc/logrotate.conf 2>/dev/null || true
    
    # Eliminar logs comprimidos antiguos
    local deleted_logs=$(find /var/log -name "*.gz" -o -name "*.1" -mtime +30 -delete -print | wc -l)
    
    # Limpiar journal (systemd)
    if command -v journalctl &>/dev/null; then
        journalctl --vacuum-time=7d
    fi
    
    echo "✓ ${deleted_logs} archivos de log antiguos eliminados"
}

cleanup_temp_files() {
    echo "  Limpiando archivos temporales..."
    
    # Directorios temporales del sistema
    local temp_dirs=(
        "/tmp"
        "/var/tmp"
        "${HOME}/.cache"
        "${HOME}/.local/share/Trash"
    )
    
    local total_deleted=0
    
    for dir in "${temp_dirs[@]}"; do
        if [[ -d "${dir}" ]]; then
            local deleted=$(find "${dir}" -type f -atime +7 -delete -print | wc -l)
            total_deleted=$((total_deleted + deleted))
        fi
    done
    
    echo "✓ ${total_deleted} archivos temporales eliminados"
}

cleanup_old_downloads() {
    echo " Limpiando descargas antiguas..."
    
    local downloads_dir="${HOME}/Downloads"
    local deleted_count=0
    
    if [[ -d "${downloads_dir}" ]]; then
        # Archivos mayores a 30 días
        deleted_count=$(find "${downloads_dir}" -type f -mtime +30 -delete -print | wc -l)
        
        # Directorios vacíos
        find "${downloads_dir}" -type d -empty -delete
    fi
    
    echo "✓ ${deleted_count} descargas antiguas eliminadas"
}

generate_cleanup_report() {
    echo " REPORTE DE LIMPIEZA"
    echo "----------------------"
    df -h / | awk 'NR==2 {print "Espacio disponible en /: " $4}'
    echo "Fecha próxima limpieza: $(date -d "+7 days" "+%Y-%m-%d")"
}
```
## 7. Seguridad y Auditoría / Security & Audit
Escaneo de Vulnerabilidades Básico / Basic Vulnerability Scan
```bash
#!/usr/bin/env bash
# escaneo-seguridad.sh - Escaneo básico de seguridad

readonly SCAN_DATE=$(date +%Y%m%d)
readonly REPORT_DIR="/var/security-scans"
readonly REPORT_FILE="${REPORT_DIR}/scan-${SCAN_DATE}.txt"

main() {
    mkdir -p "${REPORT_DIR}"
    
    echo "REPORTE DE ESCANEO DE SEGURIDAD" > "${REPORT_FILE}"
    echo "Fecha: $(date)" >> "${REPORT_FILE}"
    echo "Host: $(hostname)" >> "${REPORT_FILE}"
    echo "======================================" >> "${REPORT_FILE}"
    
    # 1. Verificar usuarios y permisos
    check_users_and_permissions >> "${REPORT_FILE}"
    
    # 2. Verificar servicios y puertos
    check_services_and_ports >> "${REPORT_FILE}"
    
    # 3. Verificar actualizaciones de seguridad
    check_security_updates >> "${REPORT_FILE}"
    
    # 4. Verificar configuración SSH
    check_ssh_config >> "${REPORT_FILE}"
    
    # 5. Verificar logs de seguridad
    check_security_logs >> "${REPORT_FILE}"
    
    # Generar resumen
    generate_security_summary >> "${REPORT_FILE}"
    
    echo "Escaneo completado. Reporte: ${REPORT_FILE}"
}

check_users_and_permissions() {
    echo ""
    echo "1. USUARIOS Y PERMISOS"
    echo "----------------------"
    
    # Usuarios sin password
    echo "Usuarios sin password:"
    grep '^[^:]*::' /etc/shadow | cut -d: -f1
    
    # Usuarios con UID 0 (root)
    echo ""
    echo "Usuarios con UID 0:"
    awk -F: '$3==0 {print $1}' /etc/passwd
    
    # Archivos con permisos peligrosos
    echo ""
    echo "Archivos con permisos peligrosos:"
    find / -type f \( -perm -4000 -o -perm -2000 \) -exec ls -la {} \; 2>/dev/null | head -10
}

check_services_and_ports() {
    echo ""
    echo "2. SERVICIOS Y PUERTOS"
    echo "----------------------"
    
    # Puertos abiertos
    echo "Puertos escuchando:"
    ss -tulpn | grep LISTEN
    
    # Servicios activos
    echo ""
    echo "Servicios activos:"
    systemctl list-units --type=service --state=running | head -20
}

check_security_updates() {
    echo ""
    echo "3. ACTUALIZACIONES DE SEGURIDAD"
    echo "-------------------------------"
    
    if command -v apt-get &>/dev/null; then
        apt-get update >/dev/null 2>&1
        apt-get upgrade --dry-run | grep -i security
    elif command -v yum &>/dev/null; then
        yum check-update --security
    fi
}

check_ssh_config() {
    echo ""
    echo "4. CONFIGURACIÓN SSH"
    echo "-------------------"
    
    local sshd_config="/etc/ssh/sshd_config"
    
    if [[ -f "${sshd_config}" ]]; then
        echo "PermitRootLogin: $(grep -i "^PermitRootLogin" "${sshd_config}" || echo "No configurado")"
        echo "PasswordAuthentication: $(grep -i "^PasswordAuthentication" "${sshd_config}" || echo "No configurado")"
        echo "Protocol: $(grep -i "^Protocol" "${sshd_config}" || echo "No configurado")"
    fi
}

check_security_logs() {
    echo ""
    echo "5. LOGS DE SEGURIDAD"
    echo "-------------------"
    
    echo "Intentos fallidos de login (últimas 24h):"
    grep "Failed password" /var/log/auth.log 2>/dev/null | tail -5 || echo "No encontrado"
    
    echo ""
    echo "Usuarios que han usado sudo (últimas 24h):"
    grep "sudo:" /var/log/auth.log 2>/dev/null | tail -5 || echo "No encontrado"
}

generate_security_summary() {
    echo ""
    echo "RESUMEN DE SEGURIDAD"
    echo "===================="
    echo "Puntos críticos encontrados: $(grep -c "CRITICO\|WARNING" "${REPORT_FILE}" 2>/dev/null || echo "0")"
    echo ""
    echo "Recomendaciones inmediatas:"
    echo "1. Actualizar paquetes de seguridad"
    echo "2. Revisar usuarios sin password"
    echo "3. Verificar servicios innecesarios"
    echo "4. Configurar fail2ban para protección SSH"
}
```
# Monitoreo de Cambios en Archivos Críticos / Critical Files Change Monitor
```bash
#!/usr/bin/env bash
# monitor-cambios.sh - Detecta cambios en archivos críticos del sistema

readonly MONITOR_FILES=(
    "/etc/passwd"
    "/etc/shadow"
    "/etc/sudoers"
    "/etc/ssh/sshd_config"
    "/etc/hosts"
    "/etc/crontab"
)

readonly BASELINE_DIR="/var/baseline"
readonly ALERT_THRESHOLD=2  # Alertar después de N cambios

main() {
    mkdir -p "${BASELINE_DIR}"
    
    # Verificar si existe baseline
    if [[ ! -f "${BASELINE_DIR}/file_hashes.txt" ]]; then
        create_baseline
        echo "Baseline creado. Ejecutar nuevamente para detectar cambios."
        return 0
    fi
    
    # Comparar con baseline
    detect_changes
}

create_baseline() {
    echo "Creando baseline de archivos críticos..."
    
    for file in "${MONITOR_FILES[@]}"; do
        if [[ -f "${file}" ]]; then
            local hash=$(sha256sum "${file}" | cut -d' ' -f1)
            echo "${file}:${hash}:$(stat -c %Y "${file}")" >> "${BASELINE_DIR}/file_hashes.txt"
        fi
    done
    
    echo "Baseline guardado en ${BASELINE_DIR}/file_hashes.txt"
}

detect_changes() {
    local changes_detected=0
    
    echo "Verificando cambios en archivos críticos..."
    
    while IFS=: read -r baseline_file baseline_hash baseline_mtime; do
        if [[ -f "${baseline_file}" ]]; then
            local current_hash=$(sha256sum "${baseline_file}" | cut -d' ' -f1)
            local current_mtime=$(stat -c %Y "${baseline_file}")
            
            if [[ "${current_hash}" != "${baseline_hash}" ]]; then
                echo "  CAMBIO DETECTADO: ${baseline_file}"
                echo "   Hash anterior: ${baseline_hash}"
                echo "   Hash actual: ${current_hash}"
                
                # Registrar cambio
                log_change "${baseline_file}" "${baseline_hash}" "${current_hash}"
                
                changes_detected=$((changes_detected + 1))
            fi
        else
            echo " ARCHIVO ELIMINADO: ${baseline_file}"
            changes_detected=$((changes_detected + 1))
        fi
    done < "${BASELINE_DIR}/file_hashes.txt"
    
    # Alertar si hay muchos cambios
    if [[ ${changes_detected} -ge ${ALERT_THRESHOLD} ]]; then
        send_alert "Múltiples cambios detectados en archivos críticos"
    fi
    
    echo "Cambios detectados: ${changes_detected}"
}

log_change() {
    local file="$1"
    local old_hash="$2"
    local new_hash="$3"
    
    local log_entry="$(date): ${file} cambió. Old: ${old_hash:0:16}... New: ${new_hash:0:16}..."
    echo "${log_entry}" >> "${BASELINE_DIR}/change_log.txt"
    
    # Opcional: crear backup del archivo modificado
    cp "${file}" "${BASELINE_DIR}/backups/$(basename "${file}").$(date +%Y%m%d_%H%M%S)"
}

send_alert() {
    local message="$1"
    
    echo " ALERTA DE SEGURIDAD: ${message}"
    
    # Enviar notificación
    echo "${message}" | mail -s "ALERTA: Cambios en archivos críticos" "security@empresa.com"
    
    # También registrar en syslog
    logger -t "file_monitor" "${message}"
}
```
## 8. Despliegue y Configuración / Deployment & Configuration
Despliegue Automatizado de Aplicación Web / Automated Web App Deployment
```bash
#!/usr/bin/env bash
# deploy-webapp.sh - Despliegue automatizado de aplicación web

readonly APP_NAME="mi-aplicacion"
readonly APP_USER="www-data"
readonly DEPLOY_DIR="/var/www/${APP_NAME}"
read readonly BACKUP_DIR="/backup/${APP_NAME}"
readonly GIT_REPO="https://github.com/usuario/repo.git"
readonly BRANCH="main"

main() {
    local action="${1:-deploy}"
    
    case "${action}" in
        "deploy")
            deploy_application
            ;;
        "rollback")
            rollback_application
            ;;
        "status")
            check_application_status
            ;;
        *)
            echo "Uso: $0 {deploy|rollback|status}"
            exit 1
            ;;
    esac
}

deploy_application() {
    echo " Iniciando despliegue de ${APP_NAME}..."
    
    # 1. Crear backup actual
    create_backup
    
    # 2. Clonar o actualizar código
    update_application_code
    
    # 3. Instalar dependencias
    install_dependencies
    
    # 4. Configurar aplicación
    configure_application
    
    # 5. Reiniciar servicios
    restart_services
    
    # 6. Verificar despliegue
    verify_deployment
    
    echo " Despliegue completado exitosamente"
}

create_backup() {
    echo " Creando backup..."
    
    local backup_timestamp=$(date +%Y%m%d_%H%M%S)
    local backup_path="${BACKUP_DIR}/${backup_timestamp}"
    
    mkdir -p "${backup_path}"
    
    if [[ -d "${DEPLOY_DIR}" ]]; then
        cp -r "${DEPLOY_DIR}"/* "${backup_path}/"
        echo "Backup creado: ${backup_path}"
    else
        echo "No existe deploy anterior, continuando..."
    fi
}

update_application_code() {
    echo " Actualizando código..."
    
    if [[ ! -d "${DEPLOY_DIR}/.git" ]]; then
        # Clonar por primera vez
        git clone "${GIT_REPO}" "${DEPLOY_DIR}"
    else
        # Actualizar existente
        cd "${DEPLOY_DIR}"
        git fetch origin
        git checkout "${BRANCH}"
        git pull origin "${BRANCH}"
    fi
    
    # Establecer permisos
    chown -R "${APP_USER}:${APP_USER}" "${DEPLOY_DIR}"
}

install_dependencies() {
    echo " Instalando dependencias..."
    
    cd "${DEPLOY_DIR}"
    
    # Dependencias de sistema
    if [[ -f "requirements.txt" ]]; then
        pip install -r requirements.txt
    fi
    
    if [[ -f "package.json" ]]; then
        npm install --production
    fi
    
    if [[ -f "composer.json" ]]; then
        composer install --no-dev
    fi
}

configure_application() {
    echo " Configurando aplicación..."
    
    # Variables de entorno
    if [[ -f "${DEPLOY_DIR}/.env.example" ]]; then
        cp "${DEPLOY_DIR}/.env.example" "${DEPLOY_DIR}/.env"
        # Aquí se podrían establecer variables desde un vault o similar
    fi
    
    # Base de datos
    if [[ -f "${DEPLOY_DIR}/database/migrations" ]]; then
        cd "${DEPLOY_DIR}"
        # Ejecutar migraciones
        # php artisan migrate --force
        # o
        # python manage.py migrate
    fi
}

restart_services() {
    echo " Reiniciando servicios..."
    
    # Reiniciar web server
    systemctl restart nginx
    systemctl restart php-fpm  # Si aplica
    
    # Reiniciar colas de trabajos
    # systemctl restart supervisor
    
    echo "Servicios reiniciados"
}

verify_deployment() {
    echo " Verificando despliegue..."
    
    # Verificar que la aplicación responde
    local response_code=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/)
    
    if [[ "${response_code}" == "200" ]]; then
        echo " Aplicación respondiendo correctamente (HTTP 200)"
    else
        echo " La aplicación no responde correctamente (HTTP ${response_code})"
        # Podría hacer rollback automático aquí
    fi
}

rollback_application() {
    echo " Iniciando rollback..."
    
    local latest_backup=$(ls -td "${BACKUP_DIR}"/*/ | head -1)
    
    if [[ -z "${latest_backup}" ]]; then
        echo " No hay backups disponibles para rollback"
        exit 1
    fi
    
    echo "Restaurando desde: ${latest_backup}"
    
    # Detener servicios
    systemctl stop nginx
    
    # Restaurar backup
    rm -rf "${DEPLOY_DIR}"/*
    cp -r "${latest_backup}"/* "${DEPLOY_DIR}/"
    
    # Restaurar permisos
    chown -R "${APP_USER}:${APP_USER}" "${DEPLOY_DIR}"
    
    # Reiniciar servicios
    systemctl start nginx
    
    echo " Rollback completado desde ${latest_backup}"
}

check_application_status() {
    echo " Estado de la aplicación:"
    
    # Versión actual
    if [[ -d "${DEPLOY_DIR}/.git" ]]; then
        cd "${DEPLOY_DIR}"
        echo "Versión: $(git log --oneline -1)"
    fi
    
    # Estado de servicios
    echo "Servicios:"
    systemctl status nginx --no-pager | grep "Active:"
    
    # Espacio utilizado
    echo "Espacio: $(du -sh "${DEPLOY_DIR}" | cut -f1)"
    
    # Último backup
    local last_backup=$(ls -td "${BACKUP_DIR}"/*/ | head -1)
    if [[ -n "${last_backup}" ]]; then
        echo "Último backup: $(basename "${last_backup}")"
    fi
}
```
## 9. Programación con Cron / Scheduling with Cron
Configuración de Cron Jobs / Cron Jobs Configuration
```bash
#!/usr/bin/env bash
# setup-cron-jobs.sh - Configura trabajos programados comunes

readonly CRON_DIR="/etc/cron.d"
readonly BACKUP_SCRIPT="/usr/local/bin/backup-mysql.sh"
readonly MONITOR_SCRIPT="/usr/local/bin/monitor-recursos.sh"
readonly CLEANUP_SCRIPT="/usr/local/bin/limpieza-sistema.sh"

main() {
    echo "Configurando trabajos programados..."
    
    # 1. Backups diarios a las 2 AM
    setup_backup_cron
    
    # 2. Monitoreo cada 5 minutos
    setup_monitor_cron
    
    # 3. Limpieza semanal los domingos a las 3 AM
    setup_cleanup_cron
    
    # 4. Verificación de seguridad diaria
    setup_security_cron
    
    echo " Configuración de cron completada"
    echo "Ver trabajos programados: crontab -l"
}

setup_backup_cron() {
    cat > "${CRON_DIR}/backup-jobs" << EOF
# Backups automáticos
# ┌────────── minuto (0-59)
# │ ┌──────── hora (0-23)
# │ │ ┌────── día del mes (1-31)
# │ │ │ ┌──── mes (1-12)
# │ │ │ │ ┌── día de la semana (0-6, 0=domingo)
# │ │ │ │ │
# │ │ │ │ │
# │ │ │ │ │
# │ │ │ │ │
# * * * * * usuario comando

# Backup completo diario a las 2 AM
0 2 * * *   root    ${BACKUP_SCRIPT} --full --compress

# Backup incremental cada 6 horas
0 */6 * * * root    ${BACKUP_SCRIPT} --incremental

# Verificación de backups los lunes a las 3 AM
0 3 * * 1   root    ${BACKUP_SCRIPT} --verify
EOF
    
    chmod 644 "${CRON_DIR}/backup-jobs"
    echo "✓ Backups programados"
}

setup_monitor_cron() {
    cat > "${CRON_DIR}/monitor-jobs" << EOF
# Monitoreo del sistema
# Nota: */5 significa cada 5 minutos

# Monitoreo de recursos cada 5 minutos
*/5 * * * *   root    ${MONITOR_SCRIPT} --check-all

# Reporte diario de recursos a las 9 AM
0 9 * * *     root    ${MONITOR_SCRIPT} --daily-report

# Alerta si recursos críticos (cada minuto)
* * * * *     root    ${MONITOR_SCRIPT} --critical-alerts
EOF
    
    chmod 644 "${CRON_DIR}/monitor-jobs"
    echo "✓ Monitoreo programado"
}

setup_cleanup_cron() {
    cat > "${CRON_DIR}/cleanup-jobs" << EOF
# Mantenimiento y limpieza

# Limpieza diaria a las 4 AM
0 4 * * *     root    ${CLEANUP_SCRIPT} --daily

# Limpieza semanal completa los domingos a las 3 AM
0 3 * * 0     root    ${CLEANUP_SCRIPT} --weekly

# Rotación de logs diaria a medianoche
0 0 * * *     root    logrotate -f /etc/logrotate.conf

# Actualización de paquetes de seguridad los lunes a las 1 AM
0 1 * * 1     root    apt-get update && apt-get upgrade --security-only -y
EOF
    
    chmod 644 "${CRON_DIR}/cleanup-jobs"
    echo "✓ Limpieza programada"
}

setup_security_cron() {
    cat > "${CRON_DIR}/security-jobs" << EOF
# Tareas de seguridad

# Escaneo de seguridad diario a las 5 AM
0 5 * * *     root    /usr/local/bin/escaneo-seguridad.sh

# Auditoría de usuarios cada domingo a las 2 AM
0 2 * * 0     root    /usr/local/bin/auditoria-usuarios.sh

# Monitoreo de cambios en archivos críticos cada hora
0 * * * *     root    /usr/local/bin/monitor-cambios.sh

# Actualización de firmas de malware diario
30 3 * * *    root    freshclam  # Para ClamAV
EOF
    
    chmod 644 "${CRON_DIR}/security-jobs"
    echo "✓ Seguridad programada"
}
Gestión de Logs de Cron / Cron Logs Management
bash
#!/usr/bin/env bash
# manage-cron-logs.sh - Gestión de logs de trabajos cron

readonly CRON_LOG_DIR="/var/log/cron"
readonly MAX_LOG_DAYS=30
readonly MAX_LOG_SIZE="100M"

main() {
    local action="${1:-rotate}"
    
    case "${action}" in
        "rotate")
            rotate_cron_logs
            ;;
        "clean")
            clean_old_cron_logs
            ;;
        "stats")
            show_cron_stats
            ;;
        *)
            echo "Uso: $0 {rotate|clean|stats}"
            exit 1
            ;;
    esac
}

rotate_cron_logs() {
    echo " Rotando logs de cron..."
    
    mkdir -p "${CRON_LOG_DIR}"
    
    # Rotar logs existentes
    if [[ -f "${CRON_LOG_DIR}/cron.log" ]]; then
        local timestamp=$(date +%Y%m%d_%H%M%S)
        mv "${CRON_LOG_DIR}/cron.log" "${CRON_LOG_DIR}/cron.log.${timestamp}"
        gzip "${CRON_LOG_DIR}/cron.log.${timestamp}"
    fi
    
    # Configurar logrotate para cron si no existe
    if [[ ! -f "/etc/logrotate.d/cron" ]]; then
        cat > "/etc/logrotate.d/cron" << EOF
${CRON_LOG_DIR}/cron.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    create 640 root adm
    sharedscripts
    postrotate
        /usr/bin/systemctl reload cron > /dev/null 2>&1 || true
    endscript
}
EOF
        echo "✓ Configuración de logrotate creada"
    fi
    
    echo "✓ Logs de cron rotados"
}

clean_old_cron_logs() {
    echo " Limpiando logs antiguos de cron..."
    
    local deleted_count=0
    
    # Eliminar logs comprimidos mayores a MAX_LOG_DAYS
    if [[ -d "${CRON_LOG_DIR}" ]]; then
        deleted_count=$(find "${CRON_LOG_DIR}" -name "*.gz" -mtime +${MAX_LOG_DAYS} -delete -print | wc -l)
    fi
    
    # Eliminar logs muy grandes
    find "${CRON_LOG_DIR}" -name "*.log" -size +${MAX_LOG_SIZE} -delete
    
    echo "✓ ${deleted_count} logs antiguos eliminados"
}

show_cron_stats() {
    echo " Estadísticas de trabajos cron"
    echo "================================="
    
    # Total de trabajos configurados
    local total_jobs=$(crontab -l 2>/dev/null | grep -v "^#" | grep -c "^[^#]")
    echo "Trabajos cron configurados: ${total_jobs}"
    
    # Logs por tamaño
    if [[ -d "${CRON_LOG_DIR}" ]]; then
        echo ""
        echo "Uso de logs:"
        du -sh "${CRON_LOG_DIR}"
        
        echo ""
        echo "Archivos de log:"
        ls -lh "${CRON_LOG_DIR}" | head -10
    fi
    
    # Errores recientes
    echo ""
    echo "Errores recientes en logs:"
    grep -i "error\|fail" "${CRON_LOG_DIR}/cron.log" 2>/dev/null | tail -5 || echo "No hay errores recientes"
    
    # Próxima ejecución
    echo ""
    echo "Próximas ejecuciones programadas:"
    when -l 2>/dev/null || echo "Comando 'when' no disponible"
}
```
## Buenas Prácticas Finales / Final Best Practices
Checklist para Scripts de Producción / Production Scripts Checklist
```bash
#  ANTES de usar en producción:
# [ ] 1. Script probado en ambiente de desarrollo
# [ ] 2. Logging implementado
# [ ] 3. Manejo de errores adecuado
# [ ] 4. Validación de parámetros
# [ ] 5. Permisos de archivos revisados
# [ ] 6. Documentación actualizada
# [ ] 7. Backup/rollback disponible
# [ ] 8. Notificaciones configuradas
# [ ] 9. Performance evaluada
# [ ] 10. Revisión de seguridad realizada
```
# Estructura Recomendada de Repositorio / Recommended Repository Structure
```text
scripts-automatizacion/
├── README.md                    # Documentación principal
├── LICENSE                      # Licencia (MIT, GPL, etc.)
├── .gitignore                   # Archivos a ignorar
├── requirements.txt             # Dependencias (si aplica)
│
├── bin/                         # Scripts ejecutables
│   ├── backup/
│   │   ├── backup-mysql.sh
│   │   ├── backup-directorios.sh
│   │   └── restore-backup.sh
│   │
│   ├── monitoreo/
│   │   ├── monitor-recursos.sh
│   │   ├── monitor-servicios.sh
│   │   └── alertas-sistema.sh
│   │
│   ├── seguridad/
│   │   ├── escaneo-seguridad.sh
│   │   ├── auditoria-usuarios.sh
│   │   └── monitor-cambios.sh
│   │
│   └── mantenimiento/
│       ├── limpieza-sistema.sh
│       ├── actualizar-sistema.sh
│       └── optimizar-servidor.sh
│
├── config/                      # Archivos de configuración
│   ├── cron-jobs/
│   ├── logrotate/
│   └── systemd/
│
├── templates/                   # Plantillas reutilizables
│   ├── script-template.sh
│   ├── config-template.conf
│   └── README-template.md
│
├── docs/                        # Documentación detallada
│   ├── instalacion.md
│   ├── uso.md
│   ├── troubleshooting.md
│   └── ejemplos/
│
├── tests/                       # Tests automatizados
│   ├── unitarios/
│   └── integracion/
│
└── logs/                        # Logs de ejemplo (en .gitignore)
```
