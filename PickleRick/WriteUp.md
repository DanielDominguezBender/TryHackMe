# Pickle Rick

Máquina con puerto **22 (SSH)** y **80 (HTTP)**. Se encontró un usuario en el código fuente web (`R1ckRul3s`) y una pista en `robots.txt` (`Wubbalubbadubdub`). No había autenticación por contraseña en SSH (solo llave pública). Se exploró la web (gobuster, assets), se intentó esteganografía sin éxito, se autenticó en un panel web con las credenciales encontradas, ese panel permitía ejecutar comandos como `www-data`. `www-data` podía usar `sudo` sin contraseña → se lanzó `sudo bash` → root → leer los ficheros con los “ingredientes”. Se consiguió reverse shell con `python3` cuando hizo falta. 

Link a la máquina en THM -> [Pickle Rick](https://tryhackme.com/room/picklerick)
Link al write-up completo -> [Pickle Rick - Write Up](https://github.com/DanielDominguezBender/TryHackMe/blob/main/PickleRick/PickleRick.pdf)

# Flujo paso a paso (resumido)

1. **Preparación**

   * Crear directorio del reto (`mkdir picklerick && cd picklerick`).
   * Escaneo con `nmap -p- -sC -sV -A --min-rat=5000 <IP> -oN escaneo_picklerick.txt`.
   * Resultado: puertos 22 (OpenSSH 8.2p1) y 80 (Apache 2.4.41). 

2. **Enumeración web**

   * Abrir la web, inspeccionar código fuente → **usuario** `R1ckRul3s` encontrado.
   * `gobuster` para directorios: encontró `login.php`, `assets/`, `clue.txt`, `robots.txt`.
   * `robots.txt` contenía `Wubbalubbadubdub` (posible contraseña/pista). 

3. **Intentos de autenticación**

   * Probó `hydra` contra SSH con `R1ckRul3s` + rockyou → falló porque SSH no acepta password auth.
   * Verificó métodos SSH con `nmap --script ssh-auth-methods` → solo key-auth. 

4. **Steganografía en assets**

   * Descargó imágenes `portal.jpg` y `rickandmorty.jpeg`. Probó `stegcracker`, `stegseek` → sin éxito. (descarta estego). 

5. **Login en la web**

   * Usando el usuario/pista, consiguió autenticarse en `login.php` → accede a un **Command Panel** que ejecuta comandos con permisos del proceso web.
   * Listado de archivos mostró `Sup3rS3cretPickl3Ingred.txt` (primer ingrediente). 

6. **Comandos remotos y privilegios**

   * Al probar `whoami` → `www-data`. `sudo -l` mostró que **www-data puede ejecutar cualquier comando como cualquier usuario sin password** (gran vector de escalada). 

7. **Reverse shell (cuando fue necesario)**

   * El panel permitía ejecutar `python3` → se construyó una reverse shell en python3 (usar `nc -lvp 9999` en attacker). Funcionó desde VirtualBox (problemas en UTM). 

8. **Escalada a root**

   * Aprovechando la política sudo (`sudo bash`) se obtuvo shell root.
   * Leer `/root/3rd.txt` (tercer ingrediente). Fichero `/home/rick` contenía el segundo ingrediente (cat estaba capado, por eso se usó shell o `echo`/otras técnicas). 

# Comandos clave citados en el write-up

* `nmap -p- -sC -sV -A --min-rat=5000 10.10.16.249 -oN escaneo_picklerick.txt`
* `nmap -p 22 --script ssh-auth-methods 10.10.16.249`
* `gobuster dir -u http://10.10.16.249 -w <wordlist> -x php,txt,png,...`
* `hydra -l R1ckRul3s -P /usr/share/wordlists/rockyou.txt ssh://10.10.16.249`
* Descarga de imágenes: `wget http://10.10.16.249/assets/portal.jpg`
* Reverse shell (ejemplo python3 desde revshells/pentestmonkey) + `nc -lvp 9999`
* `sudo -l` → ver privilegios
* `sudo bash` → escalada a root
* Cadena de decodificados base64: `echo '...' | base64 -d | base64 -d ...` (para hashes/encodings) 

# Hallazgos y artefactos importantes

* Usuario web: `R1ckRul3s`.
* `robots.txt` con pista `Wubbalubbadubdub`.
* Ficheros descubiertos: `Sup3rS3cretPickl3Ingred.txt` (ingrediente 1), archivos bajo `/home/rick` (ingrediente 2), `/root/3rd.txt` (ingrediente 3).
* SSH con solo autenticación por clave (no viable fuerza bruta).
* `www-data` con `sudo` sin contraseña → escalada trivial a root. 

# Lecciones aprendidas / recomendaciones del autor

* Comprobar siempre `ssh-auth-methods` antes de gastar tiempo en bruteforce.
* Steganografía no siempre está presente; prueba varias herramientas (stegcracker, stegseek) y no te encalles.
* Si comandos están capados en el servidor (ej. `cat`), prueba alternativas (`echo`, redirecciones, shells, reverse shell).
* Puedes encadenar decodificados (base64) en una sola línea para ahorrar trabajo.
* Ten plan B: la reverse shell funcionó en VirtualBox cuando UTM daba problemas. 

