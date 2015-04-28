---
layout: default
title: "Aprende Puppet – Índice"
canonical: "/es/learning/introduction.html"
---

## Bienvenidos
 Esto es **Aprende Puppet**, una serie de clases acerca del manejo de configuración de sistemas con Puppet Enterprise. Las [instrucciones de instalación](http://docs.puppetlabs.com/pe/latest/install_basic.html) y [referencias al lenguaje de Puppet](http://docs.puppetlabs.com/puppet/latest/reference/lang_summary.html) están disponibles en otra parte el sitio. La idea de esto es brindarte un tour guiado para construir cosas con Puppet. Si tienes buenas referencias acerca de Puppet pero no sabes por dónde empezar, éste es el lugar.

## Obtén la VM
Puppet puede configurar o desconfigurar casi cualquier aspecto de un sistema, entonces, mientras aprendes a utilizarlo es mejor si tienes sistemas de prueba contigo. Para ayudarte en esto, vamos a darte una máquina virtual con Puppet instalado para que puedas experimentar sin miedo!

[Obtén la VM de *Aprende Puppet*](http://info.puppetlabs.com/download-learning-puppet-VM.html)
Mientras esto se baja, ve al [primer capítulo de *Aprende Puppet*](http://docs.puppetlabs.com/es/learning/ral.html). Si tienes problemas para ejecutar la VM, mira [los “Tips para la VM” a continuación]().
La VM de *Aprende Puppet* está disponible en formato .vmx de VMWare y en formato multiplataforma OVF, y ha sido testeada con VMWare Fusion y VirtualBox.

### Información de login
+ Loguéate como **root** con la contraseña **puppet**.
+ La VM está configurada para escribir su dirección IP actual en la pantalla de login cerca de diez segundos después de que inicia. Si prefieres utilizar SSH, espera que se imprima la dirección IP y conéctate por SSH a **root@\<dirección ip\>**.
+ Para ver la consola web de Puppet Enterprise, ve a **https://(la IP de tu VM)** en tu navegador. Loguéate como **puppet@example.com**, con la contraseña **learningpuppet**.
+ **Nota:** Si quieres crear un nuevo usuario en la consola, los e-mails de confirmación contendrán links incorrectos. Puedes trabajar de esta forma copiando y pegando los links en el navegador y corrigiendo el hostname antes de apretar enter, o puedes asegurarte que tu consola esté disponible en un hostname confiable y [sigue las instrucciones de cambio de autenticación de hostname](http://docs.puppetlabs.com/pe/latest/trouble_console-db.html#console-account-confirmation-emails-have-incorrect-links).

Si prefieres crear tu propia VM en lugar de bajar una de la web, puedes imitarla fácilmente: este es un sistema CentOS mínimo con un hostname de “learn.localdomain”, [Puppet Enterprise](http://puppetlabs.com/puppet/puppet-enterprise/) instalado, e **iptables** inhabilitado. También tiene instalados los modos del lenguaje Puppet para Vim y Emacs, pero eso no es estrictamente necesario.

Para comenzar, no necesitarás separar VMs agente y master, esta VM puede cumplir ambos roles. Cuando llegues a los ejercicios de agente/master, duplicaremos el sistema en un nuevo nodo agente.

## Contenidos
+ **Parte uno: Puppet sin servidor**
	+ [Recursos y la RAL](http://docs.puppetlabs.com/es/learning/ral.html): Aprende sobre los bloques fundamentales de construcción de la configuración de sistemas.
	+ [Manifiestos](http://docs.puppetlabs.com/es/learning/manifests.html): Comienza a controlar tu sistema escribiendo código “real” de Puppet.
	+ [Orden](http://docs.puppetlabs.com/es/learning/ordering.html): Aprende sobre las dependencias y eventos de refresh, controla las relaciones entre recursos, y descubre el patrón de diseño fundamental de Puppet.
	+ [Variables, condicionales y facts](http://docs.puppetlabs.com/es/learning/variables.html): Haz a tus manifiestos más versátiles leyendo la información del sistema.
	+ [Módulos y clases (parte uno)](http://docs.puppetlabs.com/es/learning/modules1.html): Comienza a construir tus manifiestos en módulos autocontenidos.
	+ [Templates](http://docs.puppetlabs.com/es/learning/templates.html): Utiliza ERB para hacer tus archivos de configuración tan flexibles como tus manifiestos.
	+ [Clases parametrizadas (Módulos, parte dos)](http://docs.puppetlabs.com/es/learning/modules2.html): Aprende cómo pasar parámetros a clases y hacer más adaptables tus módulos.
	+ [Tipos definidos](http://docs.puppetlabs.com/es/learning/definedtypes.html): Modela trozos repetibles de configuración agrupando recursos básicos en super-recursos.

+ **Parte dos: Puppet Master/Agente**
	+ [Preparar una VM agente](http://docs.puppetlabs.com/es/learning/agentprep.html): Prepara tus herramientas para los próximos capítulos con el tutorial paso a paso.
	+ [Puppet Agent/Master básico](http://docs.puppetlabs.com/es/learning/agent_master_basic.html): Recorre el workflow de agente/master: firma un certificado de nodo agente, elige las clases de cada nodo, obtiene y aplica un catálogo.

## Tips para la VM
### Importar la VM a VirtualBox
Hay varias particularidades y consideraciones a tener en cuenta al importar una VM a VirtualBox:

+ Si estás utilizando VirtualBox con la versión OVF de la VM, elige “Import Appliance” desde el menú de File y ve al archivo **.ovf** incluído en el paquete. Otra alternativa es arrastrar el archivo OVF a la ventana principal de VirtualBox.

**No utilices** el “New Virtual Machine Wizard” y selecciona como disco el archivo **.vmdk**; las máquinas creadas de esta manera harán un kernel panic durante el booteo.

+ Si durante el booteo encuentras el mensaje “registered protocol family 2”, puede que necesites ir a las opciones de configuración de “System” de la VM y chequear la opción “Enable IO APIC”. Muchos usuarios pueden dejar la opción IO APIC deshabilitada sin problemas; al momento no sabemos qué es lo que causa este problema.
+ La VM debería funcionar sin modificaciones en versiones 4.x de VirtualBox. Sin embargo, en las versiones 3.x puede fallar al importar, con un error del tipo “Failed to import appliance. Error reading ‘filename.ovf’: unknown resource type 1 in hardware item, line 95.” Si ves este error, puedes actualizar tu copia de VirtualBox o editar el archivo .ovf y recalculando el hash sha1 [como está descrito aquí](http://mattiasgeniar.be/2012/03/31/importing-the-puppet-learning-vm-into-virtualbox-unknown-resource-type-in-hardware-item/). Gracias Mattias por esta solución provisoria.

### Importar la VM a Parallels Desktop
Parallels Desktop 7 en OS X pueden importar la versión VMX de esta VM, pero requiere de una configuración extra antes que pueda ejecutarse:

1. Primero, convierte la VM, pero no la inicies.
2. Navega por el menú de Virtual Machine, elige Configure->Hardware->Hard Disk 1 y cambia su ubicación de SATA a IDE (ej.: IDE 0:1)
3. Ahora puedes iniciar la VM.

Si intentas iniciar la VM sin cambiar la ubicación del disco, probablemente hará un kernel panic.

### Configurar las redes virtuales
#### Con VMware
Si estás utilizando un producto de virtualización de VMware, puedes dejar la red de la VM en NAT, que es su modo por defecto; esto te permitirá contactarte con: tu servidor, cualquier otra VM que se ejecute en modo NAT, la red local, e internet externa; la única restricción es que las computadoras fuera de tu servidor no pueden iniciar conexiones a tu VM. Si eventualmente necesitas que otras computadoras puedan contactarse con tu VM, puedes cambiar su modo de red a Bridged.

#### Con VirtualBox
El modo NAT de VirtualBox es muy limitado, no funcionará en lecciones más avanzadas de agent/master. **Debes cambiar el modo de red de VM a “Bridged Adapter” antes de iniciar la VM por primera vez**.

![](img/vbox_network.png)

![](img/vbox_network_bridged.png)

Si por algún motivo no puedes exponer la VM en tu red local, o no estás en una red con DHCP, debes configurar la VM para que tenga **dos** adaptadores de red: uno en modo NAT (para acceder a la red local y a internet) y otro en modo “Host Only Adapter” (para acceder al servidor y otras VMs). También tendrás que asignar una dirección IP al adaptador “Host Only” manualmente, o configurar el servidor DHCP de VirtualBox.

[Entra aquí para más información acerca de los modos de red de VirtualBox](http://www.virtualbox.org/manual/ch06.html), y [aquí para más información acerca del servidor DHCP de VirtualBox](http://www.virtualbox.org/manual/ch08.html#vboxmanage-dhcpserver).


Para asignar manualmente una dirección IP a un adaptador *Host Only*:
+Encuentra la dirección IP del servidor en las preferencias de VirtualBox, ve a la sección “Network”, haz doble click en la red *Host Only Adapter* que estés usando, ve a la pestaña “Adapter” y toma nota de la dirección IP en el campo “IPv4 Address”.
+Una vez que tu VM esté ejecutándose, loguéate en la consola y ejecuta **ifconfig eth1 <NEW IP ADDRESS>**, donde **<NEW IP ADDRESS>** es una dirección IP no reclamada en la subred de la red “Host Only”.
