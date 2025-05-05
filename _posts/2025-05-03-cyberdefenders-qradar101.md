---
layout: single
title: SIEM ThreatHunting IBM QRadar CyberDefenders
excerpt: "Este desafio es considerado de dificultad medium, donde exploraremos cómo investigar incidentes de seguridad utilizando una poderosa herramienta de IBM Qradar SIEM. Ya sea que tengas curiosidad sobre threath Hunting, incident response o herramientas SIEM como QRadar este articulo te puede ser de utilidad."
date: 2025-05-03
classes: wide
header:
  teaser: /assets/images/cyberdefenders-qradar101/cyberdefenders-qradar101.png
  teaser_home_page: true
  icon: /assets/images/cyberdefenders_icon.png
categories:
  - Threat Hunting
  - ctf
tags:  
  - Wireshark
  - Execution
  - Persistence
  - Privilege Escalation
  - Defense Evasion
  - Discovery
  - Lateral Movement
  - Collection
  - Command and Control
  - Exfiltration

---

![](/assets/images/cyberdefenders-qradar101/cyberdefenders-qradar101.png)

**Dificultad:** Medium

## Instrucciones

- Compatibilidad: VirtualBox
- Asegúrese de tener una subred de solo host dentro del siguiente rango de IP 192.168.20.0/24.
- Asigne el adaptador de red adecuado (192.168.20.0/24) a la VM antes de iniciarlo.
- Espere unos minutos después de que se complete la importación y luego visite: https://192.168.20.21/.
- Credenciales de desafío: Panel QRadar: **admin**:**Admin@123** - SSH: root:**cyberdefenders**

En caso de que enfrente un problema de licencia, vaya a > License Pool Management. Edite y establezca eps > 0 y edite el FPM y configúrelo en 0. Esto asegurará que no tenga un problema de licencia.

Requisitos de Hardware: 8GB de memoria y 65GB de espacio en disco.

## Escenario
Una compañía financiera se vio comprometida, y están buscando un analista de seguridad para ayudarlos a investigar el incidente. La compañía sospecha que una información privilegiada ayudó al atacante a ingresar a la red, pero no tienen evidencia.


El análisis inicial realizado por el equipo de la compañía demostró que muchos sistemas estaban comprometidos. Además, las alertas indican el uso de herramientas maliciosas bien conocidas en la red. Como analista SOC, se le asigna investigar el incidente utilizando QRadar SIEM y reconstruir los eventos llevados a cabo por el atacante.

Conjunto de datos:

Sysmon - rápido en la configuración de seguridad  
Registro de powershell  
Eventlog Windows  
IDS Suricata  
Registros de Zeek (conn, HTTP)

---
## Prepararación del laboratorio
Antes de empezar, les recomiendo preparar el laboratorio con el vídeo oficial de [Cyberdefenders](https://www.youtube.com/watch?v=4uM4JEhbEjI)

## Q1) ¿Cuántas fuentes de registro disponibles?

![](/assets/images/cyberdefenders-qradar101/figura1.png)  
**Figura1:** Navegamos al apartado "Admin" y seleccionamos Log Sources para revisar las fuentes de registros

![](/assets/images/cyberdefenders-qradar101/figura2.png)  
**Figura2:** Encontramos **15** fuentes de registros
## Q2) ¿Cuál es el software IDS utilizado para monitorear la red?

![](/assets/images/cyberdefenders-qradar101/figura3.png)  
**Figura3:** Identificamos que el IDS es un Suricata

## Q3) ¿Cuál es el nombre de dominio utilizado en la red?

![](/assets/images/cyberdefenders-qradar101/figura4.png)  
**Figura4:** Log Activity > Aplicamos el filtro desde 2020 hasta 2021 con un límite de 100,000 registros.

![](/assets/images/cyberdefenders-qradar101/figura5.png)  
**Figura5:** Añadimos un filtro, como registros relacionados con eventos de inicio de sesión exitosos (ID: 4624).  

![](/assets/images/cyberdefenders-qradar101/figura6.png)  
**Figura6:** Seleccionamos un registro de inicio de sesión del DC

![](/assets/images/cyberdefenders-qradar101/figura7.png)  
**Figura7:** Encontramos el dominio: **hackdefend.local**

## Q4) Múltiples IP se comunicaban con el servidor malicioso. Uno de ellos termina con "20". Proporcione la IP completa.

![](/assets/images/cyberdefenders-qradar101/figura8.png)  
**Figura8:** Para abordar este caso, fuimos al Dashboard y miramos los Top Orígenes.  
Y encontramos que la ip **192.168.20.20** tiene la mayor cantidad de Offenses con **7**

![](/assets/images/cyberdefenders-qradar101/figura9.png)  

**Figura9:** Editamos la búsqueda para mostrar un registro de actividad por IP de origen para ver qué IP's generaron mayor cantidad de eventos.

![](/assets/images/cyberdefenders-qradar101/figura10.png)  

**Figura10:** Se válida que la IP **192.168.20.20** es la que ha generado mayor comunicación. 

## Q5) ¿Cuál es el SID de la regla de alerta más frecuente en el conjunto de datos?

![](/assets/images/cyberdefenders-qradar101/figura11.png)  
**Figura11:** Editamos la búsqueda filtrar por RULE SID (custom) 

![](/assets/images/cyberdefenders-qradar101/figura12.png)  
**Figura12:** El SID de la regla de alerta más frecuente en el conjunto de datos es **2027865**.

## Q6) ¿Cuál es la dirección IP del atacante?

![](/assets/images/cyberdefenders-qradar101/figura13.png)  
**Figura13:** Nos dirigimos a Offenses y limpiamos los filtros para revisar las casos cerrados.

![](/assets/images/cyberdefenders-qradar101/figura14.png)  
**Figura14:** Identificamos la única IP sospechosa **192.20.80.25** del atacante.

## Q7) El atacante estaba buscando datos pertenecientes a uno de los proyectos de la compañía, ¿puede encontrar el nombre del proyecto?

![](/assets/images/cyberdefenders-qradar101/figura15.png)  
**Figura15:** Podemos buscar el proyecto con expresión regular is **project**

![](/assets/images/cyberdefenders-qradar101/figura16.png)  
**Figura16:** Tras examinar los 4 eventos y leer su carga útil encontramos que el proyecto es: **project48**

## Q8) ¿Cuál es la dirección IP de la primera máquina infectada?

![](/assets/images/cyberdefenders-qradar101/figura17.png)  
**Figura17:** Filtrado de tráfico desde la IP atacante **192.20.80.25** ordenado cronológicamente. La primera conexión comprometida corresponde a **192.168.10.15** con fecha: Nov 8, 2020, 10:30:49 PM

Información del Payload:
~~~java
"timestamp":"2020-11-08T19:39:54.261057+0000","flow_id":726286645976969,"in_iface":"bond0","event_type":"alert","src_ip":"192.20.80.25","src_port":449,"dest_ip":"192.168.10.15","dest_port":50026,"proto":"TCP","community_id":"1:IBvhyk\/xYPz6lb5qUQYH6BPlYMU=","alert":{"action":"allowed","gid":1,"signature_id":2025644,"rev":1,"signature":"ET MALWARE Possible Metasploit Payload Common Construct Bind_API (from server)","category":"A Network Trojan was detected","severity":1,"metadata":{"updated_at":["2018_07_09"],"tag":["Metasploit"],"signature_severity":["Critical"],"former_category":["TROJAN"],"deployment":["Datacenter","Internal","Internet","Perimeter"],"created_at":["2016_05_16"],"attack_target":["Client_and_Server"],"affected_product":["Any"]},"rule":"alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:\"ET MALWARE Possible Metasploit Payload Common Construct Bind_API (from server)\"; flow:from_server,established; content:\"|60 89 e5 31|\"; content:\"|64 8b|\"; distance:1; within:2; content:\"|30 8b|\"; distance:1; within:2; content:\"|0c 8b 52 14 8b 72 28 0f b7 4a 26 31 ff|\"; distance:1; within:13; content:\"|ac 3c 61 7c 02 2c 20 c1 cf 0d 01 c7 e2|\"; within:15; content:\"|52 57 8b 52 10|\"; distance:1; within:5; classtype:trojan-activity; sid:2025644; rev:1; metadata:affected_product Any, attack_target Client_and_Server, created_at 2016_05_16, deployment Perimeter, deployment Internet, deployment Internal, deployment Datacenter, former_category TROJAN, signature_severity Critical, tag Metasploit, updated_at 2018_07_09;)"},"app_proto":"failed","payload_printable"
~~~
 
El payload parece es generado mediante Metasploit.
Bind API: mecanismo donde el sistema comprometido abre un puerto (**:50026**) para escuchar conexiones entrantes del atacante

## Q9) ¿Cuál es el nombre de usuario del empleado infectado usando 192.168.10.15?

![](/assets/images/cyberdefenders-qradar101/figura18.png)    
**Figura18:** Considerando la fecha y hora **Nov 8, 2020, 10:30:13 PM** donde se detectó el ingreso y el log de ingreso que vemos en imagen es de las **10:30:49 PM**, descubrimos que el empleado es **nour** 

## Q10) A los hackers no les gusta registrar, ¿Qué registro estaba verificando el atacante para ver si estaba habilitado?

![](/assets/images/cyberdefenders-qradar101/figura19.png)    
**Figura19:** Pudimos obtener este dato filtrando por el nombre de usuario **nour**, en esta figura podemos ver el que el atacante empieza a utilizar PowerShell para evadir el registro en el dominio local.

## Q11) ¿Nombre del segundo sistema al que el atacante apuntó para encubrir al empleado?

![](/assets/images/cyberdefenders-qradar101/figura20.png)    
**Figura20:** Aplicamos filtro: **Process CommandLine (custom) contains any of del** para buscar por procesos eliminados y examinamos los registros.

![](/assets/images/cyberdefenders-qradar101/figura21.png)    
**Figura21:** Log del comando ejecutado  
**del sami.xlsx:** El atacante eliminó el archivo sami.xlsx para destruir evidencia relacionada con el empleado, entonces la respuesta es el sistema **MGNT-01**

## Q12) ¿Cuándo fue la primera conexión maliciosa al controlador de dominio (hora de inicio de registro - hh:mm:ss)?
![](/assets/images/cyberdefenders-qradar101/figura22.png)    
**Figura22:** Buscamos el event id de conexiones de red. 

![](/assets/images/cyberdefenders-qradar101/figura23.png)    
**Figura23:** Agregamos el id al filtro y examinamos los registros


![](/assets/images/cyberdefenders-qradar101/figura24.png)  
**Figura24:** En el segundo registro notamos que el puerto de destino va a 389, que es el LDAP con la orientación del sistema interno, por lo que la hora es **11:14:10**

## Q13) ¿Cuál es el hash md5 del archivo malicioso?

![](/assets/images/cyberdefenders-qradar101/figura25.png)  
**Figura25:** Aplicamos un filtro por payloads que tenga el hash md5 y ahora podemos ver los archivos que se cargan en el sistema.

![](/assets/images/cyberdefenders-qradar101/figura26.png)    
**Figura26:** Encontramos el hash **MD5=9D08221599FCD9D35D11F9CBD6A0DEA3**

## Q14) ¿Cuál es el ID de la técnica de persistencia MITRE utilizado por el atacante?

![](/assets/images/cyberdefenders-qradar101/figura27.png)    
**Figura27:** Buscamos en google las técnicas más comunes para establecer la persistencia por parte de los actores maliciosos y son el uso de las claves de ejecución del registro y las carpetas de inicio en un sistema Windows.

![](/assets/images/cyberdefenders-qradar101/figura28.png)  
**Figura28:** Aplicamos el filtro con event id 13 y agregando la columna target object encontramos la ruta \windows\current\version

![](/assets/images/cyberdefenders-qradar101/figura29.png)  
**Figura29:** Finalmente encontramos el **ID T1547.001** de [MITRE ATT&CK](https://attack.mitre.org/techniques/T1547/001/) buscando la ruta encontrada.

## Q15) ¿Qué protocolo se utiliza para realizar el descubrimiento del host?

- Zeek es un framework open source que se utiliza para el análisis y la búsqueda activa de amenazas para una red.

![](/assets/images/cyberdefenders-qradar101/figura30.png)  
**Figura30:** Podemos descubrir esta información analizando el tráfico saliente desde “192.168.10.15”, con Log source is Zeek_conn en el filtro quitando tcp y udp, vemos que el protocolo utilizado es **ICMP (Ping)**.


## Q16) Cuál es el servicio de correo electrónico utilizado por la empresa?(una palabra)

![](/assets/images/cyberdefenders-qradar101/figura31.png)  
**Figura31:**  El servidor de correo electrónico está fuera de la red interna, por lo que excluimos la red interna en el filtro con el puerto 53 que se utiliza para resolver nombres de dominio. 

![](/assets/images/cyberdefenders-qradar101/figura32.png)  
**Figura32:** Analizamos las primeras IP'S en [AbuseIPDB](https://www.abuseipdb.com/check/13.107.252.10) y encontramos que el servicio utilizado por la empresa es **office365** de Microsoft.

## Q17) ¿Cuál es el nombre del archivo malicioso utilizado para la infección inicial?

![](/assets/images/cyberdefenders-qradar101/figura33.png)  
**Figura33:** Aplicamos el filtro a la primera máquina infectada 192.168.10.15 y buscamos nuevamente por el hash.

![](/assets/images/cyberdefenders-qradar101/figura34.png)  
**Figura34:** Encontramos el archivo **important_instructions.docx**

## Q18) ¿Cuál es el nombre de la nueva cuenta agregada por el atacante?

![](/assets/images/cyberdefenders-qradar101/figura35.png)  
**Figura35:** Tras indagar en google encontramos que el id para agregar nuevas cuentas es el 4720, por lo que aplicamos el filtro con este identificador.

![](/assets/images/cyberdefenders-qradar101/figura36.png)  
**Figura36:** Encontramos que la cuenta agregada por el atacante es **Rambo**

## Q19) ¿Cuál es el PID del proceso que realizó la inyección?

![](/assets/images/cyberdefenders-qradar101/figura37.png)  
**Figura37:** Buscamos el event id del proceso que realiza inyección y identificamos el ID 8 por lo que aplicamos el filtro con este identificador.

![](/assets/images/cyberdefenders-qradar101/figura38.png)  
**Figura38:** Encontramos el PID en notepad.exe que se estaba cargando

## Q20) ¿Cuál es el nombre de la herramienta utilizada para el movimiento lateral?

![](/assets/images/cyberdefenders-qradar101/figura39.png)  
**Figura39:** podemos filtrar registros que contengan comandos como admin o Admin o cmd

![](/assets/images/cyberdefenders-qradar101/figura40.png)  
**Figura40:** Notamos que curre una edición en la ruta **Software\Policies\Microsoft\Windows\PowerShell** para cambiar de cuenta
![](/assets/images/cyberdefenders-qradar101/figura41.png)
**Figura41:** Tras realizar búsqueda de la ruta **Software\Policies\Microsoft\Windows\PowerShell** encontramos Impacket y dentro de las técnicas utilizadas **wmiexec.py**

## Q21) Atacante exfiltrado un archivo, ¿cuál es el nombre de la herramienta utilizada para la exfiltración?

![](/assets/images/cyberdefenders-qradar101/figura42.png)  
**Figura42:** Filtramos por la ip de atacante que obtendría el archivo exfiltrado y la fuente de registro de suricata IDS esto ya que tal como es el software detectará cualquier exfiltración que ocurra.

![](/assets/images/cyberdefenders-qradar101/figura43.png)
**Figura43:** Examinando los logs encontramos que utilizó **Curl** para realizar la exfiltración

## Q22) ¿Quién es el otro administrador de dominio legítimo que no sea el administrador?

![](/assets/images/cyberdefenders-qradar101/figura44.png)
**Figura44:** Buscamos en google el identificador para asignar privilegios especiales y obtuvimos el 4672. Por lo que aplicamos el filtro con este identificador.

![](/assets/images/cyberdefenders-qradar101/figura45.png)
**Figura45:** El administrador impostor es **adam**

## Q23) El atacante utilizó la técnica de descubrimiento de host para saber cuántos hosts disponibles en una determinada red, ¿cuál es la red que el hacker escaneó desde la IP del host 1 a 30?

![](/assets/images/cyberdefenders-qradar101/figura46.png)
**Figura46:** Al filtrar por **zeek** logs y especificar la ip del atacante podemos ver que comienza a escanear desde la 192.168.20.1 a 192.168.20.30 por lo que la red es **192.168.20.0.**

## Q24) ¿Cómo se llama el empleado que contrató al atacante?

![](/assets/images/cyberdefenders-qradar101/figura47.png)
**Figura47:** Aplicamos misma estrategia de filtrado que la pregunta 21

![](/assets/images/cyberdefenders-qradar101/figura48.png)
**Figura48:** El empleado es **sami**.

---