# 02-diagrama-red

# Diseño de red
## Elementos de diseño
|Elemento |Valor |
|---|---|
|Nombre del nodo Proxmox |pve-mesa |
|Interfaz de gestión |Segunda interfaz (adaptador Host-only)->vmbr1 (192.168.56.10/24) |
|Bridges y Proxmox |vmbr0 (salida NAT 10.0.67.10/24) y vmbr1 (laboratorio) |
|Red o VLAN de laboratorio |192.168.56.0/24 (VMnet1), sin VLAN |
|IPs de las VMs/LXC |srv-aso 192.168.56.20 · cli-linux 192.168.56.30 |
|Gateway y DNS |Invitados: gateway 192.168.56.10. Nodo: gateway 10.0.2.2, DNS 10.0.2.3 |

## Tabla de direccionamiento
|Elemento |Nombre |IP | Función |Recursos |
|---|---|---|---|---|
|Interfaz solo-anfitrión del host |(host) |DHCP de la red real |Acceso a Internet del equipo, ajena al laboratorio |N/A |
|Nodo Proxmox |pve-mesa |192.168.56.10/24 (vmbr1) y 10.0.67.10/24 (vmbr0) |Hipervisor |2 vCPU, 6-8 GiB RAM, 20 GiB disco |
|Servidor ASO |srv-aso |192.168.56.20/24 |Servicio principal |1-2 vCPU, 1-2 GiB RAM |
|Cliente |cli-linux |192.168.56.30/24 |Pruebas |1 vCPU, 512 MiB-1 GiB RAM |

## Interfaces
|Adaptador VirtualBox | Interfaz en Proxmox (a confirmar) |Bridge |IP |Uso |
|---|---|---|---|---|
|Adaptador1·NAT |nic0 |vmbr0 |10.0.67.10/24, gateway 10.0.67.2 |Salida del nodo |
|Adaptador2·Solo-anfitrión |nic1 |vmbr1 |192.168.56.10/24 (sin gateway) |Gestión y laboratorio |

## Puertos necesarios
|Origen |Destino |Puerto |Motivo |
|---|---|---|---|
|Host |pve-mesa |TCP 8006 |Panel web de Proxmox |
|Host |srv-aso |TCP 22 |SSH y Ansible |
|cli-linux |srv-aso |TCP 445 |Samba (SMB) |
|Invitados |DNS 10.0.2.3 |53 |Resolución de nombres |
|Invitados (vía nodo) |Repositorios Linux |TCP 80/443 |Instalación de paquetes |

## Medidas de aislamiento
- No se usa red en modo bridge (VMnet0): el laboratorio no se conecta a la red física.
- DHCP desactivado en VMnet1: todas las IP son estáticas.
- Modo promiscuo solo en el adaptador de laboratorio.
- Sin gateway en la interfaz solo-anfitrión del host.

## Límites de acceso
- El panel de Proxmox (8006) y el SSH solo son accesibles desde el host, VMnet1.
- cli-linux solo necesita llegar al puerto 445 de srv-aso.
- El laboratorio no es accesible desde la red del aula ni desde otros equipos.
- Las pruebas se limitan a las máquinas del laboratorio propio.

## Supuestos
- La subred NAT (VMnet8) la asigna VMware y es distinta en cada equipo; se leerá de mi instalación y se sustituirá X. El gateway y el DNS del NAT de VMware suelen ser la dirección .2; la IP del nodo se elegirá fuera del rango DHCP de VMnet8.
- VMnet1 se configurará manualmente en el editor de redes virtuales con subred 192.168.56.0/24 y DHCP desactivado.
- El host obtiene su IP por DHCP de la red real; se comprobará que no usa 192.168.56.0/24 ni la subred NAT de VMware.
- El diseño es una propuesta que todavía no se ha ejecutado en mi equipo.
- Los invitados salen a Internet mediante enrutamiento y NAT en el nodo.
- La guía de la asignatura se basa en VirtualBox; el uso de VMware se consultará con el profesor.










./README.md)
