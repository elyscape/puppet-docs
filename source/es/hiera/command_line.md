---
layout: default
title: "Hiera 1: Uso en la línea de comandos"
canonical: "/es/hiera/command_line.html"
---

Hiera proporciona una herramienta de línea de comandos útil para verificar que la jerarquía esté correctamente construida y que las fuentes de información estén devolviendo los valores esperados. Seguramente ejecutarás la herramienta de línea de comandos de Hiera en un puppet master, haciendo una maqueta de los facts que los agentes normalmente proveen al puppet master usando variadas [fuentes de facts](http://docs.puppetlabs.com/es/hiera/command_line.html#fact-sources).

## Invocación
El comando más simple de Hiera toma un sólo argumento (la clave de búsqueda) y busca el valor de la clave usando las [fuentes de información](http://docs.puppetlabs.com/es/hiera/data_sources.html) estáticas en la [jerarquía](http://docs.puppetlabs.com/es/hiera/hierarchy.html).

	**$ hiera ntp_server**

Una invocación estándar proveerá un conjunto de variables para que use Hiera, con lo cual podrá también usar sus [fuentes de información](http://docs.puppetlabs.com/es/hiera/data_sources.html) dinámicas:

	**$ hiera ntp_server --yaml web01.example.com.yaml**

## Ubicación del archivo de configuración

La herramienta de línea de comandos de Hiera busca su configuración en **/etc/hiera.yaml**. Puedes usar el argumento **--config** para especificar un archivo de configuración diferente. Mira la documentación de [archivo de configuración](http://docs.puppetlabs.com/es/hiera/configuring.html#location) de Hiera para saber dónde encontrar este archivo para la versión de Puppet que tengas y tu sistema operativo. Considera reconfigurar Puppet para usar **/etc/hiera.yaml** (Puppet 3) o establece un link simbólico a **/etc/hiera.yaml** (Puppet 2.7).

### Orden de los argumentos
Hiera es sensible a la posición de sus argumentos en la línea de comandos:

+ El primer valor es siempre la clave de búsqueda
+ El primer argumento luego de la clave que no incluya un signo igual (=) se convierte en el valor por defecto, el cual Hiera devolverá si no encuentra la clave. Sin un valor por defecto y en ausencia de una clave correspondiente en la jerarquía, Hiera devuelve el valor **nil** (nulo)
+ Recuerda que los argumentos deben ser pares **variable=valor**.
+ Las **opciones** pueden estar en cualquier lugar.

### Opciones
Hiera acepta las siguientes opciones de línea de comando:

<table>
 <thead>
  <tr>
	<th>Argumento</th>
	<th>Uso</th>
  </tr>
 </thead>
 <tbody>
	<tr>
	 <td><code>-V, --version</code></td>
	 <td>Información de la versión</td>
	</tr>
	<tr>
	 <td><code>-c, --config FILE</code></td>
	 <td>Especifica la ubicación del archivo de configuración alternativo</td>
	</tr>
	<tr>
	 <td><code>-d, --debug</code></td>
	 <td>Muestra información de depuración</td>
	</tr>
	<tr>
	 <td><code>-a, --array</code></td>
	 <td>Devuelve todos los valores como un array aplanado de valores únicos</td>
	</tr>
	<tr>
	 <td><code>-h, --hash</code></td>
	 <td>Devuelve todos los valores de hashes como un hash combinado</td>
	</tr>
	<tr>
	 <td><code>-j, --json FILE</code></td>
	 <td>Archivo JSON del cual cargar el scope</td>
	</tr>
	<tr>
	 <td><code>-y, --yaml FILE</code></td>
	 <td>Archivo YAML del cual cargar el scope</td>
	</tr>
	<tr>
	 <td><code>-m, --mcollective IDENTITY</code></td>
	 <td>Utilizar facts de un nodo (vía mcollective) como scope</td>
	</tr>
	<tr>
	 <td><code>-i, --inventory_service IDENTITY</code></td>
	 <td>Utilizar facts de un nodo (vía el servicio de inventario de Puppet) como scope</td>
	</tr>
 </tbody>
</table>


## Fuentes de facts
Cuando lo utilizas desde Puppet, Hiera recibe automáticamente todos los facts que necesita. En la línea de comandos necesitarás pasar manualmente esos facts.

Seguramente ejecutarás la herramienta de línea de comandos de Hiera en tu nodo puppet master, donde se espera que los facts correspondan a alguna de las siguientes posibilidades:

+ Incluídos en la línea de comandos como variables (ej: **operatingsystem=Debian**)
+ Determinados como un [archivo YAML o JSON](http://docs.puppetlabs.com/es/hiera/command_line.html#json-and-yaml-scopes)
+ Recuperados sobre la marcha con información de [MCollective](http://docs.puppetlabs.com/es/hiera/command_line.html#mcollective)
+ Buscados en el [servicio de inventario de Puppet](http://docs.puppetlabs.com/es/hiera/command_line.html#inventory-service)
Las descripciones de estas opciones puedes encontrarlas a continuación.

### Variables de línea de comandos

Hiera acepta facts en la línea de comandos en forma de pares **variable=valor**, por ejemplo,  **hiera ntp_server osfamily=Debian clientcert="web01.example.com"**. Los valores de las variables debe ser strings y deben estar entre comillas en caso de tener espacios.

Esto es útil si los valores que estás probando sólo dependen de unos pocos facts. Puede volverse difícil de manejar si tu jeraquía es grande o necesitas evaluar los valores para muchos nodos de una vez. En esos casos, debes usar una de las siguientes posibilidades.

### Scopes JSON y YAML
En lugar de pasar la lista de variables a Hiera en forma de argumentos de línea de comandos, puedes usar archivos JSON o YAML; los puedes construir tú mismo o usar un archivo YAML recuperado del caché de Puppet o generado con **facter –yaml**.

Dado este comando, utilizando asignación de variables de línea de comandos:

	**$ hiera ntp_server osfamily=Debian timezone=CST**

Los siguientes ejemplos en YAML y JSON devuelven los mismos resultados:

#### Ejemplo de Scope YAML
	**$ hiera ntp_server -y facts.yaml**

	# facts.yaml
	---
	osfamily: Debian
	timezone: CST

#### Ejemplo de Scope JSON

	**$ hiera ntp_server -j facts.json**

	# facts.json
	{
  	"osfamily" : "Debian",
  	"timezone" : "CST"
	}

### MCollective

Si estás usando Hiera en una máquina en la que está permitida la emisión de comandos MCollective, puedes pedirle a cualquier nodo que ejecute MCollective para enviarte sus facts. Hiera usará esos facts para conducir la búsqueda.

Próximamente tendremos un ejemplo de esto.

### Servicio de inventario

Si utilizas el [servicio de inventario](http://docs.puppetlabs.com/guides/inventory_service.html) de Puppet, puedes consultar el puppet master por facts de cualquier nodo. Hiera usará esos facts para conducir la búsqueda.

Próximamente tendremos un ejemplo de esto.

## Tipos de búsqueda

Por defecto, la herramienta de línea de comandos de Hiera usará una [búsqueda de prioridad](http://docs.puppetlabs.com/es/hiera/lookup_types.html#priority-default) la cual devolverá un valor simple, el primero que encuentre en la jerarquía. Existen otros dos tipos de búsqueda disponibles: Merge en array y Merge en Hash.

### Merge de array

Una búsqueda de merge de array arma un valor combinando cada valor que encuentra en la jerarquía en un array aplanado de valores únicos. [Mira “Búsqueda de merge en array”](http://docs.puppetlabs.com/es/hiera/lookup_types.html#array-merge) para más detalles.

Usa la opción **--array** para hacer una búsqueda de merge de array.

Si alguno de los valores encontrados en las fuentes de información es un hash, la opción  **--array** hará que Hiera devuelva un error.

### Hash

Una búsqueda de merge de hash arma un valor combinando las claves de nivel superior de cada hash que encuentre en la jerarquía en un hash simple. [Mira “Búsqueda de merge de hash”](http://docs.puppetlabs.com/es/hiera/lookup_types.html#hash-merge) para más detalles.

Usa la opción **--hash** para hacer una búsqueda de merge de hash.

Si alguno de los valores encontrados en las fuentes de información es un string o un array, la opción  **--hash** hará que Hiera devuelva un error.
