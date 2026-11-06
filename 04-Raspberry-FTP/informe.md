# Instalación y configuración de un servidor FTP con vsftpd en una Raspberry Pi para una red LAN

**Materia:** Telecomunicaciones

**Trabajo Integrador**

**Integrantes:** Alladio - Conejero - Corti - Ferrazzuolo - Manfredini

---

## Índice

- [Instalación y configuración de un servidor FTP con vsftpd en una Raspberry Pi para una red LAN](#instalación-y-configuración-de-un-servidor-ftp-con-vsftpd-en-una-raspberry-pi-para-una-red-lan)
  - [Índice](#índice)
  - [1. Introducción](#1-introducción)
  - [2. Objetivo](#2-objetivo)
  - [3. Marco teórico](#3-marco-teórico)
  - [4. Materiales utilizados](#4-materiales-utilizados)
  - [6. Instalación del servidor FTP](#6-instalación-del-servidor-ftp)
    - [6.1 Actualización del sistema](#61-actualización-del-sistema)
    - [6.2 Instalación de vsftpd](#62-instalación-de-vsftpd)
    - [Evidencia](#evidencia)
  - [7. Configuración del servicio FTP](#7-configuración-del-servicio-ftp)
    - [7.1 Copia de seguridad del archivo de configuración](#71-copia-de-seguridad-del-archivo-de-configuración)
    - [7.2 Edición del archivo de configuración](#72-edición-del-archivo-de-configuración)
    - [Evidencia](#evidencia-1)
  - [8. Creación del directorio FTP y configuración de permisos](#8-creación-del-directorio-ftp-y-configuración-de-permisos)
    - [8.1 Creación de la estructura de carpetas](#81-creación-de-la-estructura-de-carpetas)
    - [8.2 Configuración de permisos](#82-configuración-de-permisos)
    - [Evidencia](#evidencia-2)
  - [9. Habilitación y puesta en marcha del servicio](#9-habilitación-y-puesta-en-marcha-del-servicio)
  - [10. Pruebas de funcionamiento](#10-pruebas-de-funcionamiento)
    - [10.1 Conexión desde FileZilla](#101-conexión-desde-filezilla)
    - [10.2 Subida de archivos al servidor](#102-subida-de-archivos-al-servidor)
    - [10.3 Descarga de archivos desde el servidor](#103-descarga-de-archivos-desde-el-servidor)
    - [10.4 Resultado de las pruebas](#104-resultado-de-las-pruebas)
    - [Evidencia](#evidencia-3)
  - [11. Conclusión](#11-conclusión)

---

## 1. Introducción

En esta nueva etapa del trabajo integrador se configuró la Raspberry Pi de las actividades anteriores como **servidor FTP** de la red local del grupo, utilizando el software vsftpd.

En las entregas anteriores se preparó la Raspberry Pi con Raspberry Pi OS Lite, se habilitó el acceso remoto mediante SSH y se configuró como servidor DHCP para asignar direcciones IP automáticame[...]

En esta actividad se incorporó un nuevo servicio que permite compartir y transferir archivos entre las notebooks de los integrantes y la Raspberry Pi, utilizando el protocolo FTP (File Transfer Proto[...]

La red local del grupo 4 utiliza el segmento `192.168.4.0/24`. La Raspberry Pi mantiene su dirección IP fija `192.168.4.1`, mientras que las notebooks se conectan al switch y pueden acceder al servid[...]

Para realizar las transferencias se utilizó FileZilla, un programa que permite conectarse al servidor FTP mediante una interfaz gráfica.

---

## 2. Objetivo

Instalar y configurar un servidor FTP en la Raspberry Pi mediante vsftpd, permitiendo que los dispositivos conectados a la red LAN del grupo puedan transferir archivos de manera centralizada.

Objetivos específicos:

- Instalar el paquete `vsftpd` en la Raspberry Pi.
- Comprender el funcionamiento del protocolo FTP.
- Configurar el acceso al servidor mediante usuario y contraseña.
- Habilitar el modo pasivo para las transferencias de datos.
- Configurar el directorio donde se almacenarán los archivos.
- Establecer los permisos necesarios para permitir la subida y descarga de archivos.
- Habilitar el inicio automático del servicio.
- Comprobar la conexión desde las notebooks mediante FileZilla.
- Verificar que sea posible subir y descargar archivos correctamente.

---

## 3. Marco teórico

**FTP (File Transfer Protocol):** es un protocolo de nivel de aplicación del modelo OSI que permite transferir archivos entre dos dispositivos conectados a una red TCP/IP. Funciona mediante un modelo[...]

**Servidor FTP y cliente FTP:** el servidor es el equipo que almacena y administra los archivos disponibles para transferir. El cliente es el dispositivo que se conecta al servidor para realizar opera[...]

**Canal de control y canal de datos:** FTP utiliza dos canales separados:

- Canal de control: utiliza el puerto TCP 21 y permite enviar comandos, autenticar usuarios y administrar las conexiones.
- Canal de datos: se utiliza para transferir los archivos. En modo activo utiliza el puerto 20 del servidor, mientras que en modo pasivo utiliza puertos variables dentro de un rango configurado.

**Modo activo:** el cliente establece la conexión de control con el servidor, pero es el servidor quien inicia la conexión de datos hacia el cliente. Puede generar problemas cuando existen firewalls[...]

**Modo pasivo:** el cliente inicia tanto la conexión de control como la conexión de datos hacia el servidor. Es más compatible con redes modernas y evita algunos problemas relacionados con firewall[...]

En esta actividad se utilizó el modo pasivo, configurando un rango de puertos entre `10000` y `10100`.

**vsftpd (Very Secure FTP Daemon):** es un servidor FTP desarrollado para sistemas Linux. Se caracteriza por ser liviano, consumir pocos recursos y ofrecer distintas opciones de configuración y segur[...]

**FileZilla:** es un programa cliente FTP de código abierto que permite conectarse a servidores y transferir archivos mediante una interfaz gráfica. Presenta dos paneles: uno para los archivos de la[...]

**Autenticación:** mecanismo mediante el cual el servidor verifica la identidad del usuario que intenta conectarse. En nuestra configuración se deshabilitó el acceso anónimo y se habilitó el acce[...]

**Chroot:** mecanismo que permite restringir a un usuario a un directorio determinado del sistema de archivos. De esta manera, el usuario FTP no puede navegar libremente por el resto de los directorio[...]

**Permisos de Linux:** determinan qué usuarios pueden leer, escribir o ejecutar archivos y directorios. Son importantes para controlar dónde se pueden almacenar los archivos transferidos.

---

## 4. Materiales utilizados

- Raspberry Pi con Raspberry Pi OS Lite.
- Switch del grupo, utilizado para interconectar los dispositivos.
- Notebooks de los integrantes, utilizadas como clientes FTP.
- Cables Ethernet (UTP).
- Conexión a Internet temporal para actualizar el sistema e instalar el paquete.
- Software vsftpd para implementar el servidor FTP.
- FileZilla para realizar las conexiones y transferencias.
- Acceso SSH para administrar remotamente la Raspberry Pi.

| Elemento | Dato |
|---|---|
| Modelo de Raspberry Pi | Raspberry Pi 3 Model B Rev 1.2 |
| Sistema operativo | Raspberry Pi OS Lite |
| Nombre del equipo | `RP-003` |
| Red LAN del grupo | `192.168.4.0/24` |
| Dirección IP de la Raspberry Pi | `192.168.4.1` |
| Servidor FTP | `vsftpd` |
| Puerto de control | `21` |
| Modo de transferencia | Pasivo |
| Rango de puertos pasivos | `10000-10100` |
| Cliente FTP | FileZilla |

---

## 6. Instalación del servidor FTP

### 6.1 Actualización del sistema

Antes de instalar el servidor FTP, se actualizaron las listas de paquetes y los programas del sistema operativo para disponer de las versiones más recientes disponibles en los repositorios.

Se ejecutó el siguiente comando:

```bash
sudo apt update && sudo apt upgrade -y
```

**Explicación de los comandos:**

- `sudo`: ejecuta el comando con permisos de administrador.
- `apt`: gestor de paquetes utilizado en Raspberry Pi OS.
- `update`: actualiza la lista de paquetes disponibles.
- `upgrade`: instala las actualizaciones disponibles de los paquetes instalados.
- `&&`: permite ejecutar un segundo comando solamente si el primero finaliza correctamente.
- `-y`: confirma automáticamente las preguntas de instalación.

### 6.2 Instalación de vsftpd

Una vez actualizado el sistema, se instaló el servidor FTP mediante:

```bash
sudo apt install vsftpd -y
```

**Explicación del comando:**

- `apt install`: descarga e instala el paquete indicado.
- `vsftpd`: nombre del software que implementa el servidor FTP.
- `-y`: confirma automáticamente la instalación.

Una vez finalizada la instalación, se comprobó el estado del servicio:

```bash
sudo systemctl status vsftpd
```

**Explicación del comando:**

- `systemctl`: herramienta utilizada para administrar los servicios del sistema.
- `status`: muestra el estado actual del servicio.

Si aparece `active (running)`, significa que el servidor se encuentra funcionando.


---


## 7. Configuración del servicio FTP

### 7.1 Copia de seguridad del archivo de configuración

Antes de realizar modificaciones, se creó una copia de seguridad del archivo original de configuración:

```bash
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
```

**Explicación del comando:**

- `cp`: permite copiar archivos.
- `/etc/vsftpd.conf`: archivo original de configuración del servidor FTP.
- `/etc/vsftpd.conf.bak`: archivo de respaldo que permite recuperar la configuración anterior si ocurre algún problema.

### 7.2 Edición del archivo de configuración

Se abrió el archivo principal de vsftpd utilizando el editor de texto nano:

```bash
sudo nano /etc/vsftpd.conf
```

Dentro del archivo se verificaron o modificaron las siguientes directivas:

```ini
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
chroot_local_user=YES
allow_writeable_chroot=YES
pasv_enable=YES
pasv_min_port=10000
pasv_max_port=10100
user_sub_token=$USER
local_root=/home/$USER/ftp
```

**Explicación de las opciones:**

| Directiva | Valor | Descripción |
|---|---|---|
| `anonymous_enable` | NO | Deshabilita las conexiones anónimas. |
| `local_enable` | YES | Permite conectarse a los usuarios locales del sistema. |
| `write_enable` | YES | Habilita la escritura y la subida de archivos. |
| `local_umask` | 022 | Define los permisos predeterminados de los archivos creados. |
| `chroot_local_user` | YES | Restringe al usuario a su directorio correspondiente. |
| `allow_writeable_chroot` | YES | Permite utilizar el directorio raíz del usuario con la configuración de chroot. |
| `pasv_enable` | YES | Habilita el modo pasivo. |
| `pasv_min_port` | 10000 | Establece el primer puerto del rango pasivo. |
| `pasv_max_port` | 10100 | Establece el último puerto del rango pasivo. |
| `user_sub_token` | $USER | Utiliza el nombre del usuario conectado como variable. |
| `local_root` | `/home/$USER/ftp` | Define el directorio al que accede el usuario al conectarse. |

**Explicación adicional:**

- `anonymous_enable=NO`: evita que cualquier persona pueda conectarse sin proporcionar credenciales.
- `local_enable=YES`: habilita la autenticación de usuarios existentes en Raspberry Pi OS.
- `write_enable=YES`: permite realizar operaciones de escritura, como subir archivos y crear carpetas.
- `local_umask=022`: establece los permisos predeterminados para los archivos transferidos.
- `chroot_local_user=YES`: limita la navegación del usuario al directorio FTP asignado.
- `pasv_enable=YES`: permite utilizar el modo pasivo para las transferencias.
- `pasv_min_port` y `pasv_max_port`: delimitan el rango de puertos utilizados para establecer las conexiones de datos.
- `local_root`: establece la ubicación inicial del usuario cuando se conecta al servidor.

Una vez realizadas las modificaciones, se guardó el archivo y se cerró nano.


---

## 8. Creación del directorio FTP y configuración de permisos

### 8.1 Creación de la estructura de carpetas

Para almacenar los archivos transferidos se creó un directorio FTP dentro del directorio personal del usuario `grupo4`.

Se utilizó el siguiente comando:

```bash
mkdir -p /home/grupo4/ftp/uploads
```

**Explicación del comando:**

- `mkdir`: crea directorios.
- `-p`: permite crear también los directorios intermedios que todavía no existen.
- `/home/grupo4/ftp/uploads`: ubicación donde se almacenarán los archivos enviados al servidor.

La estructura resultante es:

```text
/home/grupo4/
    └── ftp/
        └── uploads/
```

### 8.2 Configuración de permisos

Se modificaron los permisos y propietarios de los directorios para cumplir con los requisitos de seguridad de vsftpd al utilizar `chroot_local_user`.

Se ejecutaron los siguientes comandos:

```bash
sudo chown nobody:nogroup /home/grupo4/ftp
sudo chmod a-w /home/grupo4/ftp
sudo chown grupo4:grupo4 /home/grupo4/ftp/uploads
```

**Explicación de los comandos:**

- `chown nobody:nogroup /home/grupo4/ftp`: cambia el propietario y el grupo del directorio FTP a `nobody` y `nogroup`.
- `chmod a-w /home/grupo4/ftp`: quita los permisos de escritura para todos los usuarios en el directorio raíz FTP.
- `chown grupo4:grupo4 /home/grupo4/ftp/uploads`: asigna la propiedad de la carpeta `uploads` al usuario `grupo4`.

Esta configuración permite que el directorio principal no sea modificable, mientras que la carpeta `uploads` sí permite almacenar archivos.

De esta manera, el usuario queda restringido a su directorio FTP y puede realizar transferencias dentro de la carpeta habilitada.

---

## 9. Habilitación y puesta en marcha del servicio

Una vez finalizada la configuración, fue necesario reiniciar el servicio para aplicar los cambios realizados en el archivo de configuración.

Se ejecutó:

```bash
sudo systemctl restart vsftpd
```

**Explicación del comando:**

- `restart`: detiene el servicio y vuelve a iniciarlo, aplicando la configuración actualizada.

Luego se habilitó el inicio automático del servidor:

```bash
sudo systemctl enable vsftpd
```

**Explicación del comando:**

- `enable`: configura el servicio para que se inicie automáticamente cada vez que arranca la Raspberry Pi.

De esta forma, el servidor FTP queda preparado para funcionar sin necesidad de iniciarlo manualmente después de cada reinicio del equipo.

---

## 10. Pruebas de funcionamiento

Para comprobar que el servidor FTP funciona correctamente, se realizaron pruebas desde una notebook conectada al mismo switch que la Raspberry Pi.

Se utilizó FileZilla como cliente FTP.

### 10.1 Conexión desde FileZilla

Primero se abrió FileZilla y se completaron los datos de conexión en la barra de conexión rápida.

Los parámetros utilizados fueron:

| Campo | Valor |
|---|---|
| Host | `192.168.4.1` |
| Usuario | `grupo4` |
| Contraseña | Contraseña del usuario `grupo4` |
| Puerto | `21` |

Una vez completados los campos, se hizo clic en **Conexión rápida**.

FileZilla intentó establecer la conexión con la Raspberry Pi. Cuando la conexión fue exitosa, apareció en el panel derecho el contenido del directorio remoto:

```text
/home/grupo4/ftp/uploads
```

El panel izquierdo muestra los archivos de la notebook, mientras que el panel derecho muestra los archivos almacenados en el servidor.

### 10.2 Subida de archivos al servidor

Para comprobar que el servidor permite recibir archivos, se seleccionó un archivo desde el panel izquierdo de FileZilla y se arrastró hacia el panel derecho.

FileZilla mostró el progreso de la transferencia en la parte inferior de la ventana.

Una vez finalizada la operación, el archivo apareció dentro del directorio remoto.

También se verificó desde la Raspberry Pi que el archivo se hubiera recibido correctamente:

```bash
ls -l /home/grupo4/ftp/uploads/
```

**Explicación del comando:**

- `ls`: muestra el contenido de un directorio.
- `-l`: presenta la información en formato detallado, incluyendo permisos, propietario, tamaño y fecha de modificación.

### 10.3 Descarga de archivos desde el servidor

Finalmente, se comprobó el funcionamiento de la descarga.

Desde FileZilla se seleccionó un archivo del panel derecho, correspondiente al servidor remoto, y se arrastró hacia el panel izquierdo.

También se puede realizar esta operación haciendo clic derecho sobre el archivo y seleccionando la opción **Descargar**.

El archivo quedó almacenado en el directorio local seleccionado de la notebook.

### 10.4 Resultado de las pruebas

Las pruebas realizadas permitieron comprobar que:

- La notebook pudo conectarse al servidor FTP mediante FileZilla.
- La autenticación mediante usuario y contraseña funcionó correctamente.
- Se pudo acceder al directorio remoto.
- Se pudieron subir archivos desde la notebook hacia la Raspberry Pi.
- Se pudieron descargar archivos desde la Raspberry Pi hacia la notebook.

Por lo tanto, se verificó el correcto funcionamiento del servidor FTP dentro de la red LAN del grupo.


---

## 11. Conclusión

En esta etapa del trabajo integrador se instaló y configuró vsftpd en la Raspberry Pi para convertirla en un servidor FTP de la red LAN del grupo.

Durante la actividad se aprendió cómo funciona el protocolo FTP, la diferencia entre el modo activo y el modo pasivo, y la importancia de configurar correctamente los canales de control y de datos p[...]

También se trabajó con la configuración de usuarios y contraseñas, los permisos del sistema de archivos de Linux y el mecanismo chroot, que permite restringir el acceso de los usuarios a un direct[...]

Mediante FileZilla se comprobó que las notebooks conectadas al switch pueden establecer una conexión con la Raspberry Pi, subir archivos al servidor y descargarlos correctamente.

Esta actividad permitió incorporar un nuevo servicio a la infraestructura de red que se viene desarrollando desde las entregas anteriores. De esta manera, la Raspberry Pi no solamente cumple la funci[...]

En conclusión, se logró implementar un sistema de transferencia de archivos dentro de la LAN, comprendiendo tanto su funcionamiento teórico como su configuración y utilización práctica.

