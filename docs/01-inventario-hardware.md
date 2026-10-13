# Inventario investigado y viabilidad

## Contexto de la observacion

* Fecha y sistema utilizado: 13/10/2026 Los comandos lscpu, free -h, lsblk, df -h, ip -br addr, lspci -nnk y systemd-detect-virt los he ejecutado en el PC del aula. dmidecode los he sacado desde una ISO Debian live.
* Modalidad, equipo anfitrion y VM VirtualBox si procede: Modalidad física, el equipo anfitrión es un PC de clase, pero seguramente acabemos montando Proxmox en una VM de VirtualBox
* Version de Proxmox prevista: La ultima versión de Proxmox.

## Inventario inicial

|Area|Dato observado y evidencia|Especificacion y fuente|Limite o diferencia|Justificacion|
|-|-|-|-|-|
| CPU | Intel Pentium G4560 a 3,5 GHz, x86_64, 2 nucleos y 2 hilos por nucleo 4 CPUs max 3500 MHz. Comando lscpu. | Ficha del Intel Pentium G4560 en Intel ARK.: 2 nucleos, 4 hilos, 3,50 GHz base, VT-x, VT-d. | Solo 2 nucleos fisicos, los 4 hilos no equivalen a 4 nucleos. Lo observado coincide con la ficha. | Cumple la arquitectura que pide Proxmox, pero va justa para varias cargas a la vez |
| RAM | free -h marca 7,6 GiB. dmidecode: 2 modulos Crucial CT4G4DFS824A de 4 GB DDR4 a 2400 MT/s (ChannelA-DIMM1 y ChannelB-DIMM1), otros 2 slots vacios. Error Correction: None. | Manual de la placa ASUS PRIME B250M-A: 4 slots DDR4 sin buffer, maximo 64 GB, 2400/2133 MHz, sin ECC | 8 GB sin ECC. Intel  marca ECC como soportado en el G4560, pero la placa y los modulos son non-ECC, asi que el limite lo pone la placa y no la CPU. Linux ve 7,6 GiB de 8 instalados; supongo que el resto lo reservan el firmware. | Doble canal a 2400 MT/s, que es lo maximo que admite la placa, y 2 slots libres para ampliar |
| Almacenamiento | sda de 447,1 GiB, SATA, ROTA 0 (SSD), una particion sda1 en / con 65 GiB usados y 353 GiB libres. Comandos lsblk y df -h. | Documentacion de Proxmox VE: recomienda almacenamiento rapido y redundante, mejor con SSD. Ficha de la placa: 6 puertos SATA3 y 2 slots M.2. | Un solo disco, sin RAID ni redundancia. La placa tiene mas conexiones, asi que se puede anadir otro disco. | Capacidad de sobra y al ser SSD ayuda al clonar y a las copias |
| Red | Realtek RTL8111/8168 Gigabit. eth0 aparece DOWN en ip -br addr porque no hay cable de red conectado. | Ficha de la placa: Realtek RTL8111H, 1 puerto Gigabit. Proxmox VE: minimo una tarjeta de red, recomendado Gigabit y redundante. | Una sola interfaz fisica de 1 Gbit, sin redundancia. | Solo tenemos una sola tarjeta de red |
| Plataforma | Placa ASUS PRIME B250M-A, equipo APD ALDA+. BIOS American Megatrends 0810 del 09/08/2017 con UEFI soportado. Chipset serie 200 | Ficha de la placa: socket LGA1151, chipset B250, BIOS UEFI AMI. Intel lista el B250 entre los chipsets compatibles con el G4560. | Limite de la placa para mejorar el procesador en un futuro | BIOS de 2017, conviene mirar si hay version mas nueva |

## 1\. Puedo instalar Proxmox en este equipo?

Contrastar el inventario con requisitos oficiales de la version prevista.
Responder por separado y justificar con datos y fuentes:

* Puedo instalar y arrancar Proxmox: si, con una condicion. El equipo es x86_64 de 64 bits, tiene 7,6 GiB de RAM y un SSD de 447,1 GiB con una tarjeta de red Gigabit, y eso cubre los requisitos de evaluacion de Proxmox en la documentacion oficial. La condicion es que habria que activar la virtualizacion en la BIOS.
* Puedo ejecutar el servidor ASO y el cliente del proyecto: si se podria ejecutar ya que al nodo le podriamos dar 2GiB, al servidor ASO 1GiB, al cliente otras 2 GiB y dejar 2,6 GiB de margen. Pero esto estaria muy justo.

## 2\. Que limitaciones tendre con este hardware?

Analizar RAM, CPU, disco y red para las cargas previstas. Identificar lo
medido y lo estimado. En VirtualBox, investigar host real y VM: version,
RAM reservada al anfitrion, vCPU, VDI, espacio fisico y red local.
No se utiliza el Proxmox del centro como recurso.

### Reparto de recursos

|Elemento|CPU/vCPU|RAM|Disco|Justificacion|
|-|-|-|-|-|
|Proxmox|Comparte los 4 hilos | 2,0 GiB | 40 GiB | Se recomienda dejar al menos 2 GB para el sistema y sus servicios. |
|srv-aso| 2 vCPU | 1,0 GiB | 40 GiB | Un linux sin entorno grafico consume poco. |
|cli-linux|2 vCPU | 2,0 GiB | 40 GiB | Cliente con entorno grafico necesita mas memoria |
|Plantilla, ISOs y copias|No aplica|No aplica|130 GiB | 20 de plantilla, 10 de ISOs y 100 de copias locales |
|Margen de operacion|Resto | 2,6 GiB | 197,1 GiB | Espacio para snapshots, clones y crecimiento |

Mostrar calculos de RAM y disco, reparto de vCPU, unidades y supuestos.
Explicar las cargas que pueden funcionar simultaneamente.

Calculo de RAM: 7,6 - (2,0 + 1,0 + 2,0) = 2,6 GiB de margen. La swap de 511 MiB que sale en free -h es del sistema actual y no la cuento como RAM.

Calculo de disco: 40 + 40 + 40 + 130 = 250 GiB asignados. 447,1 - 250 = 197,1 GiB de margen.

Supuestos: Discos de las VMs con thin provisioning (la suma de lo asignado no es espacio realmente ocupado) y el SSD entero dedicado a Proxmox.

CPU: se asignan 4 vCPU (2 y 2) sobre 4 hilos que son 2 nucleos fisicos. Las vCPU no son nucleos exclusivos, se reparten el tiempo de CPU. El servidor y el cliente pueden funcionar a la vez.

|Limitacion|Evidencia o estimacion|Consecuencia|Medida y limite que permanece|
|-|-|-|-|
| Solo 8 GB de RAM y sin ECC | Observado en free -h y dmidecode (Error Correction: None) | No cabe una tercera VM pesada y un error de memoria no se corregiria | Usar LXC cuando se pueda y no encender todo a la vez. No resuelve la falta de ECC. |
| CPU de 2 nucleos | Observado en lscpu. Pienso que clonar o comprimir copias subira mucho la carga. | Las copias y clones pueden dejar el cliente lento | Hacer las copias con el cliente apagado. Los nucleos siguen siendo los mismos. |
| Un solo SSD sin RAID | Observado en solo existe sda | Si falla el disco se pierde todo, y una copia en el mismo disco no sirve | Guardar las copias en otro almacenamiento. |

## 3\. Que componentes son mas criticos en mi servidor y por que?

Ordenar al menos tres componentes para este equipo y su servicio.
Distinguir cuello de botella, disponibilidad e integridad de los datos.

|Prioridad y componente|Funcion|Efecto si es insuficiente|Efecto si falla|Evidencia y justificacion|
|-|-|-|-|-|
| 1. RAM | Guarda Proxmox, el servidor ASO y el cliente a la vez | Las VMs van lentas, se usa swap y el sistema puede matar procesos | Se cae todo el nodo y sin ECC un error de memoria podria pasar sin avisar | 7,6 GiB totales y solo 2,6 GiB de margen. Es lo que antes me limita (cuello de botella). |
| 2. SSD  | Guarda el sistema, los discos de las VMs, la plantilla y las copias | Si se llena no se pueden clonar VMs ni hacer copias | Se pierde todo, incluida la copia si esta en el mismo disco | Un solo disco sin RAID. Es el que mas daño haria si falla, por integridad de datos. |
| 3. CPU | Reparte el tiempo entre las VMs y hace tareas pesadas como comprimir | Todo va lento cuando coinciden clon, copia y uso del cliente | El equipo no arranca | 2 nucleos y 4 hilos. Limita el rendimiento pero no pone en riesgo los datos. |


Componente que mejoraria primero y motivo para priorizarlo frente a otro:
la RAM porque hay 2 slots libres y dmidecode indica un maximo de 64 GB, asi que con dos modulos mas llegaria a 16 GB sin cambiar nada mas. La priorizo frente al disco porque el problema de la RAM no se arregla con software, mientras que el riesgo del disco lo puedo reducir sacando las copias a otro almacenamiento.


## Fuentes tecnicas

Consultar al menos tres fuentes primarias, incluyendo Proxmox y
fabricante o plataforma virtual. Relacionarlas con las tres respuestas.

|Fuente y URL|Version/modelo y fecha de consulta|Afirmacion respaldada|Aplicacion al nodo|
|-|-|-|-|
| Documentacion de Proxmox VE, requisitos del sistema. https://pve.proxmox.com/pve-docs/chapter-pve-installation.html | Proxmox VE. Fecha de consulta: 13/10/2026 | Requisitos minimos y recomendados de CPU, RAM, disco y red | Respuestas 1 y 2 |
| Pagina de soporte de la ASUS PRIME B250M-A. http://asus.com/latin/supportonly/prime%20b250m-a/helpdeskhelpdeskhelpdeskhelpdesk_cpu/ | PRIME B250M-A. Fecha de consulta: 13/10/2026  | Slots de memoria, maximo de RAM y arranque UEFI | Respuestas 1 y 3 |
| Ficha del Intel Pentium G4560 en Intel ARK. https://ark.intel.com | Pentium G4560. Fecha de consulta: 13/10/2026 | Nucleos, hilos, VT-x y soporte de ECC | Respuestas 1, 2 y 3 |

## Dictamen de viabilidad

Apto, apto con condiciones o no apto para la propuesta actual: 

Sintetizar las tres respuestas y las condiciones pendientes de comprobar.

El equipo cumple en arquitectura, RAM, disco y red, y el reparto deja 2,6 GiB de RAM y 197,1 GiB de disco de margen. La CPU de 2 nucleos y los 8 GB de RAM obligan a no ejecutar cargas pesadas a la vez. La condicion pendiente es VT-x activado en la BIOS 

## Evidencias y revision docente

Enlazar salidas revisadas desde ../evidencias/. Un listado automatico no
sustituye la investigacion. Registrar la pregunta de defensa y respuesta.

[Volver al indice](../README.md)

