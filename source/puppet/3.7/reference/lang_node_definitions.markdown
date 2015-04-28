---
layout: default
title: "Language: Node Definitions"
canonical: "/puppet/latest/reference/lang_node_definitions.html"
---

[hiera]: /hiera/latest
[sitepp]: ./dirs_manifest.html
[certname]: ./config_important_settings.html#basics
[classes]: ./lang_classes.html
[nodescope]: ./lang_scope.html#node-scope
[topscope]: ./lang_scope.html#top-scope
[extlookup]: /references/3.7.latest/function.html#extlookup
[custom_functions]: /guides/custom_functions.html
[import]: ./lang_import.html
[regex]: ./lang_datatypes.html#regular-expressions
[strings]: ./lang_datatypes.html#strings
[inherit]: ./lang_classes.html#inheritance
[modules]: ./modules_fundamentals.html
[enc]: /guides/external_nodes.html
[facts]: ./lang_variables.html#facts-and-built-in-variables
[catalog]: ./lang_summary.html#compilation-and-catalogs
[strict]: /references/3.7.latest/configuration.html#stricthostnamechecking
[conditional]: ./lang_conditional.html


A **node definition** or **node statement** is a block of Puppet code that will only be included in one node's [catalog][]. This feature allows you to assign specific configurations to specific nodes.

Node statements are an **optional feature** of Puppet. They can be replaced by or combined with an [external node classifier][enc], or you can eschew both and use conditional statements with [facts][] to classify nodes.

Unlike more general conditional structures, node statements only match nodes by **name.** By default, the name of a node is its [certname][] (which defaults to the node's fully qualified domain name).

Location
-----

Node definitions should go in [the main manifest (site.pp)][sitepp]. The main manifest can be a single file, or a directory containing many files.

Syntax
-----

{% highlight ruby %}
    # /etc/puppetlabs/puppet/manifests/site.pp
    node 'www1.example.com' {
      include common
      include apache
      include squid
    }
    node 'db1.example.com' {
      include common
      include mysql
    }
{% endhighlight %}

In the example above, only `www1.example.com` would receive the apache and squid classes, and only `db1.example.com` would receive the mysql class.

Node definitions look like class definitions. The general form of a node definition is:

* The `node` keyword
* The name(s) of the node(s)
* Optionally, the `inherits` keyword followed by the name of another node definition
* An opening curly brace
* Any mixture of class declarations, variables, resource declarations, collectors, conditional statements, chaining relationships, and functions
* A closing curly brace

> #### Aside: Best Practices
>
> Although node statements can contain almost any Puppet code, we recommend that you **only** use them to **set variables** and **declare classes.** Avoid using resource declarations, collectors, conditional statements, chaining relationships, and functions in them; all of these belong in classes or defined types.
>
> This will make it easier to switch between node definitions and an ENC.



Naming
-----

Node statements match nodes by name. A node's name is its unique identifier; by default, this is its [certname][] setting, which in turn resolves to the node's fully qualified domain name.

> #### Notes on Node Names
>
> * The set of characters allowed in a node name is **undefined** in this version of Puppet. For best future compatibility, you should limit node names to letters, numbers, periods, underscores, and dashes.
> * Although it is possible to configure Puppet to use something other than the [certname][] as a node name, this is not generally recommended.

A node statement's **name** must be one of the following:

* A quoted [string][strings]
* The bare word `default`
* A [regular expression][regex]

You may not create two node statements with the same name.


### Multiple Names

You can use a comma-separated list of names to create a group of nodes with a single node statement:

{% highlight ruby %}
    node 'www1.example.com', 'www2.example.com', 'www3.example.com' {
      include common
      include apache, squid
    }
{% endhighlight %}

This example creates three identical nodes: `www1.example.com`, `www2.example.com`, and `www3.example.com`.

### The Default Node

The name `default` (without quotes) is a special value for node names. If no node statement matching a given node can be found, the `default` node will be used. See [Behavior](#behavior) below.

### Regular Expression Names

[Regular expressions (regexes)][regex] can be used as node names. This is another method for writing a single node statement that matches multiple nodes.

> **Note:** Make sure all of your node regexes match non-overlapping sets of node names. If a node's name matches more than one regex, Puppet makes no guarantee about which matching definition it will get.

{% highlight ruby %}
    node /^www\d+$/ {
      include common
    }
{% endhighlight %}

The above example would match `www1`, `www13`, and any other node whose name consisted of `www` and one or more
digits.

{% highlight ruby %}
    node /^(foo|bar)\.example\.com$/ {
      include common
    }
{% endhighlight %}

The above example would match `foo.example.com` and `bar.example.com`, but no other nodes.


Behavior
-----

If site.pp contains at least one node definition, it must have one for **every** node; compilation for a node will fail if one cannot be found. (Hence the usefulness of [the `default` node](#the-default-node).) If site.pp contains **no** node definitions, this requirement is dropped.

### Matching

A given node will only get the contents of **one** node definition, even if two node statements could match a node's name. Puppet will do the following checks in order when deciding which definition to use:

1. If there is a node definition with the node's exact name, Puppet will use it.
2. If there is a regular expression node statement that matches the node's name, Puppet will use it. (If more than one regex node matches, Puppet will use one of them, with no guarantee as to which.)
3. If the node's name looks like a fully qualified domain name (i.e. multiple period-separated groups of letters, numbers, underscores and dashes), Puppet will chop off the final group and start again at step 1. (That is, if a definition for `www01.example.com` isn't found, Puppet will look for a definition matching `www01.example`.)
4. Puppet will use the `default` node.

Thus, for the node `www01.example.com`, Puppet would try the following, in order:

* `www01.example.com`
* A regex that matches `www01.example.com`
* `www01.example`
* A regex that matches `www01.example`
* `www01`
* A regex that matches `www01`
* `default`

You can turn off this fuzzy name matching by changing the Puppet master's [`strict_hostname_checking`][strict] setting to `true`. This will cause Puppet to skip step 3 and only use the node's full name before resorting to `default`.

### Regex Capture Variables

Regex node definitions will set numbered regex capture variables ($1, $2, etc.) within the body of the node definition. This is similar to the behavior of [conditional statements][conditional] that use regexes.

### Code Outside Node Statements

Puppet code that is outside any node statement will be compiled for every node. That is, a given node will get both the code in its node definition and the code outside any node definition.

### Node Scope

Node definitions create a new anonymous scope that can override variables and defaults from top scope. See [the section on node scope][nodescope] for details.

### Merging With ENC Data

Node definitions and [external node classifiers][enc] can co-exist. Puppet merges their data as follows:

* Variables from an ENC are set at [top scope][topscope] and can thus be overridden by variables in a node definition.
* Classes from an ENC are declared at [node scope][nodescope], which means they will be affected by any variables set in the node definition.

Although ENCs and node definitions can work together, we recommend that most users pick one or the other.

### Inheritance

In earlier versions of Puppet, nodes could inherit from other nodes using the `inherits` keyword. **This feature is deprecated in Puppet 3.7,** and will be removed in Puppet 4. Node inheritance often caused complications and ambiguities --- classes and defined types are more effective strategies for reuse. As of Puppet 3.7, node inheritance causes a deprecation warning in the current parser and an error in the future parser.


> #### Alternatives to Node Inheritance
>
> * Most users who need hierarchical data should keep it in an external source and have their manifests look it up. The best solution right now is [Hiera][], which is available by default in Puppet 3 and later. See our [Hiera guides][hiera] for more information about using it.
> * [ENCs][enc] can look up data from any arbitrary source and return it to Puppet as top-scope variables.
> * If you have node-specific data in an external CMDB, you can easily write [custom Puppet functions][custom_functions] to query it.
> * For very small numbers of nodes, you can copy and paste to make complete node definitions for special-case nodes.
