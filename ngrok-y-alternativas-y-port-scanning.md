Ngrok (y alternativas) y Port Scanning

Si sos de los que le interesa buscar objetivos (para hacer pruebas de hacking ético) sabrás que a veces no es fácil encontrarlos.

Encontrar posibles objetivos para posteriores ataques a veces no está al alcance de la mano...

La idea es simple, como sabrán existen servicios que exponen nuestra LAN a la WAN (cuando se está atras de una CGNAT de nuestro ISP por ej), herramientas como Ngrok, serveo, localtunnel, etc. Para más info busca sobre los mencionados.

Ahora bien, utilizando de ejemplo ngrok este genera un host y un puerto aleatorio para redireccionarlo al servicio de la LAN que quieras sea accesible desde la WAN...osea un port tunneling:

`port_lan:host_ngrok:port_ngrok_wan`

  
Ahora como podemos obtener host\_ngrok:port\_ngrok\_wan tiene un servicio X escuchando detrás? Simple nmap al host:puerto y ver que está escuchando.  
  
Meto un código que armé para que no quede solo chamuyo:

`#!/bin/bash`

`host=$1`

`start_port=$2`

`end_port=$3`

`service=$4 if [ -z "$host" ] || [ -z "$start_port" ] || [ -z "$end_port" ] || [ -z "$service" ] then    echo "Usar: $0 <host> <start port range> <end port range> <servicio>"    exit 1 fi echo "escaneando $host ......" for (( port=$start_port; port<=$end_port; port++ )) do    timeout 1 bash -c "echo >/dev/tcp/$host/$port" 2>/dev/null && nmap -sS -sV -p $port $host | grep -q "open  $service" && echo "$port -> $service" && echo "$host:$port" >> scan_result.log done`

  
El uso está más que claro. Para mi ejemplo vamos a buscar hosts con servicios ssh detrás  
  
Al ejecutable le llamamos scan\_service.sh  
  
De haber usado ngrok alguna que otra vez, los puertos que asigna son aleatorios y generalmente van del 12000 al 18000 para por ej el host 0.tcp.sa.ngrok.io  
  
Una forma de saberlo es iniciar ngrok, en diferentes regiones y ver que puerto asigna, para mi ejemplo me asignó puerto 17920, y escanear puertos cercanos a este, por ej:  
  
`$ ./scan_service.sh 0.tcp.sa.ngrok.io 17000 18000 ssh`  
  
Si encuentra alguno lo guardará en el archivo log.  
  
Bueno después es usar vuestra imaginación que hacer con esos hosts:ports. Para el caso de servicios ssh atacarlos por fuerza bruta por ej.  
  
Por ej  
  
`> sudo ./scan_ports.sh 0.tcp.sa.ngrok.io 17919 17929 ssh escaneando 0.tcp.sa.ngrok.io ...... 17920 -> ssh 17929 -> ssh`  
  
En fin esto iba más que nada de donde buscar posibles objetivos. No más que eso. Piénsenlo que podemos escanear servidores web, de correo, etc...  
  
Saludos!!

Tags: security
