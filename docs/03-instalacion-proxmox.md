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

## Configuración base del nodo (fase 4)

### Almacenamiento

| ID | Tipo | Contenido | Ruta | Espacio |
|---|---|---|---|---|
| `local` | Directory | Backup, Import, ISO image, Container template | `/var/lib/vz` | 8,4 GiB, 3,8 GiB libres (53 % usado) |
| `local-lvm` | LVM-Thin | Disk image, Container | n/a | 6,5 GiB, sin uso |

- `local` está en la partición raíz del nodo, así que ISOs, plantillas y copias compiten con el sistema por 3,8 GiB. Admite Backup, por lo que sirve como destino de prueba para la fase 9.
- Si las copias se guardan en el mismo disco del laboratorio, un fallo de ese disco afectaría a la vez a las máquinas y a sus copias.
- Espacio libre en el anfitrión: 132 GB en C:, que es donde crece el disco virtual si hay que ampliarlo.

### Red y nombres

- `vmbr0` (192.168.67.10/24) y `vmbr1` (192.168.56.10/24) coinciden con el diagrama de red.
- `hostname -f` devuelve `pve-mesa.lab.test` y resuelve a 192.168.67.10, no a 127.0.0.1.
- El nodo resuelve nombres externos (`deb.debian.org`). Acceso al repositorio de paquetes autorizado: pendiente de confirmar con el profesor.

### Accesos

- Única cuenta usada: `root@pam`. No se han creado más usuarios ni roles.
- La contraseña se guarda en un gestor de contraseñas y no aparece en capturas ni en el repositorio.
- El firewall de Proxmox no se ha activado: no es objetivo obligatorio de la práctica.

### Reparto de recursos (previsión)

| Elemento | CPU | RAM | Almacenamiento | Motivo |
|---|---|---|---|---|
| Nodo Proxmox | 2 vCPU | 5,7 GiB (uso ~1,5 GiB) | 20 GiB (`sda`) | Mínimo para administrar el nodo con margen |
| `srv-aso` | 1 | 1 GiB | 2-3 GiB (LXC) | Servicio Samba ligero |
| `cli-linux` | 1 | 512 MiB | ~2 GiB (LXC) | Solo pruebas del servicio |
| Plantilla, ISO, copias y margen | n/a | ~1 GiB libre | resto de `local-lvm` y `local` | Clonado y copias de prueba |

Las cifras de los invitados son una previsión que se ajustará con las medidas de las fases siguientes.

### Evidencias

- [Almacenamiento en el panel](../evidencias/almacenamiento_en_el_nodo.png)
- [df -h y espacio libre del anfitrión](../evidencias/df-h.png)
- [Nombre del nodo y DNS](../evidencias/nombre_del_nodo_y_DNS.png)

## Conclusion
 
Proxmox VE queda instalado y accesible desde el host por la red de gestión, con las dos redes del diseño configuradas y la virtualización anidada operativa. Pendiente: salida a Internet de los invitados mediante NAT en el nodo (con aprobación docente) y ampliar el disco si el espacio resulta insuficiente.


[Volver al indice](../README.md)
