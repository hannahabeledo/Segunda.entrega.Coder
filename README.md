# Reporte Técnico de Configuración de Laboratorio

- **Alumna:** Hannah Abeledo
- **Máquina virtual:** Laboratorio-coder
- **Sistema invitado:** Ubuntu Desktop 26.04.1
- **Host:** Oracle VirtualBox 7.2

## Objetivo

El objetivo de este laboratorio es construir una máquina de prácticas aislada y segura, donde poder instalar herramientas y hacer pruebas de ciberseguridad sin poner en riesgo la computadora real (el host). Para eso se creó una máquina virtual con Ubuntu en VirtualBox y se le aplicó un hardening básico: red aislada, separación de privilegios entre usuarios, sistema actualizado y un punto de restauración (snapshot).

**Recursos asignados a la VM:**

- **Memoria:** 4 GB de RAM
- **Procesadores:** 2 CPU virtuales
- **Disco:** 25 GB
- **Red:** NAT (configurada antes del primer encendido)
- **Servidor de escritorio remoto:** deshabilitado

## 1. La Fundación: VirtualBox y red aislada (NAT)

Antes de encender la máquina por primera vez se verificó que el Adaptador 1 estuviera conectado en modo NAT.

En modo NAT, VirtualBox actúa como un router privado entre la VM y la red real. La VM recibe una dirección IP interna que solo existe dentro de VirtualBox, y todo su tráfico sale a Internet con la dirección del host. Esto tiene tres consecuencias de seguridad:

- **La VM es invisible en la red de casa u oficina:** ningún otro dispositivo del Wi-Fi puede iniciar una conexión porque no tiene una IP propia en esa red.
- **Aislamiento ante malware o herramientas de prueba:** si en el futuro algo dentro de la VM se comporta mal, no va a poder alcanzar directamente a los demás equipos de la red local.
- **Mantiene acceso a Internet** para descargar actualizaciones, que es lo que se necesita en esta etapa.

El modo que se evitó fue **Bridge**: en ese modo la VM se conecta a la red física como si fuera una computadora más, con su propia IP en el Wi-Fi. Eso la vuelve visible y atacable por cualquier dispositivo de la red, y a la vez le permite a la VM llegar a esos dispositivos. Para un laboratorio de prácticas es justamente lo contrario a lo que se busca. La opción de **Red Interna** aísla todavía más, ya que no tiene salida a Internet, y se reserva para cuando haya que conectar varias VMs entre sí sin tocar la red real.

## 2. Capa de usuarios: separación de privilegios

Durante la instalación se creó la cuenta `hannah`, que queda como **Administrador** (puede usar `sudo`). Después se creó una segunda cuenta, el usuario `Practica-coder`, de tipo **Estándar**, que es la que se usa para el trabajo diario del laboratorio.

Es la aplicación del **principio de mínimo privilegio**: cada usuario o proceso debe tener solo los permisos imprescindibles para su tarea. Si se trabaja siempre como administrador por comodidad, cualquier programa malicioso o comando equivocado se ejecuta con control total del sistema. Con un usuario estándar, si hay un error o un malware, queda limitado a los archivos de ese usuario.

## 3. Capa Linux: permisos y gestión de paquetes

### A. Permisos de archivos con `ls -l`

Se creó un archivo de prueba, se le restringieron los permisos y se los mostró con `ls -l`:

```bash
touch notas-lab
chmod 640 notas-lab
ls -l notas-lab
```

La cadena de permisos `-rw-r-----` se lee en tres bloques: el dueño puede leer y escribir (`rw-`), el grupo solo puede leer (`r--`) y el resto de los usuarios no tiene ningún acceso (`---`). En notación octal es **640** (6 = 4+2 lectura+escritura, 4 = lectura, 0 = nada).

### B. Actualización del sistema

Se actualizaron la lista de paquetes y el sistema completo desde la terminal:

```bash
sudo apt update
sudo apt upgrade -y
```

`apt update` descarga la lista más reciente de paquetes disponibles; `apt upgrade` instala las nuevas versiones. Mantener el sistema actualizado cierra vulnerabilidades ya conocidas, que son las que más explotan los atacantes porque cualquiera puede consultarlas.

## 4. La red de seguridad: snapshot inicial

Con el sistema instalado, actualizado y con los usuarios configurados, se apagó la VM y se creó una instantánea desde el Administrador de Instantáneas de VirtualBox con el nombre **"Clean Install - Hardening applied"**.

En la descripción del snapshot se dejó registrado qué incluye este punto de restauración: Ubuntu actualizado, red NAT, usuario estándar separado del administrador y escritorio remoto deshabilitado.

El snapshot funciona como una máquina del tiempo: guarda el estado exacto del disco y la configuración de la VM en un momento conocido y limpio. Si en una práctica futura se rompe algo, se instala una herramienta que deja el sistema inestable o se infecta la máquina, se puede volver a este punto en segundos, sin reinstalar nada. También permite experimentar con más libertad, sabiendo que existe un estado seguro al que regresar.

### Hardening adicional: servicios innecesarios desactivados

Siguiendo el paso 4 de la consigna, se verificó en **Configuración → Sistema → Escritorio remoto** que la compartición de escritorio y el control remoto estén apagados. Un servicio de acceso remoto que no se usa es una puerta de entrada innecesaria: cada servicio activo amplía la superficie de ataque.

## Conclusión

El laboratorio queda listo para empezar a practicar con cuatro capas de protección:

1. La **red NAT** aísla la VM de la red real.
2. La **separación entre usuario estándar y administrador** limita el impacto de errores o malware.
3. El **sistema está al día** con los parches de seguridad.
4. El **snapshot** garantiza poder volver a un estado limpio ante cualquier problema.
