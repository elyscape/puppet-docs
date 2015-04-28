---
layout: default
title: "Aprende Puppet – Orden de recursos"
canonical: "/es/learning/ordering.html"
---

## Desorden

Echemos un vistazo a uno de los manifiestos de la página anterior:

 	   # /root/training-manifests/2.file.pp

	    file {'/tmp/test1':
	      ensure  => present,
	      content => "Hi.",
	    }

	    file {'/tmp/test2':
	      ensure => directory,
	      mode   => 644,
	    }

	    file {'/tmp/test3':
	      ensure => link,
	      target => '/tmp/test1',
	    }

	    notify {"I'm notifying you.":}
	    notify {"So am I!":}

Cuando lo ejecutamos, los recursos no fueron sincronizados en el orden que los habíamos escrito: **/tmp/test1,/tmp/test3, /tmp/test2, So am I!**, y **I'm notifying you**.

Como hemos mencionado en el capítulo anterior, Puppet combina las acciones de “chequea el estado” y “soluciona cualquier problema” en una sola declaración para cada recurso, y como cada recurso está representado por una sola declaración atómica, el orden importa mucho menos que en un script equivalente.

Mejor dicho, importa menos siempre y cuando los recursos sean independientes y no estén relacionados entre ellos; la mayoría de ellos son independientes, pero algunos dependen de otros. Consideremos un servicio instalado por un paquete: es imposible llevar el servicio a su estado deseado si el paquete no está instalado. El servicio *depende* del paquete.

Puppet tiene formas de expresar esas relaciones cuando trata con recursos relacionados.

### Sumario de esta página

+ Puedes insertar información de la relación en un recurso con los metaparámetros **before**, **require**, **notify**, y **subscribe**.
+ También puedes declarar relaciones fuera de un recurso con las flechas de encadenamiento **->** y  **~>**
+ Las relaciones se pueden ordenar (esto antes que aquello) u ordenar con notificaciones (esto antes que aquello, y avisa si aquello fue cambiado)
+ La sintaxis y el comportamiento de las relaciones de Puppet está documentada en [la página de relaciones del manual de referencia de Puppet]().

## Metaparámetros, referencias de recurso y orden

Aquí tenemos un recurso de notify que depende de un recurso de archivo.

	    file {'/tmp/test1':
	      ensure  => present,
	      content => "Hi.",
	    }

	    notify {'/tmp/test1 has already been synced.':
	      require => File['/tmp/test1'],
	    }

Cada tipo de recurso tiene su propio conjunto de atributos, pero existe otro conjunto de atributos llamados [metaparámetros](), los cuales pueden usarse en cualquier recurso. Se llaman “meta” porque no describen ninguna característica del recurso que puedas observar en el sistema luego que Puppet termine, ellos sólo describen cómo debe actuar Puppet. Hay cuatro metaparámetros que te permiten ordenar recursos:

+ **before**
+ **require**
+ **notify**
+ **subscribe**

Todos aceptan una [referencia de recurso]() ( o un [array]() de ellas) como su valor. Las referencias de recurso se ven así:

	   Type['title']

Ten en cuenta los corchetes y mayúscula en el tipo de recurso!

#### Nota aparte: Cuándo usar mayúsculas

La forma más simple, es recordar que *sólo* usamos nombres en minúscula cuando *declaramos un nuevo recurso*. El resto de las situaciones necesitan siempre el nombre en mayúscula.

### Before y Require

**before** y **require** crean relaciones de dependencia simples, donde un recurso debe sincronizarse antes que otro. **before** es usado en el recurso anterior, y lista recursos que dependen de él. **require** se usa en el recurso posterior y lista los recursos de los que él depende.

Estos dos metaparámetros son formas diferentes de escribir la misma relación. El ejemplo anterior se puede escribir así de simple:


	    file {'/tmp/test1':
	      ensure  => present,
	      content => "Hi.",
	      before  => Notify['/tmp/test1 has already been synced.'],
	    }

	    notify {'/tmp/test1 has already been synced.':}


### Notify y Subscribe
Algunos tipos de recurso (**service, exec,** y **mount**) pueden ser **”refrescados”** lo que significa que puede reaccionar a los cambios en su entorno. Para un servicio, generalmente será reiniciarse cuando un archivo de configuración sea cambiado; para un recurso **exec**, será ejecutar su comando si alguna cuenta de usuario cambia. Ten en cuenta que los “refresh” son realizados por Puppet, por lo tanto sólo pueden ocurrir durante una ejecución del agente de Puppet.

Los metaparámetros **notify** y **subscribe** generan relaciones de dependencia de la misma forma que lo hacen **before** y **require**, pero también generan **relaciones de notificación**.  No sólo se sincronizará primero el primer recurso del par, sino que si Puppet realiza algún cambio a ese recurso, enviará un evento de refresh al último recurso, el cual reaccionará de manera acorde.

Un ejemplo de una relación de notificación:

	    file { '/etc/ssh/sshd_config':
	      ensure => file,
	      mode   => 600,
	      source => 'puppet:///modules/ssh/sshd_config',
	    }
	    service { 'sshd':
	      ensure    => running,
	      enable    => true,
	      subscribe => File['/etc/ssh/sshd_config'],
	    }

En este ejemplo, el servicio **sshd** será reiniciado si Puppet tiene que editar su archivo de configuración.

## Flechas de encadenamiento

Existe una última forma de declarar relaciones: encadenar referencias de recurso con las [flechas de orden (->) y notificación (~>)](). Piensa en ellas como una representación del paso del tiempo: el recurso detrás de la flecha será sincronizado *antes* que el recurso al que apunta la flecha. Este ejemplo genera la misma dependencia que los ejemplos anteriores:

	    file {'/tmp/test1':
	      ensure  => present,
	      content => "Hi.",
	    }

	    notify {'after':
	      message => '/tmp/test1 has already been synced.',
	    }

	    File['/tmp/test1'] -> Notify['after']

Las flechas de encadenamiento pueden tomar varias cosas como sus operandos: este ejemplo utiliza referencias de recurso, pero también podría tomar declaraciones de recursos y [colectores de recursos]().

Como el espacio en blanco puede utilizarse en cualquier lugar en Puppet, y como las flechas de encadenamiento pueden ir entre declaraciones de recursos, es fácil generar  una pequeña cadena de recursos sincronizados en el orden escrito. Sólo tienes que colocar flechas de encadenamiento entre ellas:

	    file {'/tmp/test1':
	      ensure  => present,
	      content => "Hi.",
	    }
	    ->
	    notify {'after':
	      message => '/tmp/test1 has already been synced.',
	    }

Nuevamente, eso genera la misma relación que hemos visto antes.

## Autorequire

Algunos de los tipos de recuso avisarán cuando una instancia esté relacionada a otros recursos, y estos establecerán dependencias automáticas. El que utilizarás más seguido está entre los archivos y sus directorios madre: Si un archivo determinado y su directorio madre son administrados como recursos, Puppet se asegurará de sincronizar el directorio antes que el archivo. **Esto nunca crea recursos nuevos**; sólo agrega dependencias a los recursos que ya están siendo administrados.

No te preocupes mucho por los detalles del autorequire, es bastante conservador y generalmente debe hacer bien las cosas sin causar problemas. Si te olvidas que esto existe y generas dependencias explícitas, el código funcionará de todos modos. Las dependencias explícitas también sustituirán autorequires si éstos entran en conflicto.

## Ejemplo: sshd
Con suerte todo esto quedó claro! Pero incluso si es así, esto es más bien abstracto. Asegurarse de que una notificación se dispara luego de un archivo es un caso de uso bastante “hola mundo”, y no muy ilustrativo. Ahora rompamos algo!

Probablemente has estado utilizando SSH y tu aplicación de terminal favorita para interactuar con la VM de Aprende Puppet, así que vayamos directo al caso más molesto: Haremos de cuenta que alguien accidentalmente le dio a la persona equivocada (por ejemplo, nosotros) privilegios de sudo y le arruinaron la capacidad de login como root por SSH a esta máquina.

### Prepara
Hagamos una copia del archivo de configuración actual de sshd. Más adelante utilizaremos la copia nueva como la fuente canónica para ese archivo.

	# cp /etc/ssh/sshd_config ~/examples/

Ahora escribiremos código Puppet para manejar el archivo:

	    # /root/examples/break_ssh.pp (incomplete)
	    file { '/etc/ssh/sshd_config':
	      ensure => file,
	      mode   => 600,
	      source => '/root/examples/sshd_config',
	      # Y si, ésta es la primera vez que usamos el atributo “source”. Éste toma
	      # paths locales absolutos y URLs puppet:///, sobre las cuales hablaremos
	      # más adelante.
	    }

Pero esto es sólo la mitad de lo que necesitamos. Cambiará el archivo de configuración, pero esos cambios tendrán efecto sólo cuando el servicio se reinicie, lo que puede suceder dentro de mucho tiempo.

Para hacer que el servicio se reinicie siempre que hagamos cambios a la configuración, debemos decirle a Puppet que administre el servicio **sshd** y hacer que se suscriba al archivo de configuración:

	    # /root/examples/break_ssh.pp
	    file { '/etc/ssh/sshd_config':
	      ensure => file,
	      mode   => 600,
	      source => '/root/examples/sshd_config',
	    }
	    service { 'sshd':
	      ensure     => running,
	      enable     => true,
	      subscribe  => File['/etc/ssh/sshd_config'],
	    }


### Administra
Genial, tenemos nuestro código Puppet, ahora peguémoslo en **/etc/puppetlabs/puppet/manifests/site.pp** así el agente de Puppet manejará esos recursos.

### Rompe
Luego, edita el archivo original **/etc/ssh/sshd_config**. Hay una línea comentada allí, que dice **#PermitRootLogin yes**; encuéntrala, quita el comentario y cambia el *yes* por un *no*:

	PermitRootLogin no

Ahora reinicia manualmente el servicio sshd:

	# service sshd restart

..y desloguéate. No deberías poder volver a loguearte como *root* por SSH, aunque sí puedas loguearte a través de tu consola de virtualización de software. Pruébalo para asegurarte.

### Repara
Ahora que has agregado esos recursos al site.pp, Puppet reparará esto automáticamente en aproximadamente media hora; pero si eres impaciente, puedes [loguearte en la consola de Puppet Enterprise](), luego [activar una ejecución del Puppet agent]() en la página de Live Management. Y lo hará! Luego de que la ejecución del agente se haya completado y puedas ver que el reporte aparece en la consola (tendrá un ícono azul con el que moestrará los cambios hechos), podrás loguearte como *root* a través de SSH nuevamente. *Victoria*.

#### Sin cambios? Sin *refresh*
Existe una extraña situación en la que puedes terminar si aplicas un manifiesto que realice cambios de archivos de configuración antes que lo termines de escribir.

Puppet sólo envía eventos de refresh si realiza cambios al recurso que realiza la notificación en *esta ejecución*. Con lo cual, si has escrito un recurso de archivo con un nuevo contenido deseado para un archivo de configuración, el servicio no sería refrescado, ya que el recurso de archivo ya está en su estado deseado.

Esto te sucederá en general sólo en máquinas en las que estés probando versiones “borrador” de manifiestos, en lugar de las máquinas de producción. Si te sucede, puedes reiniciar el servicio manualmente con [la sección “Tareas avanzadas” de la página de Live Management de la consola PE](). Utiliza la acción “restart” en la sección “service tasks”.

## Paquete/Archivo/Servicio
El ejemplo que vimos estuvo muy cerca de ser un patrón de lo que verás constantemente en código Puppet de producción, pero le faltó una pieza; completémoslo:

	    # /root/examples/break_ssh.pp
	    package { 'openssh-server':
	      ensure => present,
	      before => File['/etc/ssh/sshd_config'],
	    }
	    file { '/etc/ssh/sshd_config':
	      ensure => file,
	      mode   => 600,
	      source => '/root/examples/sshd_config',
	    }
	    service { 'sshd':
	      ensure     => running,
	      enable     => true,
	      subscribe  => File['/etc/ssh/sshd_config'],
	    }

Este es el patrón **paquete/archivo/servicio**, una de las estructuras más útiles en Puppet: el recurso *package* se asegura de que el software y su configuración estén instalados, el archivo de configuración depende del recurso del paquete y el servicio se subscribe a los cambios en el archivo de configuración.

Es difícil de exagerar la importancia de este patrón! Si pararas aquí y sólo aprendieras esto, de todos modos tendrías una buena parte del trabajo hecho.

### Ejercicio: Apache
Escribe y aplica un manifiesto que instalará el [paquete]() Apache, luego asegúrate que el [servicio]() Apache se esté ejecutando. Prueba que funciona utilizando un navegador web en tu servidor para visualizar la página de inicio de Apache.

**Trabajo extra**:  Administra el archivo httpd.conf, y notifica al servicio. Obliga a Apache a permanecer en determinada versión. Ten en cuenta que tendrás que investigar el formato de los strings de versiones para tu sistema operativo, y también qué versiones están disponibles.

Pistas:

+ En sistemas Linux del tipo Red Hat (tu VM) y Debian, los paquetes son instalados desde los repositorios Apt o YUM del sistema operativo. Como las herramientas del sistema saben cómo encontrar e instalar un paquete (e incluso una versión específica de un paquete), todo lo que necesita saber Puppet es **ensure=>installed**, no necesita saber dónde está alojado el paquete.
+ Los nombres de los paquetes de los recursos de servicio dependen de las convenciones de nombres propias de OS. Esto significa que a menudo necesitas investigar antes de escribir un manifiesto para aprender, por ejemplo, cuál es el nombre local del paquete y del servicio Apache.  En CentOS, donde se ejecuta tu VM, el paquete y el servicio se llaman **httpd**.
+ Asegúrate de estar utilizando los valores **ensure** correctos para cada tipo de recurso; no son iguales para un paquete, archivo o servicio.

## Siguiente Paso
**Próxima clase**
Ahora que puedes expresar dependencias  entre recursos, es tiempo de hacer tus manifiestos más consistentes respecto del mundo real con [variables, facts y condicionales]().

**Nota aparte**
Ahora que puedes manejar un servicio completo de punta a punta, intenta manejar un servicio importante en tu propio sistema de prueba. [Descarga gratis Puppet Enterprise](), sigue [la guía de comienzo rápido]() para obtener un pequeño entorno instalado y luego intenta construir un patrón package/file/service al final del archivo de Pupper Master **/etc/puppetlabs/puppet/manifests/site.pp**. MySQL? Memcached?
Tú decides.
