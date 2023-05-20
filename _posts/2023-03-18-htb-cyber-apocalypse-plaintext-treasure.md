---
layout: single
title: PlainText Treasure - CTF HTB Cyber Apocalypse 2023
excerpt: "La inteligencia de amenazas descubrió que los alienígenas operan a través de un servidor de comando y control alojado en su infraestructura. Pandora logró penetrar sus defensas y tener acceso a su red interna. Debido a que su servidor usa HTTP, Pandora capturó el tráfico de la red para robar las credenciales de administrador del servidor. Abra el archivo proporcionado con Wireshark y localice el nombre de usuario y la contraseña del administrador."
date: 2023-03-18
classes: wide
header:
  teaser: /assets/images/htb-cyber-apocalypse-plaintext-treasure/cyber-apocalypse-ctf-2023.jpg
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - ctf
  - forense
tags:  
  - wireshark
  - pcap
  - strings
---

![](/assets/images/htb-cyber-apocalypse-plaintext-treasure/cyber-apocalypse-ctf-2023.jpg)

Enunciado:

_La inteligencia de amenazas descubrió que los alienígenas operan a través de un servidor de comando y control alojado en su infraestructura. Pandora logró penetrar sus defensas y tener acceso a su red interna. Debido a que su servidor usa HTTP, Pandora capturó el tráfico de la red para robar las credenciales de administrador del servidor. Abra el archivo proporcionado con Wireshark y localice el nombre de usuario y la contraseña del administrador._

## File

Ejecutamos el comando `file` para averiguar contra qué tipo de archivo nos enfrentamos. En este caso, se trata de un archivo llamado "capture.pcap" que contiene una captura de paquetes de red en formato pcap. Esta captura utiliza marcas de tiempo en microsegundos y tiene una longitud máxima de captura de 262144 bytes.

```
❯ file capture.pcap
capture.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 262144)
```

## Wireshark

Tal y como sugiere el enunciado del desafío, procedemos a abrir el archivo con `Wireshark` para encontrar el nombre de usuario y la contraseña del administrador. Nuevamente, el enunciado nos entrega una pista: el tráfico de red capturado corresponde al protocolo HTTP, lo que significa que los datos capturados se encuentran en texto plano. Por lo tanto, el usuario y la contraseña deben ser fáciles de encontrar.
```
❯ wireshark capture.pcap
```
![](/assets/images/htb-cyber-apocalypse-plaintext-treasure/wireshark1.png)

Siempre es una buena practica verificar la jerarquia de protocolos que nos entrega Wireshark, esta opción muestra una vista jerárquica de los protocolos presentes en el tráfico de red capturado. Podemos ver la estructura en capas de los protocolos, lo que nos permite comprender cómo interactúan los diferentes protocolos en una comunicación. Esta vista jerárquica nos ayuda a identificar rápidamente los protocolos involucrados y su distribución en el tráfico.

![](/assets/images/htb-cyber-apocalypse-plaintext-treasure/wireshark2.png)

Como podemos observar, efectivamente la mayor cantidad de tráfico de red está relacionada con el protocolo HTTP. En este desafío, se nos pide encontrar el nombre de usuario y la contraseña, los cuales generalmente se envían a través del método POST al interactuar con un sitio web. Por lo tanto, procedemos a aplicar el siguiente filtro en Wireshark que nos permita visualizar específicamente este tipo de paquetes: `http.request.method == "POST"`

![](/assets/images/htb-cyber-apocalypse-plaintext-treasure/wireshark3.png)
