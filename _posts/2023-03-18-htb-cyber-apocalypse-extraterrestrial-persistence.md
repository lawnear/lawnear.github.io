---
layout: single
title: Extraterrestrial Persistence - CTF HTB Cyber Apocalypse 2023
excerpt: "Este es el tercer desafío forense del evento Cyber Apocalypse 2023 de Hack The Box. Se considera de dificultad very-easy y se nos proporciona un script de shell (.sh) en cual en su interior posee un cadena codificada en base64 que contiene la flag."
date: 2023-03-20
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
  - bash
  - base64
---

![](/assets/images/htb-cyber-apocalypse-plaintext-treasure/cyber-apocalypse-ctf-2023.jpg)

**Dificultad:** very-easy

## Enunciado

_Existe el rumor de que los extraterrestres han desarrollado un mecanismo de persistencia que es imposible de detectar. Después de investigar su servidor Linux recientemente comprometido, Pandora encontró una posible muestra de este mecanismo. ¿Puedes analizarlo y averiguar cómo instalan su persistencia?_

## unzip - cat

Se nos entrega un archivo llamado "forensics_extraterrestrial_persistence.zip" el cual en su interior posee un script para Linux llamado `persistence.sh`. En un archivo `.sh`, se pueden escribir secuencias de comandos o instrucciones que serán ejecutadas por el shell en un entorno Unix/Linux. Estos archivos de script son utilizados para automatizar tareas, ejecutar comandos en secuencia, definir variables, realizar operaciones lógicas, entre otras acciones.

```
❯ unzip -l forensics_extraterrestrial_persistence.zip
Archive:  forensics_extraterrestrial_persistence.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
      639  2023-03-06 12:30   persistence.sh
---------                     -------
      639                     1 file
```

Luego de descomprimir el archivo procedemos a verificar el contenido de `persistence.sh`, el cual se ve de la siguiente manera:

![](/assets/images/htb-cyber-apocalypse-extraterrestrial-persistence/sh1.png)

A modo de información, el script `persistence.sh` realiza las siguientes acciones:
  - Asigna el nombre de usuario actual a la variable `n` utilizando el comando `whoami`.
  - Asigna el nombre del host a la variable `h` utilizando el comando `hostname`.
  - Establece la ruta del archivo a `'/usr/local/bin/service'` mediante la variable `path`.
  - Comprueba si tanto el nombre de usuario (`$n`) como el nombre del host (`$h`) no son iguales a "pandora" y "linux_HQ" respectivamente. Si la condición no se cumple, se ejecuta el comando exit, lo que termina la ejecución del script.
  - Utiliza el comando `curl` para descargar el contenido de la URL **'https://files.pypi-install.com/packeges/service'** y lo guarda en el archivo especificado por la variable `$path` utilizando la opción `-o`.
  - Utiliza el comando `chmod` para establecer los permisos de ejecución (`+x`) en el archivo especificado por la variable `$path`.
  - Utiliza el comando `echo -e `para imprimir el contenido en formato de cadena con secuencias de escape.
  - Decodifica y guarda el contenido en el archivo `'/usr/lib/systemd/system/service.service'` utilizando el comando `base64 --decode`.
  - Habilita el servicio `'service.service'` utilizando el comando `systemctl enable`.

## base64

_base64: Es un método de codificación utilizado para representar datos binarios o cualquier tipo de datos en formato de texto ASCII. Se utiliza comúnmente para transferir datos a través de canales que solo admiten texto, como el correo electrónico o en la codificación de URL._

_El nombre "Base64" proviene del hecho de que utiliza un alfabeto de 64 caracteres para representar los datos. Los 64 caracteres consisten en letras mayúsculas y minúsculas (A-Z, a-z), los dígitos del 0 al 9 y dos caracteres especiales, generalmente "+" y "/"._

Como podemos observar, el script anterior está decodificando una cadena de caracteres con el comando `base64 --decode` y la está guardando como contenido dentro del archivo `'/usr/lib/systemd/system/service.service'`, por lo que procedemos a decodificar la misma cadena y verificar el contenido real.

![](/assets/images/htb-cyber-apocalypse-extraterrestrial-persistence/sh2.png)

Muy bien, hemos decodificado el contenido y ahora podemos acceder a los datos reales que se están guardando en el archivo `'/usr/lib/systemd/system/service.service'` y, por lo tanto, ya visualizamos la **flag**.

## Flag

**HTB{th3s3_4l13nS_4r3_s00000_b4s1c}**