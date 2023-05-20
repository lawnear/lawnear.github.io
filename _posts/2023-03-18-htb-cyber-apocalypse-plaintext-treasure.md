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

Siempre es una buena practica verificar la jerarquia de protocolos que nos entrega Wireshark, esta opción muestra una vista jerárquica de los protocolos presentes en el tráfico de red capturado. Podemos ver la estructura en capas, lo que nos permite comprender cómo interactúan los diferentes protocolos en una comunicación. Esta vista nos ayuda a identificar rápidamente los protocolos involucrados y su distribución en el tráfico.

![](/assets/images/htb-cyber-apocalypse-plaintext-treasure/wireshark2.png)

Como podemos observar, efectivamente la mayor cantidad de tráfico de red está relacionado con el protocolo HTTP. En este desafío, se nos pide encontrar el nombre de usuario y la contraseña, los cuales generalmente se envían a través del método POST al interactuar con un sitio web. Por lo tanto podemos aplicar el siguiente filtro en Wireshark para visualizar específicamente este tipo de paquetes: `http.request.method == "POST"`

![](/assets/images/htb-cyber-apocalypse-plaintext-treasure/wireshark3.png)

Hemos encontrado 4 coincidencias para nuestra búsqueda. Al hacer clic en el primer paquete de red filtrado, que apunta a `/token`, podemos ver la información de interés para este desafío. Tenemos a la vista los parámetros **username** y **password**.

![](/assets/images/htb-cyber-apocalypse-plaintext-treasure/wireshark4.png)

Para poder observar de mejor manera el tráfico capturado y los datos necesarios para resolver el desafío, hacemos clic derecho sobre el paquete de red de interés y seleccionamos la opción **Seguir** -> **Secuencia HTTP**. Esta función nos permite visualizar y seguir el flujo completo de una transacción HTTP entre un cliente y un servidor. Resulta muy útil para analizar en detalle la comunicación entre ambas partes y examinar los mensajes HTTP intercambiados.


Al realizar el seguimiento de la secuencia HTTP asociada, Wireshark muestra todos los paquetes relacionados con el flujo TCP, que en este caso es el número 4. Este flujo corresponde a la sesión establecida al momento de enviar las credenciales al servidor web a través de la función `/token`.
Con esto, podemos visualizar el nombre de usuario y la contraseña enviada , esta última corresponde a la **FLAG** del desafío.

![](/assets/images/htb-cyber-apocalypse-plaintext-treasure/wireshark5.png)

## Strings

Este desafio es considerado por Hack The Box como _very-easy_, por lo tanto se podia resolver facilmente utilizando el comando `strings` y filtrando por la palabra HTB.

_Strings:se utiliza para extraer y mostrar cadenas de texto legibles desde un archivo o una entrada de datos binarios. Su objetivo principal es identificar y extraer las secuencias de caracteres legibles en archivos que de otra manera podrían contener una mezcla de datos binarios y texto._
```
❯ strings capture.pcap | grep -i HTB
HTB{th3s3_4l13ns_st1ll_us3_HTTP}
```

## Flag
**HTB{th3s3_4l13ns_st1ll_us3_HTTP}**
