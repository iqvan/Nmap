# Nmap 

Security, Owasp top 10, Cron. (#nmap, #samba, #suid, #pathvarmanipulation #smb )

## Tabla de contenido

- [Introducción](#Introducción).
- [Nmap Switches](#Nmap-Switches).
- [Tipos de escaneo - Descripción general](#Tipos-de-escaneo---Descripción-general).


--------------------------------
### Introducción
-------------------------------

¿Por qué nmap?

 La respuesta corta es que actualmente es el estándar de la industria por una razón: ninguna otra herramienta de escaneo de puertos se acerca a igualar su funcionalidad (aunque algunos recién llegados ahora la están igualando en velocidad). Es una herramienta extremadamente poderosa, aún más poderosa gracias a su motor de scripting que puede usarse para escanear vulnerabilidades y, en algunos casos, ¡incluso realizar el exploit directamente!

--------------------------------
### Nmap Switches
-------------------------------

Syn Scan
```
nmap -sS
```
UDP scan
```
nmap -sU
```
Sistema operativo se está ejecutando el objetivo
```
nmap -O
```
Detectar la versión de los servicios que se ejecutan en el objetivo
```
nmap -sV
```
Verbose
```
nmap -v
```
Doble verbose
```
nmap -vv
```
Guardar los resultados de nmap en tres formatos principales
```
nmap -oA
```
Guardar los resultados de nmap en un formato "normal"
```
nmap -oN
```
Guardar los resultados en un formato "grepable"
```
nmap -oG
```
Habilitar el modo "agresivo"
```
nmap -A
```
Cronometraje en el nivel 5
```
nmap -T5
```
Solo escanee el puerto 80
```
nmap -p 80
```
Escanee los puertos 1000-1500
```
nmap -p 1000-1500
```
Escanee todos los puertos
```
nmap -p-
```
Activar un script de la biblioteca de scripting de nmap
```
nmap --script
```
Activar todos los scripts en la categoría "vuln"
```
nmap --script=vuln
```
--------------------------------
### Tipos de escaneo - Descripción general
-------------------------------

1) Exploraciones de TCP Connect
    ```
    nmap -sT
    ```
2) Escaneos SYN "semiabiertos"
    ```
    nmap -sS
    ```
3) Escaneos UDP
    ```
    nmap -sU
    ```
4) Escaneos nulos de TCP
    ```
    nmap -sN
    ```
5) Exploraciones TCP FIN
    ```
    nmap -sF
    ```
6) Escaneos TCP Xmas
    ```
    nmap -sX
    ```
--------------------------------
### Escaneos de TCP Connect
-------------------------------

 ```
 Client          Server
  | ------SYN-----> |

  | <--SYN/ACK----  |

  | ------ACK-----> |
 ```

  ```
 Client          Server
  | ------SYN-----> |

  | <----RST------  |

 ```

Proteger con iptables:
```
iptables -I INPUT -p tcp --dport <port> -j REJECT --reject-with tcp-reset
```

Si un puerto está cerrado, ¿qué bandera debe enviar el servidor para indicar esto?
```
RST
```
--------------------------------
### Escaneos SYN Scans (Half-open,stealth)
-------------------------------

 ```
 Client          Server
  | ------SYN-----> |

  | <---SYN/ACK---  |

  | ------RST-----> |
 ```

* Ventajas:

    * Se puede utilizar para eludir los sistemas de detección de intrusiones más antiguos, ya que buscan un apretón de manos completo de tres vías. A menudo, este ya no es el caso de las soluciones IDS modernas; es por esta razón que los análisis SYN todavía se denominan con frecuencia análisis "furtivos".
    * Las aplicaciones que escuchan en puertos abiertos no suelen registrar los escaneos SYN, ya que la práctica estándar es registrar una conexión una vez que se ha establecido por completo. Nuevamente, esto juega con la idea de que los escaneos SYN sean sigilosos.
    * Sin tener que preocuparse por completar (y desconectarse) un protocolo de enlace de tres vías para cada puerto, los escaneos SYN son significativamente más rápidos que un escaneo TCP Connect estándar.

* Desventajas:

    * Requieren permisos sudo [1] para funcionar correctamente en Linux. Esto se debe a que los escaneos SYN requieren la capacidad de crear paquetes sin procesar (a diferencia del protocolo de enlace TCP completo), que es un privilegio que solo el usuario raíz tiene por defecto.
    * Los servicios inestables a veces se ven interrumpidos por los análisis SYN, lo que podría resultar problemático si un cliente ha proporcionado un entorno de producción para la prueba.

--------------------------------
### Escaneos UDP Scans
-------------------------------

Los escaneos UDP tienden a ser increíblemente lentos en comparación con los diversos escaneos TCP.

Analizar los 20 puertos UDP más utilizados
```
nmap -sU --top-ports 20 <target>
```
--------------------------------
### Escaneos UDP Scans
-------------------------------

Estos escaneos son aún más sigilosos:

1) Escaneo NULL
Solicitud TCP se envía sin ningún indicador establecido.
El host de destino debe responder con un RST si esta cerrado.
```
nmap -sN
```
2) Escaneo FIN
Solicitud con el indicador FIN.
El host de destino debe responder con un RST si esta cerrado.
```
nmap -sF
```
3) Escaneo Árbol de Navidad
Envían un paquete TCP con formato incorrecto.
Si el puerto está abierto, no hay respuesta al paquete con formato incorrecto
```
nmap -sX
```
--------------------------------
### ICMP Network Scanning
-------------------------------

Nmap envía un paquete ICMP a cada posible dirección IP para la red especificada.
```
nmap -sn 192.168.0.1-254
nmap -sn 192.168.0.0/24
```
--------------------------------
### Overview
-------------------------------
Algunas categorías útiles incluyen:

safe:       - No afectará al objetivo
intrusive:  - No seguro: es probable que afecte al objetivo
vuln:       - Escanear en busca de vulnerabilidades
exploit:    - Intento de aprovechar una vulnerabilidad
auth:       - Intente omitir la autenticación para los servicios en ejecución (por ejemplo, inicie sesión en un servidor FTP de forma anónima)
brute:      - Intento de usar la fuerza bruta de las credenciales para ejecutar servicios
discovery:  - Intente consultar los servicios en ejecución para obtener más información sobre la red (por ejemplo, consultar un servidor SNMP).

--------------------------------
### Trabajar con NSE
-------------------------------

Utilización de script:
```
nmap --script=vuln
nmap --script=safe
```
Utilizar un script específico:
```
--script=http-fileupload-exploiter
```
Concatenar múltiples scripts:
```
--script=smb-enum-users,smb-enum-shares
```

Ejemplo de un script con argumentos:
```
nmap -p 80 --script http-put --script-args http-put.url='/dav/shell.php',http-put.file='./shell.php'
```

Ayuda para saber más de los scripts:
```
nmap --script-help <script-name>
```

Link de referencia:
https://nmap.org/nsedoc/

