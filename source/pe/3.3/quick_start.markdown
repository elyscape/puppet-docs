---
layout: default
title: "PE 3.3 » Quick Start » Using PE"
subtitle: "Quick Start: Using PE 3.3"
canonical: "/pe/latest/quick_start.html"
---

[downloads]: http://info.puppetlabs.com/download-pe.html
[agent-install]: ./install_basic.html#installing-agents

Welcome to the Puppet Enterprise 3.3 quick start guide. This document is a short walkthrough to help you evaluate Puppet Enterprise (PE) and become familiar with its features. There are two parts to this guide, an introductory guide (below) that demonstrates basic use and concepts and a follow-up guide where you can build on the concepts you learned in the introduction while learning some basics about developing puppet modules for either [Windows](./quick_writing_windows.html) or [*nix](./quick_writing_nix.html) platforms.

#### Quick Start Part One: Introduction

In this first part, follow along to learn how to:

* Create a small proof-of-concept deployment

>**Note**: The installation instructions describe how to install a single agent. If you want to install more than one agent, just repeat the steps in the "Install the Puppet Enterprise Agent" section.

* Examine and control nodes in real time with live management
* Install a PE-supported Puppet module
* Apply Puppet classes to nodes using the console
* Set the parameters of classes using the console
* Use the console to inspect and analyze the results of configuration events

####  Quick Start Part Two: Developing Modules

For part two, you'll build on your knowledge of PE and  learn about module development . You can choose from either [the Linux track](./quick_writing_nix) or [the Windows track](./quick_writing_windows).

In part two, you'll learn about:

* Basic module structure
* Editing manifests and templates
* Writing your own modules
* Creating a site module that builds other modules into a complete machine role
* Applying classes to groups with the console

> Following this walkthrough will take approximately 30-60 minutes for each part.

Creating a Deployment
-----

A typical Puppet Enterprise deployment consists of:

* A number of **agent nodes,** which are computers (physical or virtual) managed by Puppet.
* At least one **puppet master server,** which serves configurations to agent nodes.
* At least one **console server,** which analyzes agent reports and presents a GUI for managing your site. (This may or may not be the same server as the master.)
* At least one **database support server** which runs PuppetDB and databases that support the console. (This may or may not be the same server as the console server.)

For this walk-through, you will create a simple deployment where the puppet master, the console, and database support components will run on one machine (a.k.a., a monolithic master). This machine will manage one or two agent nodes. In a production environment you have total flexibility in how you deploy and distribute your master, console, and database support components, but for the purposes of this guide we're keeping things simple.

> ### Preparing Your Proof-of-Concept Systems
>
> To create this small deployment, you will need the following:
>
> * At least two computers ("nodes") running a \*nix operating system [supported by Puppet Enterprise](./install_system_requirements.html).
>     * These can be virtual machines or physical servers.
>     * One of these nodes (the puppet master server) should have at least 1 GB of RAM. **Note:** For actual production use, a puppet master node should have at least 4 GB of RAM.
> * For part two, if you choose to follow the Windows track you'll need a computer running a version of Microsoft Windows [supported by Puppet Enterprise](./install_system_requirements.html).
> * [Puppet Enterprise installer tarballs][downloads] suitable for the OS and architecture your nodes are using.
> * A network --- all of your nodes should be able to reach each other.
> * All of the nodes you intend to use should have their system clocks set to within a minute of each other.
> * An internet connection or a local mirror of your operating system's package repositories, for downloading additional software that Puppet Enterprise may require.
> * [Properly configured firewalls](./install_system_requirements.html#firewall-configuration).
>     * For demonstration purposes, all nodes should allow **all traffic on ports 8140, 61613, and 443.** (Production deployments can and should partially restrict this traffic.)
> * [Properly configured name resolution](./install_system_requirements.html#name-resolution).
>     * Each node needs **a unique hostname,** and they should be on **a shared domain.** For the rest of this walkthrough, we will refer to the puppet master as `master.example.com` and the agent node as `agent1.example.com`. You can use any hostnames and any domain; simply substitute the names as needed throughout this document.
>     * All nodes must **know their own hostnames.** This can be done by properly configuring reverse DNS on your local DNS server, or by setting the hostname explicitly. Setting the hostname usually involves the `hostname` command and one or more configuration files, while the exact method varies by platform.
>     * All nodes must be able to **reach each other by name.** This can be done with a local DNS server, or by editing the `/etc/hosts` file on each node to point to the proper IP addresses. Test this by running `ping master.example.com` and `ping agent1.example.com` on every node.
>     * Optionally, to simplify configuration later, all nodes should also be able to **reach the puppet master node at the hostname `puppet`.** This can be done with DNS or with hosts files. Test this by running `ping puppet` on every node.
>     * The **control workstation** from which you are carrying out these instructions must be able to **reach every node in the deployment by name.**
> * [Properly configured SSH](./install_pe_mono.html#ssh-prerequisites-and-notes).
>     * If you have a properly configured SSH agent with agent forwarding enabled, you don't need to perform any additional SSH configurations. Your SSH agent will be used by the installer.
>     * If you have any kind of authentication security system in place (e.g., two factor authentication or RSA key prompting) you will need to temporarily disable that system when installing with the web-based installer.
>     * Are you installing using root with a password? The installer will ask you to provide the username and password for the node on which you're installing PE. Remote root ssh login must be enabled, including on the node from which you're running the installer.
>     * Are you installing using root with an ssh key? The installer will ask you to provide the username, private key path, and key passphrase (as needed) for each node on which you're installing a PE component. Remote root ssh login must enabled on each node, including the node from which you're running the installer. And the public root ssh key must be added to `authorized_keys` on each node on which you're installing a PE component.
> * Please ensure that port 3000 is reachable, as the web-based installer uses this port. You can close the port when the installation is complete.
>
> * The web-based installer does not support sudo configurations with `Defaults targetpw` or `Defaults rootpw`. Make sure your `/etc/sudoers` file does not contain, or else comment out, those lines.
>
> * **For Debian Users**: If you gave the root account a password during the installation of Debian, sudo may not have been installed. In this case, you will need to either install PE as root, or install sudo on any node(s) on which you want to install PE.

### Installing the Puppet Master

1. [Download and verify the appropriate PE tarball](./install_basic.html#downloading-puppet-enterprise).

   > **Tip**: Be sure to download the full PE tarball, not the agent-only tarball. The agent-only tarball is used for [package management-based agent installation](./install_agents.html) which is not covered by this guide.

2. Unpack the tarball. (Run `tar -xf <tarball>`.)
3. From the PE installer directory, run `sudo ./puppet-enterprise-installer`.
4. When prompted, choose "Yes" to install the setup packages. (If you choose "No," the installer will exit.)

   At this point, the PE installer will start a web server and provide a web address: `https://<install platform hostname>:3000`. Please ensure that port 3000 is reachable. If necessary, you can close port 3000 when the installation is complete. Also be sure to use `https`.

   >**Warning**: Leave your terminal connection open until the installation is complete; otherwise the installation will fail.

5. Copy the address into your browser.
6. When prompted, accept the security request in your browser.

   The web-based installation uses a default SSL certificate; you’ll have to add a security exception in order to access the web-based installer. This is safe to do.

   You'll be taken to the installer start page.

7. On the start page, click **Let's get started**.
8. Next, you'll be asked to choose your deployment type. Select **Monolithic**.
9. Provide the following information about the puppet master server:

   a. **Puppet master FQDN**: provide the fully qualified domain name of the server you're installing PE on; for example, `master.example.com`.

   b. **DNS aliases**: provide a comma-separated list of aliases agent nodes can use to reach to the master; for example `master`.

   c. **SSH Username**: provide the SSH username for the user connecting to the puppet master; in this case, `root`.

10. When prompted about database support, choose the default option **Install PostgreSQL for me**.
11. Provide the following information about the PE console administrator user:

    a. **Console superuser email address**: provide the address you'll use to log in to the console as the administrator.

    b. **Console superuser passphrase**: create a password for the console login; as indicated, the password must be at least eight characters.

12. For **SMTP Hostname** use `localhost`.

13. Click **Submit**.
14. On the confirm plan page, review the information you provided, and, if it looks correct, click **Continue**.

    If you need to make any changes, click **Go Back** and make whatever changes are required.

15. On the validation page, the installer will verify various configuration elements (e.g., if SSH credentials are correct, if there is enough disk space, and if the OS is the same for the various components). If there aren't any outstanding issues, click **Deploy now**.

The installer will then install and configure Puppet Enterprise. It may also need to install additional packages from your OS's repository. **This process may take up to 10-15 minutes.** When the installation is complete, the installer script that was running in the terminal will close itself.

> You have now installed the puppet master node. As indicated by the installer, the puppet master node is also an agent node, and can configure itself the same way it configures the other nodes in a deployment. Stay logged in as root for further exercises.

#### Log in to the Console

To log in to the console, you can select the **Start Using Puppet Enterprise Now** button that appears at the end of the web-based installer or follow the steps below.

1. **On your control workstation**, open a web browser and point it to the address supplied by the installer; for example, https://master.example.com.
   You will receive a warning about an untrusted certificate. This is because _you_ were the signing authority for the console's certificate, and your Puppet Enterprise deployment is not known to the major browser vendors as a valid signing authority. **Ignore the warning and accept the certificate**. The steps to do this [vary by browser][console_cert].
2. On the login page for the console, **log in** with the email address and password you provided when installing the puppet master.

   ![The console login screen](./images/quick/login.png)

   The console GUI loads in your browser.

### Installing the Puppet Enterprise Agent

>**Note**: This procedure references RHEL and Debian, but it can be used for all supported platforms except Windows. For instructions on installing agents on Windows, refer to the [Windows agent installation instructions](./install_windows.html).

>**Tip**: If you don't have internet connectivity, refer to [Installing Agents in a Puppet Enterprise Infrastructure without Internet Access](./install_agents.html#installing-agents-in-a-puppet-enterprise-infrastructure-without-internet-access) to choose a method that is suitable for your needs.

The puppet master that you've installed hosts a package repository for the agent of the same OS and architecture as the puppet master. When you run the installation script on your agent (for example, `curl -k https://<master.example.com>:8140/packages/current/install.bash | sudo bash`), the script will detect the OS on which it is running, set up an apt (or yum, or zypper) repo that refers back to the master, pull down and install the `pe-agent` packages.

Note that if install.bash can't find agent packages corresponding to the agent's platform, it will fail with an error message telling you which `pe_repo` class you need to add to the master.

If your agent is the same OS and architecture as the puppet master, run the script above to set up the agent, and then continue on to [Connecting Agents to the Master](#connecting-agents-to-the-master)

If your master OS and architecture is different than the agent (for example, the master is on a node running RHEL6 and you want to add an agent node running Debian 6 on AMD64 hardware) follow this example:

1. **On the console**, click the __Add classes__ button in the sidebar:

   ![The console's add classes button][classbutton]

2. Search for the `pe_repo::platform::debian_6_amd64` class in the list of classes, and click its checkbox to select it. Click the __Add selected classes__ button at the bottom of the page.

3. Navigate to the master.example.com node page, click the __Edit__ button, and begin typing "`pe_repo::platform::debian_6_amd64`" in the __Classes__ field; you can select the `pe_repo::platform::debian_6_amd64` class from the list of autocomplete suggestions.

4. Click the __Update__ button after you have selected it.

5. Note that the `pe_repo::platform::debian_6_amd64` class now appears in the list of classes for the master.example.com node.

6. Navigate to the live management page, and select the __Control Puppet__ tab. Use the __runonce__ action to trigger a puppet run.

   The new repo will be created in `/opt/puppet/packages/public`. It will be called `puppet-enterprise-3.3.0-debian-6-amd64-agent`.

8. SSH into the Debian node where you want to install the agent, and run `curl -k https://<master.example.com>:8140/packages/current/install.bash | sudo bash`.

The installer will then install and configure the Puppet Enterprise agent.

> You have now installed the puppet agent node. Stay logged in as root for further exercises.

Connecting Agents to the Master
-----

After installing, the agent nodes are **not yet allowed** to fetch configurations from the puppet master; they must be explicitly approved and granted a certificate.

### Approving the Certificate Request

[console_cert]: ./console_accessing.html#accepting-the-consoles-certificate

During installation, the agent node contacted the puppet master and requested a certificate. To add the node to the console and to start managing its configuration, you'll need to **approve its request on the puppet master**. This is most easily done via the console.

1. From the console, note the pending __node requests__ indicator in the upper right corner. Click it to load a list of currently pending node requests.

   ![Node Request Indicator](./images/console/request_indicator.png)

2. Click the __Accept All__ button to approve all the requests and add the nodes.

> The puppet agents can now retrieve configurations from the master the next time puppet runs.

### Testing the Agent Nodes

During this walkthrough, we will be running the puppet agent interactively. By default, the agent runs in the background and fetches configurations from the puppet master every 30 minutes. (This interval is configurable with the `runinterval` setting in puppet.conf.) However, you can also trigger a puppet run manually from the command line.

1. **On the agent node,** log in as root and run `puppet agent --test` on the command line. This will trigger a single puppet run on the agent with verbose logging.

   > **Note**: You may receive a `-bash: puppet: command not found` error; this is due to the fact that PE installs its binaries in `/opt/puppet/bin` and `/opt/puppet/sbin`, which aren't included in your default `$PATH`. To include these binaries in your default `$PATH`, manually add them to your profile or run `PATH=/opt/puppet/bin:$PATH;export PATH`.

2. Note the long string of log messages, which should end with `notice: Finished catalog run in [...] seconds`.


> You are now fully managing the agent node. It has checked in with the puppet master for the first time and received its configuration info. It will continue to check in and fetch new configurations every 30 minutes. The node will also appear in the console, where you can make changes to its configuration by assigning classes and modifying the values of class parameters.

### Viewing the Agent Node in the Console


[console_nav]: ./console_navigating.html


1. Click __Nodes__ in the primary navigation bar.
   You'll see various UI elements, which show a summary of recent puppet runs and their status. Notice that the master and any agent nodes appear in the list of nodes:

   ![The console front page](./images/quick/front.png)

2. **Explore the console**. Note that if you click on a node to view its details, you can see its recent history, the Puppet classes it receives, and a very large list of inventory information about it. [See here for more information about navigating the console.][console_nav]

> You now know how to find detailed information about any node PE is managing, including its status, inventory details, and the results of its last puppet run.

### Avoiding the Wait

Although the puppet agent is now fully functional on the agent node, some other Puppet Enterprise software is not; specifically, the daemon that listens for orchestration messages is not yet configured. This is because Puppet Enterprise **uses Puppet to configure itself**.

Puppet Enterprise does this automatically within 30 minutes of a node's first check-in. To speed up the process and avoid the wait, do the following:

1. **On the console**, use the sidebar to navigate to the __mcollective__ group:

   ![the mcollective group link](./images/quick/mcollective_link.png)

2. Check the list of nodes at the bottom of the page for `agent1.example.com` --- depending on your timing, it may already be present. If so, skip to "on each agent node" below.
3. If `agent1` is not a member of the group already, click the __Edit__ button:

   ![the edit button](./images/quick/mcollective_edit.png)

4. In the __nodes__ field, begin typing `agent1.example.com`'s name. You can then select it from the list of autocompletion guesses. Click the __Update__ button after you have selected it.

   ![the nodes field](./images/quick/default_nodes.png)

5. **On each agent node**, run `puppet agent --test` again, [as described above](#testing-the-agent-nodes). Note the long string of log messages related to the `pe_mcollective` class.

In a normal environment, you would usually skip these steps and allow orchestration to come on-line when Puppet runs automatically.

> The agent node can now respond to orchestration messages and its resources can be viewed live in the console.

Using Live Management to Control Agent Nodes
-----

Live management uses Puppet Enterprise's orchestration features to view and edit resources in real time. It can also trigger puppet runs and perform other orchestration tasks.

1. **On the console**, click the __Live Management__ tab in the top navigation.

   ![live management](./images/quick/live_mgmt.png)

2. Note that the master and the agent nodes are all listed in the sidebar.

### Discovering Resources

1. Note that you are currently in the __Browse Resources__ tab.
2. Choose __user resources__ from the list of resource types, then click the __Find Resources__ button:

   ![the find resources button](./images/quick/find_resources.png)

3. Examine the **complete list of user accounts** found on all of the nodes currently selected in the sidebar node list. (In this case, both the master and the agent node are selected.) Most of the users will be identical, as these machines are very close to a default OS install, but some users related to the puppet master's functionality are only on one node:

   ![different users](./images/quick/different_users.png)

4. Click on any user to view details about its properties and where it is present.

The other resource types work in a similar manner. Choose the node(s) whose resources you wish to browse. Select a resource type, click __Find Resources__ to discover the resource on the selected nodes, click on one of the resulting found resources to see details about it.

### Triggering Puppet Runs

Rather than using the command line to kick off puppet runs with `puppet agent -t` one at a time, you can use live management to run Puppet on several selected nodes.

1. **On the console, in the live management page**, click the __Control Puppet__ tab.
2. Make sure one or more nodes are selected with the node selector on the left.
3. Click the `runonce` action to reveal the red __Run__ button and additional options, and then click the __Run__ button to run Puppet on the selected nodes.

> **Note**: You can't always use the `runonce` action's additional options --- with \*nix nodes, you must stop the `pe-puppet` service before you can use options like `noop`. [See this note in the orchestration section of the manual](./orchestration_puppet.html#behavior-differences-running-vs-stopped) for more details.
<br>

![The runonce action and its options](./images/quick/console_runonce.png)

You have just triggered a puppet run on several agents at once; in this case, the master and the agent node. The __runonce__ action will trigger a puppet run on every node currently selected in the sidebar.

When using this action in production deployments, select target nodes carefully, as running it on dozens or hundreds of nodes at once can strain  the Puppet master server. If you need to do an immediate Puppet run on many nodes, [you should use the orchestration command line to do a controlled run series](./orchestration_puppet.html#run-puppet-on-many-nodes-in-a-controlled-series).

Installing Modules
-----

Puppet configures nodes by applying classes to them. Classes are chunks of Puppet code that configure a specific aspect or feature of a machine.

Puppet classes are **distributed in the form of modules**. You can save time by **using pre-existing modules**. Pre-existing modules are distributed on the [Puppet Forge](http://forge.puppetlabs.com), and **can be installed with the `puppet module` subcommand**. Any module installed on the Puppet master can be used to configure agent nodes.

### Installing a Forge Module

We will install a Puppet Enterprise supported module: `puppetlabs-ntp`. While you can use any module available on the Forge, PE customers can take advantage of [supported modules](http://forge.puppetlabs.com/supported) which are supported, tested, and maintained by Puppet Labs.

1. **On your control workstation**, point your browser to [http://forge.puppetlabs.com/puppetlabs/ntp](http://forge.puppetlabs.com/puppetlabs/ntp). This is the Forge listing for a module that installs, configures, and manages the NTP service.

2. **On the puppet master**, run `puppet module search ntp`. This searches for modules from the Puppet Forge with `ntp` in their names or descriptions and results in something like:

        Searching http://forgeapi.puppetlabs.com ...
        NAME             DESCRIPTION                                                 AUTHOR        KEYWORDS
        puppetlabs-ntp   NTP Module                                                  @puppetlabs   ntp aix
        saz-ntp          UNKNOWN                                                     @saz          ntp OEL
        thias-ntp        Network Time Protocol...                                    @thias        ntp ntpd
        warriornew-ntp   ntp setup                                                   @warriornew   ntp

   We want `puppetlabs-ntp`, which is the PE supported NTP module. You can view detailed info about the module in the "Read Me" on the Forge page you just visited: <http://forge.puppetlabs.com/puppetlabs/ntp>.

4. Install the module by running `puppet module install puppetlabs-ntp`:

        Preparing to install into /etc/puppetlabs/puppet/modules ...
        Notice: Downloading from http://forgeapi.puppetlabs.com ...
        Notice: Installing -- do not interrupt ...
        /etc/puppetlabs/puppet/modules
        └── puppetlabs-ntp (v3.0.1)

> You have just installed a Puppet module. All of the classes in it are now available to be added to the console and assigned to nodes.

There are many more modules, including PE supported modules, on [the Forge](http://forge.puppetlabs.com). In part two of this guide you'll learn more about modules, including customizing and writing your own modules on either  [Windows](./quick_writing_windows.html) or [*nix](./quick_writing_nix.html) platforms.

### Using Modules in the PE Console

[classbutton]: ./images/quick/add_class_button.png
[add_ntp]: ./images/quick/add_ntp.png
[assign_ntp_default]: ./images/quick/assign_ntp_default.png
[ntp-params]: ./images/quick/ntp-params.png

Every module contains one or more **classes**. [Classes](/puppet/3.6/reference/lang_classes.html) are named chunks of puppet code and are the primary means by which Puppet configures nodes. The module you just installed contains a class called `ntp`. To use any class, you must first **tell the console about it** and then **assign it to one or more nodes**.

1. **On the console**, click the __Add classes__ button in the sidebar:

   ![The console's add classes button][classbutton]

2. Locate the `ntp` class in the list of classes, and click its checkbox to select it. Click the __Add selected classes__ button at the bottom of the page.

   ![the add class field][add_ntp]

3. Navigate to the __default__ group page (by clicking the link in the __Groups__ menu in the sidebar), click the __Edit__ button, and begin typing "`ntp`" in the __Classes__ field; you can select the `ntp` class from the list of autocomplete suggestions. Click the __Update__ button after you have selected it.

   ![assigning the ntp class to default group][assign_ntp_default]

4. Note that the `ntp` class now appears in the list of classes for the __default__ group. Also note that the __default__ group contains your master and agent.
5. Navigate to the live management page, and select the __Control Puppet__ tab. Use the __runonce__ action to trigger a puppet run on both the master and the agent. This will configure the nodes using the newly-assigned classes. Wait one or two minutes.
6. On the agent, stop the NTP service.

   **Note**: the NTP service name may vary depending on your operating system; for example, on Debian nodes, the service name is "NTP."
7. Run `ntpdate us.pool.ntp.org`. The result should resemble the following:

   `28 Jan 17:12:40 ntpdate[27833]: adjust time server 50.18.44.19 offset 0.057045 sec`

8. Finally, restart the NTP service.

> Puppet is now managing NTP on the nodes in the __default__ group. So, for example, if you forget to restart the NTP service on one of those nodes after running `ntpdate`, PE will automatically restart it on the next puppet run.

#### Setting Class Parameters

You can use the console to set the values of the class parameters of nodes by selecting a node and then clicking __Edit parameters__ in the list of classes. For example, you want to specify an NTP server for a given node.

1. Click a node in the node list.
2. On the node view page, click the __Edit__ button.
3. Find __NTP__ in the class list, and click __Edit Parameters__.

   ![the NTP parameters list][ntp-params]

4. Enter a value for the parameter you wish to set (e.g., `time.apple.com`) in the box next to the __servers__ parameter.

  The grey text that appears as values for some parameters is the default value, which can be either a literal value or a Puppet variable. You can restore this value with the __Reset value__ control that appears next to the value after you have entered a custom value.

For more information, see the page on [classifying nodes with the console](./console_classes_groups.html).

### Viewing Changes with Event Inspector

[EI-default]: ./images/quick/EI_default.png
[EI-class_change]: ./images/quick/EI_class-change.png
[EI-detail]: ./images/quick/EI_detail.png

The event inspector lets you view and research changes and other events. Click the __Events__ tab in the main navigation bar. The event inspector window is displayed, showing the default view: classes with failures. Note that in the summary pane on the left, one event, a successful change, has been recorded for Nodes. However, there are two changes for Classes and Resources. This is because the NTP class loaded from the Puppetlabs-ntp module contains additional classes---a class that handles the configuration of NTP (`Ntp::Config`)and a class that handles the NTP service (`Ntp::Service`).

![The default event inspector view][EI-default]

You can click on events in the summary pane to inspect them in detail. For example, if you click __With Changes__ in the __Classes: with events__ summary view, the main pane will show you that the `Ntp::Config` and `Ntp::Service` classes were successfully added when you triggered the last puppet run.

![Viewing a successful change][EI-class_change]

You can keep clicking to drill down and see more detail. You can click the previous arrow (left of the summary pane), the bread-crumb trail at the top of the page, or bookmark a page for later reference (but note that after subsequent puppet runs, the bookmarks may be different when you revisit them). Eventually, you will end up at a run summary that shows you the details of the event. For example, you can see exactly which piece of puppet code was responsible for generating the event; in this case, it was line 15 of the `service.pp` manifest and line 21 of the `config.pp` manifest.

![Event detail][EI-detail]

If PE was unable to apply this class, the information would tell you exactly what piece of code you need to fix. In this case, you can see that PE is now managing NTP.

In the upper right corner of the detail pane is a link to a run report which contains information about the puppet run that made the change, including metrics about the run, logs, and more information. Visit the [reports page](./console_reports.html#reading-reports) for more information.

Summary
-----

You have now experienced the core features and workflows of Puppet Enterprise. In summary, a Puppet Enterprise user will:

* Install the PE agent on nodes they wish to manage ([\*nix](./install_basic.html) and [Windows](./install_windows.html) instructions), and [add the nodes by approving their certificate requests](./console_cert_mgmt.html).
* Use [pre-built, PE supported modules from the Puppet Forge](http://forge.puppetlabs.com) to save time and effort.
* [Assign classes from modules to nodes in the console.](./console_classes_groups.html)
* [Use the console to set values for class parameters.](./console_classes_groups.html)
* [Allow nodes to be managed by regularly scheduled Puppet runs.](./puppet_overview.html#when-new-configurations-take-effect)
* Use [live management](./console_navigating_live_mgmt.html) to [inspect and compare nodes](./orchestration_resources.html), and to [trigger on-demand puppet agent](./orchestration_puppet.html) runs when necessary.
* Use [event inspector](./console_event-inspector.html) to learn more about events that occurred during puppet runs, such as what was changed or why something failed.

### Next Steps

Beyond what this brief walkthrough has covered, most users will go on to:

* Edit Forge modules to customize them to your infrastructure's needs.
* Create new modules from scratch by writing classes that manage resources.
* Use a **site module** to compose other modules into machine roles, allowing console users to control policy instead of implementation.
* Configure multiple nodes at once by adding classes to groups in the console instead of individual nodes.

To learn about these workflows, continue to part two of this quick start guide. Choose from either the [Windows](./quick_writing_windows.html) or the [Linux](./quick_writing_nix.html) tracks.

#### Other Resources

Puppet Labs offers many opportunities for learning and training, from formal certification courses to guided on-line lessons. We've noted a few below; head over to the [learning Puppet page](https://puppetlabs.com/learn) to discover more.

* [Learning Puppet](/learning/) is a series of exercises on various core topics on deploying and using PE.  It includes the [Learning Puppet VM](http://info.puppetlabs.com/download-learning-puppet-VM.html) which provides PE pre-installed and configured on VMware and VirtualBox virtualization platforms.
* The Puppet Labs workshop contains a series of self-paced, online lessons that cover a variety of topics on Puppet basics. You can sign up at the [learning page](https://puppetlabs.com/learn).
* To explore the rest of the PE user's manual, use the sidebar at the top of this page, or [return to the index](./index.html).

* * *

- Next: [Quick Start: Writing Modules (Windows)](./quick_writing_windows.html) or [Quick Start Writing Modules (Linux)](./quick_writing_nix.html)
