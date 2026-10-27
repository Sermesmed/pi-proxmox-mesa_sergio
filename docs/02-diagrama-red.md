# 02-diagrama-red

# Diseño de red
## Elementos de diseño
|Elemento |Valor |
|---|---|
|Nombre del nodo Proxmox |pve-mesa |
|Interfaz de gestión |enp0s8->vmbr1 (192.168.56.10/24 |
|Bridges y Proxmox |vmbr0 (salida NAT 10.0.2.15/24) y vmbr1 (laboratorio) |
|Red o VLAN de laboratorio |192.168.56.0/24, sin VLAN |
|IPs de las VMs/LXC |srv-aso 192.168.56.20 · cli-linux 192.168.56.30 |
|Gateway y DNS |Invitados: gateway 192.168.56.10. Nodo: gateway 10.0.2.2, DNS 10.0.2.3 |

## Tabla de direccionamiento
|Elemento |Nombre |IP | Función |Recursos |
|---|---|---|---|---|
|Interfaz solo-anfitrión del host |(host) |192.168.56.1/24 |Navegador, SSH y Ansible |N/A |
|Nodo Proxmox |pve-mesa |192.168.56.10/24 (vmbr1) y 10.0.2.15/24 (vmbr0) |Hipervisor |2 vCPU, 6-8 GiB RAM, 80 GiB disco |
|Servidor ASO |srv-aso |192.168.56.20/24 |Servicio principal |1-2 vCPU, 1-2 GiB RAM |
|Cliente |cli-linux |192.168.56.30/24 |Pruebas |1 vCPU, 512 MiB-1 GiB RAM |

## Interfaces
|Adaptador VirtualBox | Interfaz en Proxmox (a confirmar) |Bridge |IP |Uso |
|---|---|---|---|---|
|Adaptador1·NAT |enp0s3 |vmbr0 |10.0.2.15/24, gateway 10.0.2.2 |Salida del nodo |
|Adaptador2·Solo-anfitrión |enp0s8 |vmbr1 |192.168.56.10/24 (sin gateway) |Gestión y laboratorio |

## Puertos necesarios
|Origen |Destino |Puerto |Motivo |
|---|---|---|---|
|Host |pve-mesa |TCP 8006 |Panel web de Proxmox |
|Host |srv-aso |TCP 22 |SSH y Ansible |
|cli-linux |srv-aso |TCP 445 |Samba (SMB) |
|Invitados |DNS 10.0.2.3 |53 |Resolución de nombres |
|Invitados (vía nodo) |Repositorios Linux |TCP 80/443 |Instalación de paquetes |

## Medidas de aislamiento
- Red solo-anfitrión sin bridge a la red física del aula.
- DHCP desactivado: todas las IP son estáticas.
- Modo promiscuo solo en el adaptador de laboratorio.
- Sin gateway en la interfaz solo-anfitrión del host.

## Límites de acceso
- El panel de Proxmox (8006) y el SSH solo son accesibles desde el host, por la red solo-anfitrión.
- cli-linux solo necesita llegar al puerto 445 de srv-aso.
- El laboratorio no es accesible desde la red del aula ni desde otros equipos.
- Las pruebas se limitan a las máquinas del laboratorio propio.

## Supuestos
- Las direcciones son las del ejemplo de la guía y se cambiarán si colisionan con una VPN u otra red del host.
- El diseño es una propuesta que todavía no se ha ejecutado en mi equipo.
- Los nombres de las interfaces del nodo dependen de la versión de Proxmox.
- Los invitados salen a Internet mediante enrutamiento y NAT en el nodo.



















[Volver al indice](../README.md)
