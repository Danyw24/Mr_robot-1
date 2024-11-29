# Mr_robot-1 - CTF how to pwn

Esta máquina CTF o **(Capture The Flag)** (Captura la bandera) **mr_robot 1** esta basada en la escena de la serie de mismo nombre **Mr robot** :), En la que se exploran diversas técnicas de explotación, escalada de privilegios y obtención de acceso en un entorno de CTF. A continuación, se describen los pasos y herramientas utilizadas para comprometer la máquina y obtener acceso al **root** de este ctf.

# Requisitos

- [mr_robot 1 - VulnHub](https://www.vulnhub.com/entry/mr-robot-1,151/)
- Virtual Box
- Kali linux 2024.4 - instalado u OS de preferencia 
![image](https://github.com/user-attachments/assets/66db3048-492c-4c8b-8449-d3302f01a3a6)

## Técnicas de Exploración y Explotación

## Enumeracion y recopilacion de datos
-El primer paso es entender la red en la que se trabaja y descubrir dispositivos, en este caso utilizando **arp-scan** (herramienta ya incluida en kali)

![image](https://github.com/user-attachments/assets/d7171dfd-c4eb-41a6-acc0-b96dfcc0ec35)
- **arp-scan** trabaja en la capa 2 del modelo OSI, lo que lo convierte en una buena alternativa a herramientas como **netdiscover**, ya que puede detectar dispositivos en la red local incluso si están ocultos a través de ICMP o sin nombre de host.


## Tecnicas de Enumeración

## Nmap

```bash
nmap -iL hosts.txt --randomize-hosts -f --data-length --top-ports 100 --min-rate 600 -sV -T4 -Pn
```

- `nmap -iL hosts.txt`: Escanea los hosts listados en el archivo `hosts.txt`.
- `--randomize-hosts`: Aleatoriza el orden en el que se escanean los hosts.
- `-f`: Fragmenta los paquetes para evadir detección por sistemas de seguridad, firewalls, IDS , etc..
- `--data-length`: Modifica el tamaño del paquete de datos para evitar detección.
- `--top-ports 100`: Escanea los 100 puertos más comunes.
- `--min-rate 600`: Establece una tasa mínima de envío de 600 paquetes por segundo ( en este caso sabiendo que la maquina es posible que no tenga sistema de deteccion de intrusiones).
- `-sV`: Detecta las versiones de los servicios en los puertos abiertos.
- `-T4`: Establece la velocidad de escaneo a un nivel más alto.
- `-Pn`: Omite el escaneo de ping (sabiendo que todos los hosts están activos).

![image](https://github.com/user-attachments/assets/2d3cf124-93c9-4d6c-be05-e06454d8a30c)


- El escaneo muestra que hay un servicio corriendo en el puerto 80, en este caso la dirección http://192.168.0.19:80
  
![image](https://github.com/user-attachments/assets/e03519c5-0809-4602-9bee-f8024c86c3d8)

- Usando `Crtl + u` es posible acceder al codigo fuente con el fin de obtener mas información

-![image](https://github.com/user-attachments/assets/5230a9af-f777-4e8b-86fc-153ca447f441)

-Esto ya nos puede dar indicios de que probablemente el sitio este basado en **Wordpress**

## Busqueda de directorios

## Gobuster (kali)
```bash
gobuster dir -u http://192.168.0.19/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
```
-Gobuster es una herramienta de fuerza bruta utilizada para descubrir directorios y archivos ocultos en servidores web.

![image](https://github.com/user-attachments/assets/0eb150f3-0db0-4dec-90d1-c4138213c6ce)

-Es util en estos casos ademas de confirmar que el sitio efectivamente usa **Wordpress**

![image](https://github.com/user-attachments/assets/d0707503-0267-4528-b606-e791be060bb6)

-Aqui obtenemos la ruta a la primera llave y otro archivo pareciendo ser un diccionario

-**Key 1-of-3:** `073403c8a58a1f80d943455fb30724b9`

![image](https://github.com/user-attachments/assets/6fdb80ee-a5c7-46d4-97a6-a88622feb979)

- Limpiamos el diccionario de posibles palabras repetidas 

## Tecnicas de Explotación

## Wpscan (kali)

Se encontró un login en WordPress en la dirección `http://192.168.0.19/wp-login.php`. Se utiliza **WPScan** para realizar un ataque de fuerza bruta utilizando el diccionario obtenido `fsociety.txt` con el objetivo de encontrar credenciales válidas.


![image](https://github.com/user-attachments/assets/0b596a4f-56c3-4b67-9ec0-da616e5b337e)



![image](https://github.com/user-attachments/assets/844d2a6f-bb9d-4505-a6d8-99149a222628)


![image](https://github.com/user-attachments/assets/5a72bbec-7ab0-466f-ad69-66eedcc9e0b8)


#Explotación

## Reverse Shell

Una vez obtenidas las **credenciales válidas** para WordPress, ingresamos al **dashboard**. Luego, modifiqué el archivo **404.php** de error, alterando su código en PHP para **obtener una reverse shell**. La **reverse shell** se encuentra en la ruta `/usr/share/webshells/php/php-reverse-shell.php`. 


![image](https://github.com/user-attachments/assets/bacd3660-3390-4360-8fc7-ec249b66123c)




A continuación, provoqué un error 404 en el servidor, lo que causó que el servidor ejecutara el código PHP modificado. Del lado del atacante, **escuchamos la conexión** con **ncat** usando el siguiente comando:

```bash
nc -l -p 4545
```


![image](https://github.com/user-attachments/assets/ba9a79a2-3988-49bc-b027-0198b38737b2)


Una vez adentro busqué "upgradear" la shell en este caso sh, habiendo hecho busquedas en el sistema pude se identifico que habia python instalado por lo cual se ejecuta:



```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Una vez dados estos pasos y dentro de la maquina, el objetivo es conseguir el acceso **root** y las demas **flags** restantes

![image](https://github.com/user-attachments/assets/21435db5-43d6-43cd-a3a2-a81a96ac4797)

- En el directorio de **robot** se encuentra la contraseña de el usuario pero encriptada en md5 por lo cual usé [CrackStation](https://crackstation.net/)


![image](https://github.com/user-attachments/assets/8bf0b219-bcca-4b11-bc98-ab2e21dcc330)

- Por lo cual la password del usuario robot es **abcdefghijklmnopqrstuvwxyz**


![image](https://github.com/user-attachments/assets/e344caf1-521b-4b9c-bfa9-def445af1e91)


- Una vez obtenida la segunda key **822c73956184f694993bede3eb39f959** paso a intentar escalar privilegios

## Escala de privilegios

- Buscando formas de escalar privilegios encontré archivos con permisos SUID **( lo cual indica que estos archivos son ejecutados con permisos root )**

- ![image](https://github.com/user-attachments/assets/5eebaad7-387e-4b92-978b-959820541f25)

## GFTOBins 

-[GFTOBins](https://gtfobins.github.io/)

-**GFTOBins** contiene informacion acerca de como aprovecharse de algunos programas con permisos mal configurados, en este caso **nmap**


![image](https://github.com/user-attachments/assets/d8f56c2f-18d2-4397-a042-70f15785399b)


-Ejecutando la siguientes lineas de codigo es posible escalar privilegios a esta maquina 


```bash
nmap --interactive
```


```bash
!sh
```


```bash
whoami && id 
```


![image](https://github.com/user-attachments/assets/f49ac76d-8e9f-4098-bfaa-91a9a48fd84e)


- De esta manera consiguiendo la tercera y ultima key de este CTF (Capture The Flag) **04787ddef27c3dee1ee161b21670b4e4**


## Extra


- Durante la etapa de enumeración, se identificó un servidor web. Se intentó modificar los archivos del sitio web, logrando completar el **CTF** y declarar el **"Hackeo completo"** de la máquina.


![image](https://github.com/user-attachments/assets/767b1b9f-681e-4d29-a5d0-64d78f04586a)









