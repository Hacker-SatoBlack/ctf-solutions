**Solución CTF  -  Anonforce**

**CTF de Try Hack Me, paso a paso con explicación de técnicas de
enumeración, explotación y post-explotación en Linux.**

![](media_anonforce/media/image1.png)

**La IP objetivo será: 10.66.135.20**

![](media_anonforce/media/image2.png)

Se prueba conexión hacia la IP objetivo por icmp.

![](media_anonforce/media/image3.png)

Se realiza la búsqueda de los primeros 1000 puertos más comunes con
nmap. (**nmap 10.66.135.20**). Se encontró 2 puertos abiertos (**FTP 21
y SSH 22**)

![](media_anonforce/media/image4.png)

Se realizó un escaneo con Nmap utilizando las opciones **-sV** para
identificar versiones de servicios, **-sC** para ejecutar scripts por
defecto y **-sS** (half-open scan) para un escaneo sigiloso, **-O** para
poder identificar el **SO** de la IP Objetivo. Como resultado, se
identificó que el servicio FTP permite **login anónimo** y cuenta con
varios directorios/archivos, lo que representa una posible
vulnerabilidad. (Comando: **nmap -sS -sV -sC -p 21,22 10.66.135.20
-O**), además de una parte informativa del Servidor FTP.

![](media_anonforce/media/image5.png)

Se ingresar por ftp con las credeciales: **user: anonymous, password:
anonymous. **Comando:** ftp 10.66.135.20**

![](media_anonforce/media/image6.png)

Entre todo el contenido que se observa, se ingresa varios directorios
interesantes.

![](media_anonforce/media/image7.png)

Se ingresa a la ruta "/home/melodías" (**cd home** y **cd melodias**) y
se encuentra el archivo "**user.txt**", se descarga dicho archivo.
(**get user.txt**). Agregar que se encontró al usuario "**melodias**"
que puede ser usado para un ataque de fuerza bruta.

![](media_anonforce/media/image8.png)

Se revisa dicho archivo y el valor es la 1era Flag:
**606083fd33beb1284fc51f411a706af8**

![](media_anonforce/media/image9.png)

Revisamos el directorio "**notread**" y se visualizan 2 archivos, se
descargan ambos, comandos: **get backup.pgp** y **get private.asc.**

![](media_anonforce/media/image10.png)

Se visualiza el 1er archivo es una llave privada en formato PGP.

![](media_anonforce/media/image11.png)

Y el 2do archivo requiere una clave secreta para descifrarla.

![](media_anonforce/media/image12.png)

Se convierte la clave PGP en un hash que John entienda con el comando:
**gpg2john private.asc \> pgp_hash.txt**, luego realiza fuerza bruta
para encontrar el passphrase con el comando: **john
\--wordlist=/usr/share/wordlists/rockyou.txt pgp_hash.txt**, encontrando
el passphrase: **xbox360**

![](media_anonforce/media/image13.png)

Se procede a importar la clave privada del formato PGP con el comando:
**gpg \--import private.as**, luego se ingresa el passphrase:
**xbox360**

![](media_anonforce/media/image14.png)
![](media_anonforce/media/image15.png)

Se procede a descifrar el archivo "**backup.pgp**" con el comando: **gpg
-d backup.pgp**, luego se ingresa el passphrase: **xbox36**, obteniendo
como resultado el archivo de password de usuarios **(/etc/passwd).**

![](media_anonforce/media/image16.png)
![](media_anonforce/media/image17.png)

Se crea el archivo "**passwd_root.txt**" y se copia solo el hash del
usuario root
"**\$6\$07nYFaYf\$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0**",
comandos: nano **passwd_root.txt** y cat **passwd_root.txt**

![](media_anonforce/media/image18.png)

Se utiliza el comando: **hashcat \--help \| grep "\\\$6"**, para mostrar
**tod**os los tipos de hashes soportados por **hashcat** que inicien con
\$6\$, se observa que es **SHA512** y modo de hash 1800.

![](media_anonforce/media/image19.png)

Se descifró el hash **SHA-512 (\$6\$)** con usando un ataque de
diccionario con la wordlist **rockyou.txt**, con modo de hash **-m
1800**, con **\--force** ignora advertencias y **-a 0** indica ataque
directo probando cada contraseña de la lista. Comando: **hashcat -m 1800
\--force -a 0 passwd_root.txt /usr/share/wordlists/rockyou.txt**,
obteniendo el password:
**hikari**
![](media_anonforce/media/image20.png)

Se ingresó por ssh con el user: **root** y password: **hikari**

![](media_anonforce/media/image21.png)

Se revisa el contenido de la carpeta actual, encontrando el archivo
**root.txt**, luego se visualiza dentro la 2da Flag:
**f706456440c7af4187810c31c6cebdce**

![](media_anonforce/media/image22.png)

Adicionalmente se pudo obtener el password con un ataque de diccionario.
Comando: **hydra -l root -P /usr/share/wordlists/rockyou.txt -t 64 -f
ssh://10.66.135.20**

![](media_anonforce/media/image23.png)

Eso es todo xd, estaré subiendo mas soluciones de CTF en los siguientes
días, comenten si quiere que resuelva un CTF en especifico.
