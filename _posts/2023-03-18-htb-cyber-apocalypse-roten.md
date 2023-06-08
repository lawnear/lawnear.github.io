---
layout: single
title: Roten - CTF HTB Cyber Apocalypse 2023
excerpt: "Este es el cuarto desafío forense del evento \"Cyber Apocalypse 2023\" de Hack The Box, considerado de dificultad easy. En este desafío, analizaremos una captura de tráfico utilizando wireshark. La captura contiene datos de un servidor web comprometido en el cual se ha cargado una webshell ofuscada y escrita en PHP. Deberemos intentar desofuscarla para encontrar la flag."
date: 2023-03-21
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
  - php
---

![](/assets/images/htb-cyber-apocalypse-roten/cyber-apocalypse-ctf-2023.jpg)

**Dificultad:** easy

## Enunciado

_El iMoS es responsable de recopilar y analizar datos de objetivos en varias galaxias. Los datos se recopilan a través de su servidor web, al que solo puede acceder el personal autorizado. Sin embargo, iMoS sospecha que su servidor web se ha visto comprometido y no puede localizar el origen de la infracción. Sospechan que se ha cargado algún tipo de shell, pero no pueden encontrarlo. El iMoS le ha proporcionado algunos datos de red para analizar, depende de usted salvarnos._

## File

Ejecutamos el comando `file` para averiguar contra qué tipo de archivo nos enfrentamos. En este caso, se trata de un archivo llamado "challenge.pcap" que contiene una captura de paquetes de red en formato pcap. Esta captura utiliza marcas de tiempo en microsegundos y tiene una longitud máxima de captura de 262144 bytes.

```
❯ file challenge.pcap
challenge.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 262144)
```

## Wireshark

Procedemos a abrir el archivo en `wireshark` para analizar el contexto de los paquetes de red capturados y encontrar algún tipo de webshell cargada en el sitio web. Para ello, debemos identificar quien es el servidor web, así como las rutas normales que se sirven desde el sitio web y determinar qué elementos resultan sospechosos.

```
❯ wireshark challenge.pcap
```
![](/assets/images/htb-cyber-apocalypse-roten/wireshark1.png)

Como ya hemos visto, es una buena práctica verificar la jerarquía de protocolos que nos proporciona `wireshark`. En esta captura de tráfico en particular, la herramienta nos indica que el 75,9% de los paquetes capturados corresponden al protocolo HTTP.

![](/assets/images/htb-cyber-apocalypse-roten/wireshark2.png)

Aplicamos un filtro para ver únicamente el tráfico HTTP y observar el tipo de datos que se están transmitiendo entre clientes y servidor.

![](/assets/images/htb-cyber-apocalypse-roten/wireshark3.png)

Como podemos observar, el servidor web es `targetaggregator.intergalacticministry.com` y su dirección IP es `172.31.9.156`. Con esta información, podemos comenzar a aplicar otros filtros relevantes. Recordemos que el enunciado del desafío nos indica que se ha cargado algún tipo de webshell. Por lo tanto, es recomendable filtrar las solicitudes web que utilizaron el método POST para interactuar con el sitio, ya que este método podría haber sido utilizado por el atacante para subir información al servidor y, por lo tanto, nos permitiría identificar la carga de la webshell.

Procedemos a filtrar los paquetes utilizando el siguiente filtro en `wireshark`: 
```
http.host == "targetaggregator.intergalacticministry.com" && http.request.method == "POST"
```

![](/assets/images/htb-cyber-apocalypse-roten/wireshark4.png)

La imagen anterior nos indica que existen dos rutas para subir parámetros o información al sitio web. En primer lugar, tenemos la ruta `/map-update.php`, la cual, al realizar un seguimiento HTTP, nos indica que se utiliza para subir datos de objetivos geográficos. Y en segundo lugar, tenemos la ruta `/results_display.php`, que se utiliza para mostrar resultados de ubicación según los parámetros enviados a través del método POST.

En este punto, hay un dato que nos llama la atención, y es el hecho de que a través de la ruta `/map-update.php` se han subido dos archivos del tipo PDF y un tercero del tipo PHP.

![](/assets/images/htb-cyber-apocalypse-roten/wireshark5.png)

Al realizar un seguimiento de la secuencia HTTP en la cual se envió el archivo PHP, podemos observar que se trata de un archivo llamado `galacticmap.php` y que, según su contenido, se encuentra ofuscado.

![](/assets/images/htb-cyber-apocalypse-roten/wireshark6.png)

Por lo tanto, procedemos a guardar el contenido ofuscado en un archivo `.php` para analizarlo más adelante. Esto se puede realizar de muchas maneras, pero para evitar complicaciones, simplemente copiaré y pegaré el contenido del archivo PHP en un archivo local en mi equipo. El archivo quedaría de la siguiente manera:

![](/assets/images/htb-cyber-apocalypse-roten/galacticmap.png)

## Acciones del atacante

Con el análisis anterior, hemos podido identificar la dirección IP de origen `146.70.38.48`, la cual ha subido un archivo PHP ofuscado al servidor web comprometido. Sin embargo, nos preguntamos qué otras acciones realizó esa dirección IP.

Para buscar más información de manera más cómoda, utilizaré la herramienta `tshark`, la cual proporciona funcionalidades similares a las de `wireshark`, pero se ejecuta en modo terminal a través de comandos. Dicho esto, utilizaré el siguiente comando para filtrar únicamente las solicitudes "POST" efectuadas por la dirección IP sospechosa: 
```
tshark -r challenge.pcap -Y "http.request.method == POST and ip.addr == 146.70.38.48"
```

![](/assets/images/htb-cyber-apocalypse-roten/tshark.png)

Como podemos observar, hemos encontrado exactamente lo mismo que anteriormente en la interfaz de `wireshark`, por lo que la herramienta funciona sin problemas. Ahora filtraremos todo lo que haya hecho la dirección IP sospechosa, pero que no sea a través de "POST".

```
tshark -r challenge.pcap -Y "http.request.method !== POST and ip.addr == 146.70.38.48"
```

El comando anterior nos da como resultado múltiples solicitudes utilizando el método "GET", como se muestra en la siguiente imagen:

![](/assets/images/htb-cyber-apocalypse-roten/tshark2.png)

Esto llama mi atención, ya que parece ser que, posterior a subir el archivo `galacticmap.php` al servidor, nuestro posible atacante no tiene claridad de dónde puede haber quedado alojado el archivo que acaba de subir. Por lo tanto, recurre a la enumeración de rutas para lograr dar con su archivo. Perfeccionemos nuestro comando de tshark para validar esta hipótesis de la siguiente forma:

```
tshark -r challenge.pcap -Y "http.request.method !== POST and ip.addr == 146.70.38.48" -T fields -e ip.src -e ip.dst -e http.request.method -e http.user_agent -e http.host -e http.request.uri
```

El comando anterior filtrará todas las solicitudes HTTP que no utilicen el método POST y que tengan la dirección IP `146.70.38.48` como origen o destino. Luego, mostrará información específica de cada solicitud, como las direcciones IP, el método de solicitud, el User-Agent, el host y el URI solicitado.

El resultado del comando anterior nos permite validar que efectivamente nuestro sospechoso utilizó la herramienta Wfuzz para realizar una enumeración de rutas y poder encontrar el archivo `galacticmap.php`.

![](/assets/images/htb-cyber-apocalypse-roten/tshark3.png)

Si lo vemos desde el punto de vista del atacante, es muy probable que el comando ejecutado haya sido el siguiente:

```
wfuzz -w <wordlist> -u http://targetaggregator.intergalacticministry.com/FUZZ/galacticmap.php --hc 404
```

Finalmente, podemos observar que una vez que el atacante logra encontrar el archivo `galacticmap.php`, procede a ejecutar comandos, confirmando así que se trata de una webshell.

![](/assets/images/htb-cyber-apocalypse-roten/tshark4.png)

Los comandos ejecutados se ven en formato URL-encoded, por lo que, decodificados, quedan de la siguiente manera:

![](/assets/images/htb-cyber-apocalypse-roten/cyberchef.png)

Hemos finalizado el análisis y hemos logrado determinar la dirección IP del atacante, las herramientas utilizadas, la ruta que utilizó para subir la webshell programada en PHP y el contenido del archivo `galacticmap.php`, pero, ¿dónde está la flag?.

## PHP

En pasos anteriores hemos hecho una copia del archivo `galacticmap.php`, el cual se encontraba ofuscado y probablemente contiene nuestra flag. En primera instancia, quise validar las capacidades que poseía la webshell del atacante, por lo que monté un servidor web Apache en mi máquina local y realicé algunas pruebas. A continuación, se muestra el procedimiento:

Copiamos el archivo `galacticmap.php` en la ruta `/var/www/html` de nuestro equipo Linux y luego habilitamos el servidor web Apache.

Después, a través de un navegador, podemos acceder a nuestro localhost y visualizar la webshell del atacante, la cual permite ejecutar comandos y subir archivos, como se muestra en la siguiente imagen:

![](/assets/images/htb-cyber-apocalypse-roten/php.png)

Tenía la esperanza de encontrar la flag en el código HTML del sitio, pero no fue el caso. Después de investigar durante un tiempo, encontré una forma factible de desofuscar el contenido del archivo `galacticmap.php`.

Al final del archivo `galacticmap.php`, se está utilizando la función `eval()` de PHP, la cual evalúa la cadena `$bhrTeZXazQ` como código PHP en tiempo de ejecución. Esto me hizo pensar que quizás no era necesario evaluar dicha cadena, ya que no necesitaba ejecutar el código en un sitio web, sino desofuscarlo y hacerlo legible. Por lo tanto, realicé un cambio en la función final y la reemplacé por la función `echo()` de PHP, como se muestra en la siguiente imagen:

![](/assets/images/htb-cyber-apocalypse-roten/php2.png)

Finalmente, ejecuté el código PHP modificado con un intérprete PHP y encontré la bendita flag, la cual se encontraba como un comentario dentro del código desofuscado.

`php galacticmap_modificado.php`

![](/assets/images/htb-cyber-apocalypse-roten/php3.png)

## Flag

**HTB{W0w_ROt_A_DaY}**
