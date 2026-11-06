# Instalación y configuración de un servidor FTP con vsftpd en una Raspberry Pi para una red LAN

**Materia:** Telecomunicaciones  
**Trabajo integrador**  
**Integrantes:** Alladio - Conejero - Corti - Ferrazzuolo - Manfredini

---

## Índice

1. [Introducción](#1-introducción)
2. [Objetivo](#2-objetivo)
3. [Marco teórico](#3-marco-teórico)
4. [Materiales y datos de la red](#4-materiales-y-datos-de-la-red)
5. [Instalación del servidor FTP](#5-instalación-del-servidor-ftp)
   - [5.1 Actualización del sistema](#51-actualización-del-sistema)
   - [5.2 Instalación de vsftpd](#52-instalación-de-vsftpd)
6. [Configuración del servicio FTP](#6-configuración-del-servicio-ftp)
   - [6.1 Copia de seguridad](#61-copia-de-seguridad-del-archivo-de-configuración)
   - [6.2 Edición del archivo de configuración](#62-edición-del-archivo-de-configuración)
7. [Creación del directorio FTP y configuración de permisos](#7-creación-del-directorio-ftp-y-configuración-de-permisos)
   - [7.1 Creación de la estructura de carpetas](#71-creación-de-la-estructura-de-carpetas)
   - [7.2 Configuración de permisos](#72-configuración-de-permisos)
8. [Habilitación y puesta en marcha del servicio](#8-habilitación-y-puesta-en-marcha-del-servicio)
9. [Pruebas de funcionamiento](#9-pruebas-de-funcionamiento)
   - [9.1 Conexión desde FileZilla](#91-conexión-desde-filezilla)
   - [9.2 Subida de archivos](#92-subida-de-archivos-al-servidor)
   - [9.3 Descarga de archivos](#93-descarga-de-archivos-desde-el-servidor)
   - [9.4 Resultado de las pruebas](#94-resultado-de-las-pruebas)
10. [Conclusión](#10-conclusión)

---

## 1. Introducción

En esta etapa del trabajo integrador se configuró la Raspberry Pi utilizada en las actividades anteriores como servidor FTP de la red local del grupo, mediante el software **vsftpd**.

En las entregas anteriores se preparó la Raspberry Pi con Raspberry Pi OS Lite, se habilitó el acceso remoto mediante SSH y se configuró el servicio DHCP para asignar direcciones IP automáticamente a los dispositivos de la red.

En esta actividad se incorporó un nuevo servicio para compartir y transferir archivos entre las notebooks de los integrantes y la Raspberry Pi mediante el protocolo FTP (*File Transfer Protocol*).

La red local del grupo 4 utiliza el segmento `192.168.4.0/24`. La Raspberry Pi tiene asignada la dirección IP fija `192.168.4.1`. Las notebooks se conectan al switch y acceden al servidor a través de la red LAN.

Para realizar las transferencias se utilizó **FileZilla**, un programa que permite conectarse al servidor FTP mediante una interfaz gráfica.

## 2. Objetivo

Instalar y configurar un servidor FTP en la Raspberry Pi mediante vsftpd, para permitir que los dispositivos conectados a la red LAN del grupo puedan transferir archivos de manera centralizada.

**Objetivos específicos:**

- Instalar el paquete `vsftpd` en la Raspberry Pi.
- Comprender el funcionamiento básico del protocolo FTP.
- Configurar el acceso al servidor mediante usuario y contraseña.
- Habilitar el modo pasivo para las transferencias de datos.
- Definir el directorio destinado al almacenamiento de archivos.
- Establecer los permisos necesarios para permitir la subida y descarga de archivos.
- Configurar el inicio automático del servicio.
- Comprobar la conexión desde las notebooks mediante FileZilla.
- Verificar la subida y descarga de archivos.

## 3. Marco teórico

**FTP (*File Transfer Protocol*):** es un protocolo de nivel de aplicación que permite transferir archivos entre dispositivos conectados a una red TCP/IP. Utiliza una conexión de control y otra conexión para el intercambio de datos.

**Servidor FTP y cliente FTP:** el servidor es el equipo que ofrece el servicio y administra el acceso a los archivos. El cliente es el dispositivo o programa que se conecta al servidor para realizar operaciones, como subir o descargar archivos.

**Canal de control y canal de datos:** FTP utiliza dos conexiones diferenciadas:

- **Canal de control:** normalmente utiliza el puerto TCP 21. Se emplea para enviar comandos, autenticar al usuario y administrar la sesión.
- **Canal de datos:** se utiliza para transferir archivos y listados de directorios. Los puertos utilizados dependen del modo de conexión configurado.

**Modo activo:** el cliente establece la conexión de control con el servidor y el servidor inicia la conexión de datos hacia el cliente. Este funcionamiento puede generar inconvenientes cuando el cliente se encuentra detrás de un firewall o NAT.

**Modo pasivo:** el cliente inicia tanto la conexión de control como la conexión de datos hacia el servidor. Esto suele facilitar la conexión cuando existen firewalls en el lado del cliente. En esta actividad se configuró un rango de puertos pasivos entre `10000` y `10100`.

**vsftpd (*Very Secure FTP Daemon*):** es un servidor FTP para sistemas Linux. Se caracteriza por ser liviano y ofrecer distintas opciones para administrar usuarios, permisos y conexiones.

**FileZilla:** es un programa cliente FTP que permite conectarse a servidores y transferir archivos mediante una interfaz gráfica. Su ventana presenta un panel para los archivos locales y otro para los archivos remotos.

**Autenticación:** es el mecanismo mediante el cual el servidor verifica la identidad del usuario que intenta conectarse. En esta configuración se deshabilitó el acceso anónimo y se habilitó el acceso de usuarios locales mediante sus credenciales.

**Chroot:** es un mecanismo que permite restringir el entorno de archivos al que puede acceder un usuario. En esta configuración se utiliza para limitar la navegación del usuario FTP a su directorio asignado.

**Permisos de Linux:** determinan qué usuarios pueden leer, escribir o ejecutar archivos y directorios. Su configuración permite controlar dónde se pueden almacenar los archivos transferidos.

**Consideración de seguridad:** FTP tradicional no cifra las credenciales ni el contenido transferido. Por eso, esta implementación debe utilizarse únicamente en una red de confianza y con fines de práctica. Para transferir información sensible conviene utilizar alternativas cifradas, como SFTP o FTPS.

## 4. Materiales y datos de la red

**Materiales y herramientas utilizados:**

- Raspberry Pi con Raspberry Pi OS Lite.
- Switch del grupo para interconectar los dispositivos.
- Notebooks de los integrantes, utilizadas como clientes FTP.
- Cables Ethernet (UTP).
- Conexión a Internet temporal para actualizar el sistema e instalar el paquete.
- Software vsftpd para implementar el servidor FTP.
- FileZilla para realizar las conexiones y transferencias.
- Acceso SSH para administrar remotamente la Raspberry Pi.

**Parámetros de la implementación:**

| Elemento | Dato |
|---|---|
| Modelo de Raspberry Pi | Raspberry Pi 3 Model B Rev 1.2 |
| Sistema operativo | Raspberry Pi OS Lite |
| Nombre del equipo | `RP-003` |
| Red LAN del grupo | `192.168.4.0/24` |
| Dirección IP de la Raspberry Pi | `192.168.4.1` |
| Servidor FTP | `vsftpd` |
| Puerto de control | TCP `21` |
| Modo de transferencia | Pasivo |
| Rango de puertos pasivos | `10000-10100` |
| Cliente FTP | FileZilla |
| Usuario utilizado | `grupo4` |

## 5. Instalación del servidor FTP

### 5.1 Actualización del sistema

Antes de instalar el servidor FTP, se actualizaron las listas de paquetes y los programas del sistema operativo para disponer de las actualizaciones disponibles en los repositorios.

Se ejecutó el siguiente comando:

```bash
sudo apt update && sudo apt upgrade -y
```

**Explicación de los comandos:**

- `sudo`: ejecuta el comando con permisos de administrador.
- `apt`: es el gestor de paquetes utilizado en Raspberry Pi OS.
- `update`: actualiza la lista de paquetes disponibles.
- `upgrade`: instala las actualizaciones disponibles para los paquetes instalados.
- `&&`: ejecuta el segundo comando solamente si el primero finaliza correctamente.
- `-y`: confirma automáticamente las preguntas de instalación.

### 5.2 Instalación de vsftpd

Una vez actualizado el sistema, se instaló el servidor FTP mediante el siguiente comando:

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

`systemctl` permite administrar los servicios del sistema y `status` muestra su estado actual. Si aparece `active (running)`, significa que el servicio se encuentra en ejecución.

## 6. Configuración del servicio FTP

### 6.1 Copia de seguridad del archivo de configuración

Antes de realizar modificaciones, se creó una copia de seguridad del archivo original:

```bash
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
```

- `cp`: permite copiar archivos.
- `/etc/vsftpd.conf`: es el archivo original de configuración del servidor FTP.
- `/etc/vsftpd.conf.bak`: es el archivo de respaldo, que permite recuperar la configuración anterior si ocurre algún problema.

### 6.2 Edición del archivo de configuración

Se abrió el archivo principal de vsftpd utilizando el editor de texto nano:

```bash
sudo nano /etc/vsftpd.conf
```

En el archivo se verificaron o modificaron las siguientes directivas:

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

**Descripción de las directivas:**

| Directiva | Valor | Función |
|---|---|---|
| `anonymous_enable` | `NO` | Deshabilita las conexiones anónimas. |
| `local_enable` | `YES` | Permite el acceso de usuarios locales del sistema. |
| `write_enable` | `YES` | Habilita las operaciones de escritura, necesarias para subir archivos. |
| `local_umask` | `022` | Define la máscara de permisos predeterminada para los archivos creados. |
| `chroot_local_user` | `YES` | Restringe a los usuarios locales al entorno de directorios configurado. |
| `allow_writeable_chroot` | `YES` | Permite que el directorio raíz del entorno chroot sea escribible, si así estuviera configurado. En esta implementación, el directorio raíz se mantiene sin permiso de escritura. |
| `pasv_enable` | `YES` | Habilita el modo pasivo. |
| `pasv_min_port` | `10000` | Define el primer puerto del rango pasivo. |
| `pasv_max_port` | `10100` | Define el último puerto del rango pasivo. |
| `user_sub_token` | `$USER` | Define el nombre de usuario como variable de sustitución en las rutas donde vsftpd admite esta opción. |
| `local_root` | `/home/$USER/ftp` | Define el directorio local al que se dirige el usuario al iniciar sesión. |

Una vez realizadas las modificaciones, se guardó el archivo y se cerró nano.

## 7. Creación del directorio FTP y configuración de permisos

### 7.1 Creación de la estructura de carpetas

Para almacenar los archivos transferidos se creó un directorio FTP dentro del directorio personal del usuario `grupo4`.

Se utilizó el siguiente comando:

```bash
mkdir -p /home/grupo4/ftp/uploads
```

- `mkdir`: crea directorios.
- `-p`: permite crear también los directorios intermedios que todavía no existen.
- `/home/grupo4/ftp/uploads`: indica la ruta de la carpeta destinada a recibir los archivos.

La estructura resultante es:

```text
/home/grupo4/
└── ftp/
    └── uploads/
```

### 7.2 Configuración de permisos

Se modificaron los permisos y propietarios de los directorios para cumplir con los requisitos de vsftpd al utilizar `chroot_local_user`.

Se ejecutaron los siguientes comandos:

```bash
sudo chown nobody:nogroup /home/grupo4/ftp
sudo chmod a-w /home/grupo4/ftp
sudo chown grupo4:grupo4 /home/grupo4/ftp/uploads
```

**Explicación de los comandos:**

- `chown nobody:nogroup /home/grupo4/ftp`: cambia el propietario y el grupo del directorio `ftp` a `nobody` y `nogroup`.
- `chmod a-w /home/grupo4/ftp`: quita los permisos de escritura para todos los usuarios en el directorio raíz FTP.
- `chown grupo4:grupo4 /home/grupo4/ftp/uploads`: asigna la propiedad de la carpeta `uploads` al usuario `grupo4`.

Esta configuración mantiene el directorio raíz del entorno FTP sin permisos de escritura y permite que el usuario `grupo4` trabaje dentro de la carpeta `uploads`.

De esta manera, el usuario queda restringido al entorno FTP y puede realizar transferencias dentro de la carpeta habilitada.

## 8. Habilitación y puesta en marcha del servicio

Una vez finalizada la configuración, se reinició el servicio para aplicar los cambios realizados en el archivo de configuración:

```bash
sudo systemctl restart vsftpd
```

La opción `restart` detiene el servicio y vuelve a iniciarlo.

Luego se habilitó el inicio automático del servidor:

```bash
sudo systemctl enable vsftpd
```

La opción `enable` configura el servicio para que se inicie automáticamente cada vez que arranca la Raspberry Pi.

De esta forma, el servidor FTP queda preparado para iniciarse sin necesidad de ejecutar el comando manualmente después de cada reinicio del equipo.

## 9. Pruebas de funcionamiento

Para comprobar el funcionamiento del servidor FTP, se realizaron pruebas desde una notebook conectada al mismo switch que la Raspberry Pi. Se utilizó FileZilla como cliente.

### 9.1 Conexión desde FileZilla

Primero se abrió FileZilla y se completaron los datos de conexión en la barra de conexión rápida.

Los parámetros utilizados fueron:

| Campo | Valor |
|---|---|
| Host | `192.168.4.1` |
| Usuario | `grupo4` |
| Contraseña | Contraseña del usuario `grupo4` |
| Puerto | `21` |

Una vez completados los campos, se hizo clic en **Conexión rápida**.

Cuando la conexión fue exitosa, FileZilla mostró el contenido del directorio remoto en el panel derecho. Como el usuario está restringido mediante `chroot`, la ruta se presenta desde la raíz de su entorno FTP. La carpeta de destino para las transferencias es `uploads`.

El panel izquierdo muestra los archivos de la notebook, mientras que el panel derecho muestra los archivos disponibles en el servidor.

### 9.2 Subida de archivos al servidor

Para comprobar que el servidor permite recibir archivos, se seleccionó un archivo desde el panel izquierdo de FileZilla y se arrastró hacia la carpeta `uploads` del panel derecho.

FileZilla mostró el progreso de la transferencia en la parte inferior de la ventana. Una vez finalizada la operación, el archivo apareció dentro del directorio remoto.

También se verificó desde la Raspberry Pi que el archivo se hubiera recibido correctamente:

```bash
ls -l /home/grupo4/ftp/uploads/
```

- `ls`: muestra el contenido de un directorio.
- `-l`: presenta la información en formato detallado, incluyendo permisos, propietario, tamaño y fecha de modificación.

### 9.3 Descarga de archivos desde el servidor

Finalmente, se comprobó el funcionamiento de la descarga.

Desde FileZilla se seleccionó un archivo de la carpeta `uploads`, ubicada en el panel derecho, y se arrastró hacia el panel izquierdo. También se puede realizar esta operación haciendo clic derecho sobre el archivo y seleccionando **Descargar**.

El archivo quedó almacenado en el directorio local seleccionado de la notebook.

### 9.4 Resultado de las pruebas

Las pruebas realizadas permitieron comprobar que:

- La notebook pudo conectarse al servidor FTP mediante FileZilla.
- La autenticación mediante usuario y contraseña funcionó correctamente.
- Se pudo acceder al directorio remoto.
- Se pudieron subir archivos desde la notebook hacia la Raspberry Pi.
- Se pudieron descargar archivos desde la Raspberry Pi hacia la notebook.

Por lo tanto, se verificó el funcionamiento del servidor FTP dentro de la red LAN del grupo.

## 10. Conclusión

En esta etapa del trabajo integrador se instaló y configuró vsftpd en la Raspberry Pi para convertirla en un servidor FTP de la red LAN del grupo.

Durante la actividad se estudió el funcionamiento del protocolo FTP, la diferencia entre los modos activo y pasivo y la función de los canales de control y de datos.

También se trabajó con la autenticación de usuarios, los permisos del sistema de archivos de Linux y el mecanismo `chroot`, que permite restringir el acceso de los usuarios a un directorio determinado.

Mediante FileZilla se comprobó que las notebooks conectadas al switch pueden establecer una conexión con la Raspberry Pi, subir archivos al servidor y descargarlos correctamente.

Esta actividad permitió incorporar un nuevo servicio a la infraestructura de red desarrollada en las entregas anteriores. De esta manera, la Raspberry Pi amplía sus funciones dentro de la LAN y ofrece un espacio centralizado para el intercambio de archivos.

En conclusión, se logró implementar y probar un sistema de transferencia de archivos dentro de la red local, relacionando los conceptos teóricos con su configuración y utilización práctica.
