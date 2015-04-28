---
layout: default
title: "PE 3.8 » Deploying PE » Non-Root Agents"
subtitle: "Running PE Agents without Root Privileges"
canonical: "/pe/latest/deploy_nonroot-agent.html"
---

**IMPORTANT**: these procedures assume some degree of experience with Puppet Enterprise (PE). If you are new to PE, we strongly recommend you work through the [Quick Start Guide](./quick_start.html) and some of our other educational resources before attempting to implement non-root agent capability.

Configuring Non-root Agent Access: Overview
------

In some circumstances, users without root access privileges may need to run the Puppet agent. For example, consider the following use case:

For security or organizational reasons, your infrastructure’s platform is maintained by one team with root privileges while your infrastructure’s applications are managed by a separate team (or teams) with diminished privileges. The applications team would like to be able to manage its part of the infrastructure using Puppet Enterprise, but the platform team cannot give them root privileges. So, the applications team needs a way to run Puppet without root privileges. In this scenario, PE is only used for application management, which is performed by a single (applications) team. The platform team does not use PE to manage any of the application team’s nodes.

PE is installed with root privileges, so you will need a root user to set up and provide non-root access to a monolithic PE master. The root user who performs this installation, will set up the non-root user(s) on the master and any nodes running a Puppet agent.

When you or another user are set up as a non-root user, you will have a reduced set of configuration management tasks that you can perform. As a non-root user, you will be able to configure Puppet settings (i.e., edit `~/.puppet/puppet.conf`), configure Facter external facts, run `puppet agent --test`, and run Puppet via non-privileged cron jobs (or a similar scheduling service). You can classify your nodes by writing or editing manifests in the directories where you have write privileges.

> **Note**: Non-root users are not able to use PE’s orchestration capabilities to manage your nodes, and Mcollective must be disabled on all nodes.

### Installation & Configuration

To properly configure non-root agent access, you will need to:

   * Install a monolithic PE master
   * Install and configure PE agents, disable the `pe-puppet` service on all nodes, and create non-root users
   * Verify the non-root configuration

#### Install and Configure a Monolithic Master

1. As a root user, install and configure a monolithic PE master. Use the [monolithic installation method](./install_pe_mono.html), or use [an answer file](./install_automated.html) to automate your installation.
2. If you've enabled live management, disable it.

   To disable live management, edit `/etc/puppetlabs/puppet-dashboard/settings.yml` and set the `disable_live_management` setting to `true`. Note that you will need to run `service pe-httpd restart` after editing `settings.yml`.

3. Use the PE console to make sure no new agents can get added to the MCollective group.

   a. From the navigation bar, click **Classification**.

   b. From the **Classificiation** page, click **PE_MCollective**.

   c. Click the **Rules** tab.

   d. Under **Fact**, locate **pe_version** and click **Remove**.

   e. Click the Commit change button.

![Remove the pe_version rule][remove_pe_version]

#### Install and Configure PE Agents and Create Non-Root Users

1. On each agent node, install a PE agent while logged in as a root user. Refer to the [instructions for installing agents](/pe/latest/install_basic.html#installing-agents).

2. As **a root user**, log in to an agent node, and add the non-root user with `puppet resource user <UNIQUE NON-ROOT USERNAME> ensure=present managehome=true`.

   >**Note**: Each and every non-root user *must* have a unique name.

3. As **a root user**, still on the agent node, set the non-root user’s password. For example, on most *nix systems you would run `passwd <USERNAME>`.

4. By default, the `pe-puppet` service runs automatically as a root user, so it needs to be disabled. As **a root user** on the agent node, stop the service by running `puppet resource service pe-puppet ensure=stopped enable=false`.

5. Disable the MCollective service on the agent node. **As a root user**, run `puppet resource service pe-mcollective ensure=stopped enable=false`.

6. Change to the non-root user.

   >**Tip**: If you wish to use `su - <NON-ROOT USERNAME>` to switch between accounts, make sure to use the `-` (`-l` in some unix variants) argument so that full login privileges are correctly granted. Otherwise you may see "permission denied" errors when trying to apply a catalog.

7. As the **non-root user**, generate and submit the cert for the agent node. From the agent node, execute the following command:

    `puppet agent -t --certname "<UNIQUE NON-ROOT USERNAME.HOSTNAME>" --server "<PUPPET MASTER HOSTNAME>"`

    This Puppet run will submit a cert request to the master and will create a `~/.puppet` directory structure in the non-root user’s home directory.

8. **As an admin user**, log into the console, navigate to the [pending node requests](./console_cert_mgmt.html), and accept the requests from non-root user agents.

    **Note**: It is possible to also sign the root user certificate in order to allow that user to also manage the node. However, you should do so only with great caution as this introduces the possibility of unwanted behavior and potential security issues. For example, if your site.pp has no default node configuration, running agent as non-admin could lead to unwanted node definitions getting generated using alt hostnames, which is a potential security issue. In general, if you deploy this scenario, you should ensure that the root and non-root users never try to manage the same resources,ensure that they have clear-cut node definitions, and ensure that classes scope correctly.


9. As the **non-root user**, run `puppet config set certname <UNIQUE NON-ROOT USERNAME.HOSTNAME> --section agent`.
10. As the **non-root user**, run `puppet config set server <PUPPET MASTER HOSTNAME> --section agent`.

   Steps 7 and 8 create and set the configuration for the non-root agent's `puppet.conf`, created in `/.puppet` in the non-root user's home directory.

		[main]
		 certname = <UNIQUE NON-ROOT USERNAME.HOSTNAME>
		 server = <PUPPET MASTER HOSTNAME>


11. You can now connect the non-root agent node to the master and get PE to configure it. Log into the agent node as the non-root user and run `puppet agent -t`.

   PE should now run and apply the configuration specified in the catalog. Keep an eye on the output from the run——if you see Facter facts being created in the non-root user’s home directory, you know that you have successfully created a functional non-root agent.

   ![non-root user first run output][nonrootuser_first_run]

#### Verify the Non-Root Configuration

Check the following to make sure the agent is properly configured and functioning as desired:

- The non-root agent node should be able to request certificates and be able to download and apply the catalog from the master without issue when a non-privileged user executes `puppet agent -t`.
- The Puppet agent service should not be running. Check it with `service pe-puppet status`.
- You can check the PE console to ensure that the non-root user node is not listed in the **PE MCollective** group.
- Non-privileged users should be able to collect existing facts by running `facter` on agents, and they should be able to define new, external Facter facts.

#### Install and Configure Windows Agents and Their Certificates

If you need to run agents without admin privileges on nodes running a Windows OS, take the following steps:

1. Connect to the agent node as an admin user and install the [Windows agent](./install_windows.html).

2. As an admin user, add the non-admin user with the following command: `puppet resource user <unique non-admin username> ensure=present managehome=true password="puppet" groups="Users"`.

    **Note:** Each and every non-admin user *must* have a unique name. If the non-privileged user needs remote desktop access, edit the user resource to include the "Remote Desktop Users" group.

3. While still connected as an admin user, disable the pe-puppet service with `puppet resource service pe-puppet ensure=stopped enable=false`.

4. While still connected as **an admin user**, disable the pe-mcollective service with `puppet resource service pe-mcollective ensure=stopped enable=false`.

5. Log out of the Windows agent machine and log back in as the non-admin user, and then run the following command:

   `puppet agent -t --certname "<unique non-privileged username>" --server "<master hostname>"`

   This Puppet run will submit a cert request to the master and will create a `~/.puppet` directory structure in the non-root user’s home directory.

6. As the non-admin user, create a Puppet configuration file (`%USERPROFILE%/.puppet/puppet.conf`) to specify the agent certname and the hostname of the master:

        [main]
        certname = <unique non-privileged username.hostname>
        server = <master hostname>

7. While still connected as the non-admin user, send a cert request to the master by running Puppet with `puppet agent -t`.

8. On the master node, as an admin user, sign the non-root certificate request using the console or by running `puppet cert sign nonrootuser`.

    **Note**: It is possible to also sign the root user certificate in order to allow that user to also manage the node. However, you should do so only with great caution as this introduces the possibility of unwanted behavior and potential security issues. For example, if your site.pp has no default node configuration, running agent as non-admin could lead to unwanted node definitions getting generated using alt hostnames, a potential security issue. In general, then, if you deploy this scenario you should be careful to ensure the root and non-root users never try to manage the same resources, have clear-cut node definitions, ensure that classes scope correctly, and so forth.

9. On the agent node, verify that the agent is connected and working by again starting a Puppet run while logged in as the non-admin user. Running `puppet agent -t` should download and process the catalog from the master without issue.

###Usage

Non-root users can only use a subset of PE’s functionality. Basically, any operation that requires root privileges (e.g., installing system packages) cannot be managed by a non-root Puppet agent.

On ***nix systems**, as non-root agent you should be able to enforce the following resource types:

* `cron` (only non-root cron jobs can be viewed or set)
* `exec` (cannot run as another user or group)
* `file` (only if the non-root user has read/write privileges)
* `notify`
* `schedule`
* `ssh_key`
* `ssh_authorized_key`
* `service`
* `augeas`

You should also be able to inspect the following resource types (use `puppet resource <resource type>`):

* `host`
* `mount`
* `package`

On **windows systems** as non-admin user you should be able to enforce the following resource types :

* `exec`
* `file`

You should also be able to inspect the following resource types (use `puppet resource <resource type>`):

* `host`
* `package`
* `user`
* `group`
* `service`


#### Issues & Warnings

 - When running a cron job as non-root user, using the `-u` flag to set a user with root privileges will cause the job to fail, resulting in the following error message:
 `Notice: /Stage[main]/Main/Node[nonrootuser]/Cron[illegal_action]/ensure: created must be privileged to use -u`


[remove_pe_version]: ./images/console/remove_pe_version.png
[nonrootuser_first_run]: ./images/console/nonrootuser_first_run.png
[nonroot_no_mco_group]: ./images/console/nonroot_no_mco_group.png

* * *

- [Next: Beginners Guide to Modules](/guides/module_guides/bgtm.html)
