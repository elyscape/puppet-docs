---
layout: default
title: "El archivo de configuración hiera.yaml"
canonical: "/es/hiera/configuring.html"
---

*Archivo de configuración Hiera* generalmente refiere a **hiera.yaml**. Usa este archivo para configurar la [jerarquía](http://docs.puppetlabs.com/es/hiera/hierarchy.html), qué backend(s) usar, y la configuración para cada backend.

Hiera fallará si no puede encontrar el archivo de configuración aunque está permitido un archivo de configuración vacío.

## Ubicación
Hiera usa diferentes ubicaciones para archivos de configuración, dependiendo de cómo sea ejecutado.

### Desde Puppet
Por defecto, el archivo de configuración es **$confdir/hiera.yaml**, que generalmente es uno de estos:

+ **/etc/puppet/hiera.yaml** en Puppet *open source* \*nix
+ **/etc/puppetlabs/puppet/hiera.yaml** en Puppet Enterprise \*nix
+ **[COMMON_APPDATA]( http://docs.puppetlabs.com/windows/installing.html#the-commonappdata-folder )\PuppetLabs\puppet\etc\hiera.yaml** en Windows

En Puppet 3 o superior, puedes especificar un archivo de configuración diferente con [la opción de configuración **hiera_config**](http://docs.puppetlabs.com/references/latest/configuration.html#hieraconfig) en **puppet.conf**. En Puppet 2.x, no puedes especificar un archivo de configuración diferente, pero sí puedes crear un link simbólico de **$confdir/hiera.yaml** a un archivo diferente.

### Desde la Línea de comando

+ **/etc/hiera.yaml** en \*nix
+ [**COMMON_APPDATA**](http://docs.puppetlabs.com/windows/installing.html#the-commonappdata-folder)**\PuppetLabs\hiera\etc\hiera.yaml** en Windows

Puedes especificar un archivo de configuración diferente con la opción **-c (--config)**.

### Desde Ruby

+ **/etc/hiera.yaml** en \*nix
+ **[COMMON_APPDATA](http://docs.puppetlabs.com/windows/installing.html#the-commonappdata-folder)\PuppetLabs\hiera\etc\hiera.yaml** en Windows

Puedes especificar un archivo de configuración diferente, o un *hash* de configuraciones cuando llamas al método **Hiera.new**.

## Formato
La configuración de Hiera debe ser un hash [YAML](http://www.yaml.org/YAML_for_ruby.html). El archivo debe ser YAML válido, pero puede no contener información.
Cada clave de alto nivel en el hash **debe ser un símbolo Ruby con un prefijo de dos puntos (:)**. Las configuraciones disponibles están en una lista debajo, en [“Configuraciones Globales”](http://docs.puppetlabs.com/es/hiera/configuring.html#global-settings) y [“Configuraciones específicas para backends”](http://docs.puppetlabs.com/es/hiera/configuring.html#backend-specific-settings).

### Ejemplo de archivo de configuración

	---
	:backends:
 		 -	 yaml
 		 -	 json
	:yaml:
		:datadir: 		/etc/puppet/hieradata
	:json:
	 	 :datadir:		/etc/puppet/hieradata
	:hierarchy:
  		-	 %{::clientcert}
 	 	-	 %{::custom_location}
		-	 common

### Valores de configuración por defecto
Si el archivo de configuración existe pero no contiene información, la configuración por defecto será equivalente a la siguiente:

	---
	:backends: yaml
	:yaml:
	  :datadir: /var/lib/hiera
	:hierarchy: common
	:logger: console

## Configuraciones globales
El archivo de configuración Hiera puede contener cualquiera de las siguientes configuraciones. En su ausencia, tendrá valores por defecto. **Ten en cuenta que cada opción de configuración debe ser un símbolo Ruby con un prefijo de dos puntos (:)**

### :hierarchy
Debe ser un **string** o un **array de strings** donde cada string es el nombre para una fuente de información estática o dinámica. Una fuente dinámica es simplemente una que contiene un símbolo de interpolación **%{variable}** (Mira [“Crear jerarquías”](http://docs.puppetlabs.com/es/hiera/hierarchy.html) para más detalles). Las fuentes de información en la jerarquía son chequeadas en orden, de arriba hacia abajo.

**Valor por defecto: “common”** (es decir, una jerarquía de un sólo elemento cuyo único nivel se llama *"common"*)

### :backends
Debe ser un **string** o un **array de strings** donde cada string es el nombre para un backend disponible de Hiera. Los backends integrados son **yaml** y **json**; hay un backend adicional de **puppet** disponible cuando usas Hiera con Puppet. Los backends adicionales están disponibles como complemento.

**Backends personalizados**: Mira [“Escribir backends personalizados”](http://docs.puppetlabs.com/es/hiera/custom_backends.html) para más detalles sobre cómo escribir un nuevo backend. Los backends personalizados pueden interactuar con casi cualquier data store existente.
La lista de backends es procesada en orden: en el [ejemplo debajo](http://docs.puppetlabs.com/es/hiera/configuring.html#example-config-file), Hiera chequearía la jerarquía completa en el backend **yaml** antes de comenzar nuevamente con el backend **json**.

**Valor por defecto: “yaml”**

### :logger
Debe ser un **string** con el nombre de un logger disponible.

Los loggers sólo controlan hacia dónde se direccionan las alertas y los mensajes de depuración. Puedes usar uno de los loggers inegrados o escribir el tuyo propio. Los loggers integrados son:

**console** (los mensajes van directo a *STDERR*)

**puppet** (los mensajes van al sistema de logueo de Puppet)

**noop** (los mensajes son silenciados)

**Loggers personalizados**: Puedes hacer tus propios loggers proporcionando una clase llamada, por ejemplo, **Hiera::Foo_logger** (en tal caso el nombre interno en Hiera para el logger sería **foo**), y dándole métodos de clase llamados **warn** y **debug**, los cuales deben aceptar cada uno un string simple.

**Valor por defecto**: **”console”**;  Ten en cuenta que Puppet lo sustituye por **”puppet”**, independientemente de lo que haya en el archivo de configuración.

### :merge_behavior
Debe ser uno de los siguientes:

+ **native**(por defecto) – combina sólo las claves de nivel superior
+ **deep** - merge recursivo; en caso de conflicto con las claves, permite que prevalezcan valores **de menor prioridad**. Casi nunca querrás esto.
+ **deeper** - merge recursivo; en caso de conflicto, permite que prevalezcan valores **de mayor prioridad**.

Cualquiera de estas opciones a excepción de **native** requiere que el Gem **deep_merge** esté instalado.

¿Qué estrategia de combinación usar cuándo estamos haciendo una [búsqueda combinada de hash?](http://docs.puppetlabs.com/es/hiera/lookup_types.html#hash-merge). Mira [“Deep merging” en Hiera≥1.2.0”](http://docs.puppetlabs.com/es/hiera/lookup_types.html#deep-merging-in-hiera--120) para más detalles.

## Opciones de configuración específicas para Backends
Todos los backends pueden definir sus propias opciones de configuración y leerlas desde el archivo de configuración de Hiera. Si existe, el valor de una clave de un backend determinado debe ser un **hash**, cuyas claves son los valores de configuración que usa.

Las siguientes opciones de configuración están disponibles para backends integrados:

### :yaml y :json
**datadir**
El directorio en el cual encontrarás los archivos de fuentes de información.

Puedes [interpolar variables](http://docs.puppetlabs.com/es/hiera/variables.html) en el *datadir* usando símbolos  de interpolación del tipo **%{variable}**. Esto te permite, por ejemplo, apuntarlo en **/etc/puppet/hieradata/%{::environment}** para mantener la información de desarrollo y producción completamente separadas.

**Valor por defecto**: **/var/lib/hiera ** en \*nix, y **[COMMON_APPDATA](http://docs.puppetlabs.com/windows/installing.html#the-commonappdata-folder)\PuppetLabs\Hiera\var** ) en Windows.

### :puppet

**:datasource**

La clase Puppet en la que debes buscar información.

**Valor por defecto: data**


