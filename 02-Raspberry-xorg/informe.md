# Configuración del servidor gráfico X11 y reenvío remoto sobre SSH

**Materia:** Telecomunicaciones  
**Trabajo Integrador**  
**Integrantes:** Alladio - Conejero - Corti - Ferrazzuolo - Manfredini

---

## Índice

1. [Introducción](#1-introducción)
2. [Objetivo del informe](#2-objetivo-del-informe)
3. [Desarrollo teórico](#3-desarrollo-teórico)
4. [Configuración en la Raspberry Pi](#4-configuración-en-la-raspberry-pi)
5. [Configuración del servidor X11 en el cliente](#5-configuración-del-servidor-x11-en-el-cliente)
6. [Conexión remota mediante SSH con soporte X11](#6-conexión-remota-mediante-ssh-con-soporte-x11)
7. [Explicación de los comandos](#7-explicación-de-los-comandos)
8. [Pruebas de funcionamiento](#8-pruebas-de-funcionamiento)
9. [Conclusión](#9-conclusión)
10. [Bibliografía y fuentes consultadas](#10-bibliografía-y-fuentes-consultadas)

---

## 1. Introducción

En esta actividad continuamos trabajando con la Raspberry Pi que configuramos en la entrega anterior. Esta vez habilitamos el reenvío gráfico X11 mediante SSH. Esto nos permite ejecutar aplicaciones gráficas en la Raspberry Pi y ver sus ventanas en nuestra computadora, sin tener que instalar un entorno de escritorio completo en la placa.

---

## 2. Objetivo del informe

- Instalar las herramientas necesarias para ejecutar aplicaciones X11 en Raspberry Pi OS Lite.
- Habilitar y comprobar el reenvío X11 en el servidor SSH.
- Configurar un servidor X11 en la computadora con Windows utilizando XLaunch.
- Conectarnos a la Raspberry Pi mediante SSH y ejecutar aplicaciones gráficas de forma remota.
- Comprobar el funcionamiento con las aplicaciones `xclock` y `xeyes`.

---

## 3. Desarrollo teórico

### 3.1. X11

X11, también conocido como X Window System, es un sistema que permite ejecutar aplicaciones gráficas y mostrar sus ventanas en una pantalla. En esta arquitectura, el servidor X se encarga de mostrar las ventanas y recibir las acciones del teclado y del mouse, mientras que las aplicaciones envían las instrucciones gráficas.

En nuestro caso:

- La **Raspberry Pi** ejecuta las aplicaciones gráficas, por lo que actúa como cliente X.
- La **computadora con Windows** muestra las ventanas mediante un servidor X, como XLaunch.

```text
Raspberry Pi (aplicación gráfica) → Túnel SSH con X11 → Computadora (servidor X)
```

### 3.2. X11 Forwarding sobre SSH

X11 Forwarding es una función de SSH que permite transportar los datos de las aplicaciones gráficas a través de una conexión cifrada. Así podemos utilizar programas gráficos de la Raspberry Pi y ver sus ventanas en la computadora cliente, sin exponer directamente el tráfico X11 a la red.

### 3.3. Uso de Raspberry Pi OS Lite

Raspberry Pi OS Lite no incluye un entorno de escritorio completo. Esto reduce el consumo de memoria y procesador, algo útil cuando la Raspberry Pi se utiliza como servidor. Con X11 Forwarding podemos abrir únicamente las aplicaciones gráficas que necesitamos, sin instalar un escritorio completo en la placa.

---

## 4. Configuración en la Raspberry Pi

### 4.1. Instalación de paquetes y herramientas de prueba

Una vez conectados por SSH a la Raspberry Pi, instalamos `xauth` y las aplicaciones de prueba de X11:

```bash
sudo apt update
sudo apt install -y xauth x11-apps
```

`xauth` permite gestionar la autorización para las conexiones X11, mientras que `x11-apps` incluye programas gráficos que podemos utilizar para comprobar el funcionamiento.

### 4.2. Habilitación de X11 Forwarding en SSH

Abrimos el archivo de configuración del servidor SSH:

```bash
sudo nano /etc/ssh/sshd_config
```

Verificamos que las siguientes directivas estén habilitadas:

```text
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost yes
```

Guardamos los cambios y reiniciamos el servicio SSH para aplicarlos:

```bash
sudo systemctl restart ssh
```

---

## 5. Configuración del servidor X11 en el cliente

Para mostrar las ventanas gráficas en Windows utilizamos **XLaunch**, incluido con VcXsrv. Configuramos el servidor X de la siguiente manera:

1. Seleccionamos **Multiple windows**, para que cada aplicación remota aparezca en una ventana independiente.
2. En la pantalla de inicio del cliente seleccionamos **Start no client**, porque iniciaremos las aplicaciones desde la terminal SSH.
3. En las opciones adicionales activamos **Disable access control** o **No Access Control**, según la versión del programa utilizada.
4. Finalizamos el asistente y dejamos el servidor X11 funcionando en segundo plano.

---

## 6. Conexión remota mediante SSH con soporte X11

### 6.1. Configuración de la variable DISPLAY en Windows

Antes de iniciar la conexión SSH, configuramos la variable `DISPLAY` desde PowerShell para indicar la pantalla del servidor X local:

```powershell
$env:DISPLAY="localhost:0.0"
```

### 6.2. Inicio de la sesión SSH

Nos conectamos a la Raspberry Pi utilizando la opción `-Y`, que habilita el reenvío confiable de X11:

```bash
ssh -Y grupo4@192.168.60.182
```

Una vez dentro de la Raspberry Pi, consultamos la variable `DISPLAY`:

```bash
echo $DISPLAY
```

El resultado esperado es similar al siguiente:

```text
localhost:10.0
```

Este valor indica que SSH configuró la pantalla virtual que utilizarán las aplicaciones gráficas para enviar sus ventanas al equipo cliente.

---

## 7. Explicación de los comandos

### `sudo apt update`

Actualiza la lista de paquetes disponibles en los repositorios para que el sistema pueda consultar las versiones más recientes.

### `sudo apt install -y xauth x11-apps`

- `sudo`: ejecuta el comando con permisos de administrador.
- `apt install`: instala los paquetes indicados.
- `-y`: confirma automáticamente las preguntas de instalación.
- `xauth`: administra las claves de autorización de X11.
- `x11-apps`: instala aplicaciones gráficas de prueba.

### `sudo nano /etc/ssh/sshd_config`

Abre el archivo de configuración del servidor SSH con el editor de texto `nano`. Allí se habilitan las opciones relacionadas con el reenvío X11.

### `sudo systemctl restart ssh`

Reinicia el servicio SSH para que se apliquen los cambios realizados en su configuración.

### `$env:DISPLAY="localhost:0.0"`

Comando de PowerShell que establece la variable `DISPLAY` en la sesión local de Windows. Esta variable indica dónde se encuentra la pantalla gráfica.

### `ssh -Y usuario@IP`

- `ssh`: inicia una conexión remota.
- `-Y`: habilita el reenvío confiable de X11.
- `usuario`: nombre de usuario de la Raspberry Pi.
- `IP`: dirección de la Raspberry Pi en la red.

### `echo $DISPLAY`

Muestra el valor de la variable `DISPLAY` en la Raspberry Pi. Un valor como `localhost:10.0` indica que SSH preparó el destino para el reenvío gráfico.

### `xclock` y `xeyes`

- `xclock`: abre una ventana gráfica con un reloj.
- `xeyes`: abre una ventana con dos ojos que siguen el movimiento del cursor.

Estos programas sirven para comprobar que las aplicaciones gráficas de la Raspberry Pi pueden mostrarse en Windows.

---

## 8. Pruebas de funcionamiento

Con XLaunch activo en Windows y la conexión SSH abierta, ejecutamos en la terminal de la Raspberry Pi:

```bash
xclock
```

Después ejecutamos:

```bash
xeyes
```

**Resultado:** las ventanas de las aplicaciones se mostraron en el escritorio de Windows. De esta manera comprobamos que los programas se ejecutan en la Raspberry Pi, mientras que las ventanas se visualizan en la computadora cliente mediante el reenvío X11 por SSH.

---

## 9. Conclusión

Habilitamos el reenvío gráfico X11 sobre SSH y comprobamos su funcionamiento mediante las aplicaciones `xclock` y `xeyes`. Esta configuración permite ejecutar aplicaciones gráficas en la Raspberry Pi sin instalar un entorno de escritorio completo. Además, al utilizar SSH, la comunicación entre la Raspberry Pi y la computadora cliente viaja por una conexión cifrada.

---

## 10. Bibliografía y fuentes consultadas

- Raspberry Pi, documentación oficial: https://www.raspberrypi.com/documentation/
- Debian Wiki, X11 Forwarding: https://wiki.debian.org/X11Forwarding
- Chris Titus Tech, *Remote GUI Applications via SSH X11 Forwarding*.
- Unix & Linux Stack Exchange, consultas sobre el reenvío de X11 mediante SSH.
