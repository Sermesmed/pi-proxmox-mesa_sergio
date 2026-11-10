# 03-instalacion-proxmox
 
## Objetivo
 
Instalar Proxmox VE en una máquina virtual de VMware (modalidad virtualizada), dejar accesible el panel web desde el host y verificar la red de gestión y el almacenamiento.
 
## Procedimiento
 
1. Creada la VM `pve-mesa` en VMware sobre un host Windows 11: 2 vCPU, 6 GB de RAM (5,74 GiB visibles), disco de 20 GiB, arranque BIOS legacy y virtualización anidada activada.
2. Adaptador 1 en NAT (VMnet8) y adaptador 2 en Host-only (VMnet1, subred 192.168.56.0/24 sin DHCP).
3. Instalado Proxmox VE con la ISO 9.2-1 (instalador gráfico, sistema de ficheros por defecto con LVM). Tras actualizar, `pve-manager` es la 9.2.2 sobre Debian 13 (trixie), kernel 7.0.2-6-pve.
4. Red de gestión durante la instalación: hostname `pve-mesa.lab.test`, `vmbr0` sobre `nic0` con 192.168.67.10/24, gateway y DNS 192.168.67.2.
5. Añadido `vmbr1` sobre `ens37` con 192.168.56.10/24 y sin gateway, para la red de laboratorio.
6. Comprobado el acceso al panel desde el host en `https://192.168.56.10:8006` y por SSH.
| Elemento | Valor |
|---|---|
| Nodo | `pve-mesa` |
| Versión del instalador | 9.2-1 |
| `pve-manager` | 9.2.2 |
| Disco | 20 GiB (`sda`) |
| Almacenamiento `local` | Directorio, 8,3 GiB (ISOs y plantillas) |
| Almacenamiento `local-lvm` | LVM-thin, 6,5 GiB (discos de VMs y LXC) |
| `vmbr0` | 192.168.67.10/24 (gateway 192.168.67.2) |
| `vmbr1` | 192.168.56.10/24 (red de laboratorio) |
 
## Comandos usados
 
```bash
ssh root@192.168.56.10
hostnamectl
ip -br addr
ip route
ifreload -a
pveversion
df -h
lsblk
pvesm status
ls /dev/kvm
```
 
## Evidencias
 
- [Resumen del nodo en el panel](../evidencias/Captura_Proxmox_Web.png)
- [Acceso por SSH](../evidencias/Captura_SSH_Powershell.png)
- [hostnamectl](../evidencias/Captura_hostnamectl_SSH.png)
- [Direcciones, rutas, versión y disco](../evidencias/Captura_info_SSH.png)
Resultado: el nodo responde en ambas redes, la ruta por defecto sale solo por `vmbr0` y `/dev/kvm` existe, así que la virtualización anidada funciona.
 
## Incidencias
 
- **`ifreload -a` fallaba** con errores en las líneas de `vmbr1`. Causa: había escrito `auto vmbr1 inet static` en una sola línea. Solución: separarla en `auto vmbr1` e `iface vmbr1 inet static`.
- **VMnet1 sin la IP esperada**: tenía una IP automática 169.254.x.x y la 192.168.56.1 estaba asignada a VMnet10, aunque el editor de redes mostraba bien las subredes. Causa no determinada. Solución: reasignar a mano las IPs de VMnet1 (192.168.56.1/24) y VMnet10 (192.168.10.1/24) con PowerShell.
- **Contraseña de `root` olvidada**. Solución: restablecida desde GRUB arrancando con `init=/bin/bash` y `passwd root`. A partir de ahora se guarda en un gestor de contraseñas.
- **Repositorio Enterprise activo sin suscripción**: el panel lo avisa. Pendiente de cambiarlo según indique el profesor.
- **Disco de 20 GiB**, menor que los 80 GiB previstos en el diseño. Se deja así de momento; para el servicio se usarán preferentemente contenedores LXC.
## Conclusion
 
Proxmox VE queda instalado y accesible desde el host por la red de gestión, con las dos redes del diseño configuradas y la virtualización anidada operativa. Pendiente: salida a Internet de los invitados mediante NAT en el nodo (con aprobación docente) y ampliar el disco si el espacio resulta insuficiente.


[Volver al indice](../README.md)
