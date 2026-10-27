# Bitacora de sesiones

La bitacora es el registro cronologico del trabajo. Anade una entrada al final
de cada sesion y conserva las anteriores. Escribe datos comprobables y enlaza
las evidencias utiles; evita frases generales como "he trabajado en Proxmox".

## Sesion 01 - 06/10/2026

- Objetivo: iniciar GitHub y la documentacion Markdown.
- Acciones realizadas: cree el repositorio individual en GitHub.  Añadi la estructura obligatoria de carpetas (docs, scripts, ansible y evidencias) con los documentos Markdown de la plantilla. He publicado el primer commit y entregue la URL del repositorio.
- Decisiones y motivos: he eleegido la modalidad fisica, porque dispongo de un equipo que se puede usar para instalar Proxmox directamente.
- Pruebas y resultados: he comprobado en la web de GitHub que aparece el primer commit con toda la estructura y que los enlaces del indice del README abren cada documento.
- Documentos o evidencias: [Ficha inicial](00-ficha-inicial.md).
- Incidencias y solucion o estado pendiente: ninguna incidencia al crear y publicar el repositorio.
- Siguiente paso: investigar el hardware del equipo con los comandos de inventario y redactar el inventario y la viabilidad.

Anadir una entrada con esta estructura en cada sesion. El historial de
GitHub registra los commits; no hace falta incluir el hash del propio
commit dentro de su entrada de bitacora.

## Sesion 02 - 13/10/2026

- Objetivo: investigacion de componentes de nuestro equipo fisico y comprobacion de su viabilidad
- Acciones realizadas: he ejecutado en el PC lscpu, free -h, lsblk, df -h, ip -br addr, lspci -nnk y systemd-detect-virt. Desde una ISO Live Debian he sacado dmidecode (system, baseboard, bios y memory). He consultado  los requisitos y la descarga de Proxmox VE, la ficha de Intel del G4560, la de la placa ASUS PRIME B250M-A.
- Decisiones y motivos: modalidad fisica, porque el equipo cumple los requisitos de Proxmox VE (x86_64, unos 8 GiB de RAM y SSD de 447 GiB). La RAM es el componente que mejoraria primero, ya que hay 2 slots libres y la placa admite hasta 64 GB.
- Pruebas y resultados: esperaba ver VT-x activo. lscpu muestra VT-x  pero la linea Itlb multihit dice KVM: Mitigation: VMX disabled, asi que hay que activarlo en  la BIOS systemd-detect-virt devuelve none, por lo que es hardware fisico.
- Documentos o evidencias: [Inventario](01-inventario-hardware.md)
- Incidencias y solucion o estado pendiente: Intel indica que el G4560 admite ECC, pero la placa y los modulos no, asi que no hay ECC.
- Siguiente paso: diseño de la red

## Sesion 03 - 20/10/2026

- Objetivo: Diseño de nuestra red en Virtualbox para instalar Proxmox
- Acciones realizadas: He tenido que pensar la estructura de la red y sin configurar nada todavia he hecho el diseño de la red para posteriormente aplicar este diseño.
- Decisiones y motivos: Aqui las unicas decisiones que he tomado han sido los parametros de red como IPs, poniendo los que yo he pensado que eran mas convenientes.
- Pruebas y resultados: Aqui no pongo nada ya que en esta sesion no he tenido que realizar ninguna prueba.
- Documentos o evidencias: [Diagrama de red](docs/02-diagrama-red.md)
- Incidencias y solucion o estado pendiente: En esta sesion no he tenido ninguna incidencia
- Siguiente paso: Instalacion de Proxmox

## Sesion 04 - 27/10/2026

- Objetivo: Instalacion de Proxmox en una Maquina Virtual en Virtualbox
- Acciones realizadas: He tenido que instalar Proxmox con la configuracion y adaptadores de red correspondientes. Una vez instalado he modificado el archivo de configuracion de las interfaces creando dos puentes y enlazandolos a los dos adaptadores de red. Por ultimo he comprobado que me he podido conectar al panel web de Proxmox.
- Decisiones y motivos: Las decisiones que he tomado han sido la asignacion de recursos necesarios para el correcto funcionamiento, dos adaptadores de red (nat y solo anfitrion) y por ultimo he configurado algunas de las IP que propuse en la sesion 02.
- Pruebas y resultados: Aqui la unica prueba que he hecho ha sido poder conectarme al panel web de Proxmox desde mi equipo anfitrion con la IP configurada de la red solo anfitrion.
- Documentos o evidencias: [Instalacion de Proxmox](/docs/03-instalacion-proxmox.md)
- Incidencias y solucion o estado pendiente: En esta sesion no he tenido ninguna incidencia
- Siguiente paso: Configuracion base del nodo para administrarlo y que sea seguro.


[Volver al indice](../README.md)
