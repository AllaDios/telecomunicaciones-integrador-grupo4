# Instalación y configuración de un servidor DHCP en una Raspberry Pi para una red LAN

**Materia:** Telecomunicaciones

**Trabajo Integrador**

**Integrantes:** Alladio - Conejero - Corti - Ferrazzuolo - Manfredini

---

## Índice

- [1. Introducción](#1-introducción)
- [2. Objetivo](#2-objetivo)
- [3. Marco teórico](#3-marco-teórico)
- [4. Materiales utilizados](#4-materiales-utilizados)
- [5. Diseño de la red](#5-diseño-de-la-red)
- [6. Instalación del servidor DHCP](#6-instalación-del-servidor-dhcp)
- [7. Configuración del servicio DHCP](#7-configuración-del-servicio-dhcp)
- [8. Validación de la configuración y habilitación del servicio](#8-validación-de-la-configuración-y-habilitación-del-servicio)
- [9. Dirección IP fija y puesta en marcha en la red LAN](#9-dirección-ip-fija-y-puesta-en-marcha-en-la-red-lan)
- [10. Pruebas de funcionamiento](#10-pruebas-de-funcionamiento)
- [11. Conclusión](#11-conclusión)

---

## 1. Introducción

En esta tercera etapa se configuró la Raspberry Pi de las actividades anteriores como **servidor DHCP** de una red local propia del grupo. La red LAN está formada por un switch al que se conectan tanto la Raspberry Pi como las notebooks de los integrantes. La Raspberry Pi asigna automáticamente la configuración de red a cada equipo que se conecta al switch.

El segmento de red del grupo 4 es `192.168.4.0/24`, es decir, la red `192.168.<número de grupo>.0` solicitada en la consigna.

---

## 2. Objetivo

Instalar y configurar un servidor DHCP en la Raspberry Pi, de modo que sea el encargado de asignar direcciones IP a los dispositivos de la red LAN del grupo (`192.168.4.0/24`), y documentar el diseño de la red mediante un diagrama realizado en Packet Tracer.

Objetivos específicos:

- Instalar el servidor DHCP `isc-dhcp-server`.
- Configurar la interfaz de red por la cual el servidor atenderá las solicitudes.
- Definir la subred, el rango de direcciones, la puerta de enlace y los servidores DNS que se entregarán a los clientes.
- Validar la sintaxis de la configuración y habilitar el servicio.
- Asignar una dirección IP fija a la Raspberry Pi dentro de la red LAN.
- Comprobar que las notebooks conectadas al switch reciben su configuración por DHCP.
- Representar la red en un diagrama de Packet Tracer.

---

## 3. Marco teórico

**DHCP (*Dynamic Host Configuration Protocol*):** protocolo de red que permite asignar de forma automática la configuración IP a los dispositivos de una red (dirección IP, máscara de subred, puerta de enlace y servidores DNS). Evita tener que configurar manualmente cada equipo y reduce el riesgo de conflictos por direcciones repetidas. Utiliza el protocolo UDP: el servidor escucha en el puerto 67 y el cliente en el puerto 68.

**Servidor DHCP y cliente DHCP:** el servidor es el equipo que administra un conjunto de direcciones y las entrega; el cliente es el dispositivo que solicita su configuración al conectarse a la red. En esta actividad el servidor es la Raspberry Pi y los clientes son las notebooks.

**Proceso DORA:** intercambio de cuatro mensajes entre cliente y servidor:

1. *Discover*: el cliente envía un mensaje de difusión (*broadcast*) buscando un servidor DHCP.
2. *Offer*: el servidor responde ofreciendo una dirección IP disponible.
3. *Request*: el cliente solicita formalmente la dirección ofrecida.
4. *Acknowledge*: el servidor confirma la asignación.

**Concesión (*lease*):** tiempo durante el cual un cliente puede usar la dirección asignada. Cuando se agota debe renovarla. Se configura con `default-lease-time` y `max-lease-time`.

**Ámbito o rango (*scope*):** conjunto de direcciones IP que el servidor puede entregar dentro de una subred. Las direcciones fuera del rango, como la del propio servidor, quedan disponibles para configuración fija.

**Subred y máscara:** la notación `192.168.4.0/24` indica una red cuyos primeros 24 bits identifican la red, equivalente a la máscara `255.255.255.0`. Admite hasta 254 equipos, con `192.168.4.255` como dirección de difusión.

**Puerta de enlace (*gateway*) y DNS:** la puerta de enlace es el equipo al que los clientes envían el tráfico destinado a otras redes. El servidor DNS traduce nombres de dominio a direcciones IP.

**Switch:** dispositivo de capa 2 que interconecta los equipos de una red local. En esta actividad no requiere configuración: solo permite que la Raspberry Pi y las notebooks formen la red LAN.

**`isc-dhcp-server`:** implementación del servidor DHCP desarrollada por el *Internet Systems Consortium*. Su configuración principal está en `/etc/dhcp/dhcpd.conf`, y la interfaz por la que atiende se define en `/etc/default/isc-dhcp-server`.

**NetworkManager y `nmcli`:** NetworkManager es el servicio que gestiona las conexiones de red en Raspberry Pi OS. `nmcli` es su herramienta de línea de comandos.

**Packet Tracer:** simulador de redes de Cisco que permite diseñar y probar topologías antes de implementarlas.

---

## 4. Materiales utilizados

- Raspberry Pi con Raspberry Pi OS Lite y SSH habilitado (Actividades 1 y 2).
- Switch del grupo, que forma la red LAN.
- Notebooks de los integrantes, utilizadas como clientes.
- Cables Ethernet (UTP).
- Conexión a Internet temporal para la instalación del paquete.
- Packet Tracer para el diagrama de red.

| Elemento | Dato |
|---|---|
| Modelo de Raspberry Pi | Raspberry Pi 3 Model B Rev 1.2 |
| Sistema operativo | Raspberry Pi OS Lite (basado en Debian 13 «trixie») |
| Nombre del equipo | `RP-003` |
| Usuario | `grupo4` |
| Paquete instalado | `isc-dhcp-server` 4.4.3-P1 |
| Red LAN del grupo | `192.168.4.0/24` |
| Dirección IP de la Raspberry Pi (LAN) | `192.168.4.1` |
| Rango asignado por DHCP | `192.168.4.100` a `192.168.4.200` |

---

## 5. Diseño de la red

La red del grupo se arma con el switch como elemento central:

```
   Raspberry Pi (192.168.4.1)  ──┐
                                 ├── SWITCH ── Notebooks (clientes DHCP)
   Notebooks                   ──┘
```

- La Raspberry Pi se conecta por su puerto Ethernet (`eth0`) al switch y brinda el servicio DHCP.
- Las notebooks se conectan al mismo switch y obtienen su configuración de forma automática.
- El switch no está conectado a ninguna otra red, de modo que el único servidor DHCP del segmento es la Raspberry Pi.

Como el servidor debe tener una dirección dentro de la red que administra, la Raspberry Pi usa una IP fija (`192.168.4.1`), mientras que el rango dinámico queda reservado para los clientes.

| Parámetro | Valor |
|---|---|
| Red | `192.168.4.0` |
| Máscara | `255.255.255.0` (/24) |
| Servidor DHCP / puerta de enlace | `192.168.4.1` |
| Rango dinámico | `192.168.4.100` – `192.168.4.200` |
| Difusión | `192.168.4.255` |
| DNS entregado | `8.8.8.8` y `8.8.4.4` |

### Diagrama en Packet Tracer

<!-- TODO: insertar el diagrama de Packet Tracer, por ejemplo: ![](./images/diagrama_packet_tracer.png) -->

El diagrama representa un **Server-PT** (equivalente de la Raspberry Pi) con IP `192.168.4.1/24` y el servicio DHCP activo, conectado a un **switch 2960** y a las notebooks (**Laptop-PT**) configuradas para obtener su dirección por DHCP.

---

## 6. Instalación del servidor DHCP

Antes de instalar, se comprobó el estado de la interfaz de red con:

```bash
ip a show eth0
nmcli -f ipv4.method,ipv4.addresses con show "Wired connection 1"
```

**Explicación de los comandos:**

- `ip a show eth0`: muestra la configuración de la interfaz `eth0`.
- `nmcli ... con show`: consulta el método de configuración IPv4 de la conexión (`auto` significa que obtiene la dirección por DHCP).

La interfaz estaba conectada a una red con acceso a Internet y con dirección asignada dinámicamente (`ipv4.method: auto`). Esto es necesario para descargar el paquete: la instalación se realizó **antes** de pasar a la red del switch, que no tiene salida a Internet.

Luego se actualizó la lista de paquetes y se instaló el servidor:

```bash
sudo apt update
sudo apt install -y isc-dhcp-server
```

**Explicación de los comandos:**

- `apt update`: actualiza la lista de paquetes disponibles en los repositorios.
- `apt install`: instala el paquete indicado, junto con sus dependencias (`isc-dhcp-common`, entre otras).
- `-y`: responde "sí" automáticamente a las confirmaciones.

Al finalizar la instalación el servicio intenta iniciarse y falla, ya que todavía no tiene configuración. Este comportamiento es normal y se resuelve en los pasos siguientes.

### Evidencia

![](./images/image1.png)

---

## 7. Configuración del servicio DHCP

### 7.1 Interfaz de red

Se indicó al servidor que atienda únicamente por la interfaz Ethernet:

```bash
sudo tee /etc/default/isc-dhcp-server > /dev/null <<'EOF'
INTERFACESv4="eth0"
INTERFACESv6=""
EOF
```

**Explicación del comando:**

- `tee`: escribe el texto recibido en el archivo indicado. Se usa junto con `sudo` porque una redirección simple (`>`) la realizaría el usuario sin privilegios y fallaría.
- `> /dev/null`: descarta la copia que `tee` muestra por pantalla.
- `<<'EOF' ... EOF`: *here-document*; entrega a `tee` el texto escrito entre las dos marcas.
- `INTERFACESv4="eth0"`: el servidor atiende solicitudes DHCP (IPv4) por la interfaz `eth0`.
- `INTERFACESv6=""`: no se utiliza DHCP para IPv6.

### 7.2 Subred y parámetros entregados a los clientes

Se escribió la configuración principal del servicio:

```bash
sudo tee /etc/dhcp/dhcpd.conf > /dev/null <<'EOF'
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 192.168.4.0 netmask 255.255.255.0 {
    range 192.168.4.100 192.168.4.200;
    option routers 192.168.4.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.4.255;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
}
EOF
```

**Explicación de las opciones:**

- `default-lease-time 600`: tiempo de concesión por defecto, en segundos (10 minutos).
- `max-lease-time 7200`: tiempo máximo de concesión, en segundos (2 horas).
- `authoritative`: indica que este es el servidor oficial de la red. Si un cliente solicita una dirección incorrecta, el servidor la rechaza para que pida una nueva.
- `subnet ... netmask ...`: declara la red que administra el servidor.
- `range`: rango de direcciones disponibles para los clientes.
- `option routers`: puerta de enlace que reciben los clientes.
- `option subnet-mask`: máscara de subred.
- `option broadcast-address`: dirección de difusión de la red.
- `option domain-name-servers`: servidores DNS entregados a los clientes.

Cada línea de configuración termina en punto y coma (`;`); su ausencia es el error más frecuente y hace que el servicio no arranque.

### Evidencia

![](./images/image2.png)

---

## 8. Validación de la configuración y habilitación del servicio

Se comprobó que el archivo no tuviera errores de sintaxis:

```bash
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf
```

**Explicación del comando:**

- `dhcpd -t`: prueba la configuración sin iniciar el servidor.
- `-cf`: indica el archivo de configuración a probar.

La salida mostró la versión del servidor (4.4.3-P1), la ruta del archivo de configuración y las rutas de la base de datos de concesiones (`/var/lib/dhcp/dhcpd.leases`) y del archivo de PID, sin mensajes de error. Esto confirma que la configuración es válida.

Luego se dejó el servicio detenido pero habilitado para el siguiente arranque, y se habilitó el servicio SSH:

```bash
sudo systemctl stop isc-dhcp-server
sudo systemctl enable isc-dhcp-server
sudo systemctl enable --now ssh
```

**Explicación de los comandos:**

- `systemctl stop`: detiene el servicio. Se hizo mientras la Raspberry Pi seguía conectada a la red con Internet, para que el servidor DHCP del grupo no respondiera en una red donde ya existe otro servidor DHCP y no interfiriera con ella.
- `systemctl enable`: configura el servicio para que inicie automáticamente con el sistema.
- `enable --now ssh`: habilita el servicio SSH y lo inicia en el momento, para poder acceder a la Raspberry Pi de forma remota.

Los mensajes `is not a native service, redirecting to systemd-sysv-install` son normales: el paquete incluye un script de inicio tradicional (SysV) y `systemd` lo redirige a su propia herramienta de habilitación.

### Evidencia

![](./images/image3.png)

---

## 9. Dirección IP fija y puesta en marcha en la red LAN

Una vez instalado y configurado el servicio, se pasó la Raspberry Pi a la red del switch. Se asignó una dirección IP fija a `eth0`, dentro de la red `192.168.4.0/24`:

```bash
sudo nmcli con mod "Wired connection 1" ipv4.method manual ipv4.addresses 192.168.4.1/24 ipv4.gateway "" ipv4.dns ""
sudo nmcli con up "Wired connection 1"
```

**Explicación de los comandos:**

- `nmcli con mod`: modifica la conexión `Wired connection 1`.
- `ipv4.method manual`: desactiva la configuración automática y usa una dirección fija.
- `ipv4.addresses 192.168.4.1/24`: dirección y máscara de la Raspberry Pi en la LAN.
- `ipv4.gateway "" ipv4.dns ""`: sin puerta de enlace ni DNS propios, ya que la red del switch no tiene salida a Internet.
- `nmcli con up`: aplica los cambios.

Este paso se realizó con teclado y monitor conectados a la Raspberry Pi, ya que al cambiar la dirección se corta la sesión SSH. A continuación se conectó el cable de `eth0` al switch del grupo y se verificó:

```bash
ip a show eth0
```

El resultado esperado es una interfaz en estado `UP` con la dirección `192.168.4.1/24`.

Finalmente se inició el servidor DHCP y se comprobó su estado:

```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```

El estado esperado es `active (running)`.

### Evidencia

<!-- TODO: agregar captura de `ip a show eth0` con 192.168.4.1/24 y de `systemctl status isc-dhcp-server` en active (running). -->

---

## 10. Pruebas de funcionamiento

Para verificar el servicio, se conectó una notebook al mismo switch con el adaptador Ethernet configurado en **"Obtener una dirección IP automáticamente"**. Desde PowerShell se ejecutaron:

```powershell
ipconfig /renew
ipconfig
ping 192.168.4.1
```

**Explicación de los comandos:**

- `ipconfig /renew`: solicita una nueva configuración al servidor DHCP.
- `ipconfig`: muestra la configuración IP actual del equipo.
- `ping 192.168.4.1`: comprueba la conectividad con la Raspberry Pi.

El resultado esperado es que la notebook reciba una dirección entre `192.168.4.100` y `192.168.4.200`, con puerta de enlace `192.168.4.1`, y que el `ping` obtenga respuesta.

En la Raspberry Pi se pueden consultar las concesiones entregadas:

```bash
cat /var/lib/dhcp/dhcpd.leases
```

### Evidencia

<!-- TODO: agregar captura de `ipconfig` en la notebook con la IP recibida y de `dhcpd.leases` en la Raspberry Pi. -->

---

## 11. Conclusión

Se instaló y configuró el servidor DHCP `isc-dhcp-server` en la Raspberry Pi, que quedó lista para brindar el servicio en la red LAN del grupo (`192.168.4.0/24`) formada con el switch. La configuración definió el rango de direcciones, la puerta de enlace y los servidores DNS entregados a los clientes, y se validó su sintaxis antes de habilitar el servicio.

Se comprendió la importancia del orden de las tareas: el paquete debe instalarse mientras el equipo tiene acceso a Internet, y la dirección IP fija se configura al final, cuando la Raspberry Pi pasa a la red aislada del switch. Además, se mantuvo el servidor detenido mientras estuvo conectado a otra red, para evitar que interfiriera con un servidor DHCP ajeno.

De esta manera, la Raspberry Pi pasa a cumplir la función de servidor DHCP de la red del grupo, y las notebooks se configuran automáticamente al conectarse al switch, sin necesidad de asignar direcciones de forma manual. El diseño de la red se documentó mediante un diagrama en Packet Tracer.
