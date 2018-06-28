---
layout: default
group: cloud
title: Configuration management for store settings
version: 2.2
github_link: cloud/live/sens-data-over.md
functional_areas:
  - Cloud
  - Deploy
---

Configuration management, or [Pipeline Deployment]({{page.baseurl}}config-guide/deployment/pipeline/), provides a way to deploy across your environments with minimal downtime. The process extracts all configuration settings from your Magento implementation into a single file. Add this file to your commit and push it across all of your environments to keep consistent settings and reduce downtime. It provides the following benefits:

*  Better way to [manage and synchronize](#cloud-confman-over) the configuration across your Integration, Staging, and Production environments.
*  Less time required to [build](#cloud-confman-scd-over) and deploy your project by moving static file deployment from deploy to the build phase. Your site is in maintenance mode until deployment completes. For details, see [Deployment Process]({{page.baseurl}}cloud/reference/discover-deploy.html).
*  Sensitive data is automatically added into an environment variables file (`/app/etc/env.php`). You can also manually add sensitive environment variables using the Project Web Interface, the CLI, or directly in the Magento Admin. For example, payment processor passwords and API keys.

These methods are optional, but strongly recommended. The process ensures faster deployments and consistent configurations across your environments.

To complete configuration management tasks, you must have a project reader role with [environment administrator]({{page.baseurl}}cloud/project/user-admin.html#cloud-role-env) privileges.

<div class="bs-callout bs-callout-info" id="info" markdown="1">
For extended technical information, see [Pipeline Deployment]({{page.baseurl}}config-guide/deployment/pipeline/). When configuring and using these features, follow this topic specifically. {{site.data.var.ece}} provides the build server, build and deploy scripts, and deployment environments. You only need to configure settings, generate the file, and deploy.
</div>

## How it works {#cloud-confman-over}
The database contains default configurations for your Magento store. When you update the configurations in the Integration, Staging, and Production environments using the Magento Admin, you generate a file of all Magento configuration settings into a single text file: `app/etc/config.php`.

To generate the file, use one of the following:

* `php vendor/bin/m2-ece-scd-dump`—**Recommended** Exports only modified configuration settings
* `php ./vendor/bin/ece-tools config:dump`—Exports every configuration setting, including modified and default settings

<div class="bs-callout bs-callout-warning" markdown="1">
For {{site.data.var.ece}}, we **do not recommend** using the `app:config:dump` command. This command pulls and locks all values in a read-only state. This affects Fastly and other important modules.
</div>

Any data that exports to the file becomes locked. The corresponding field in the Magento Admin becomes read-only. This ensures consistent configurations as you push the file across all environments. And every time you run this command, any new configurations are appended to your config.php file. If you need to modify or delete an existing configuration, you must edit the file manually.

By using the `scd-dump` command, you can configure only the settings you want copied across all environments. After you merge the code, you can configure additional settings in Staging and Production. For sensitive configurations, you can also add those settings to environment variables. For example, you may want to add different PayPal merchant account credentials for Staging (sandbox) and Production (live).

If sensitive data is found in your configurations, it is generated as environment variables to `env.php`. This file remains in the environment and should not be added to your Git environment.

### Switching between commands {#commands}
Can you switch between using `php vendor/bin/m2-ece-scd-dump` and `php ./vendor/bin/ece-tools config:dump`? Only in one specific way.

In 2.2 and later, you cannot delete the `config.php` file to regenerate it. You can only edit the file to change or remove a value. This file includes more information than just configuration values.

If you use `php vendor/bin/m2-ece-scd-dump` to generate the file, you can switch to `php ./vendor/bin/ece-tools config:dump`. Once switched, you cannot switch back. Using the command will append the file with all configured values (default and modified) not currently captured in the file. Any modified values must be entered by editing the file.

### Configuration data {#data}
System settings refer to the configurations in the Magento Admin in **Stores** > **Settings** > **Configuration**. Depending on the command used, all or just modified system configurations save to the file.

This file can include all system configuration settings for stores, sites, extensions, and static file optimization settings. System values related to static content deployment (for example, static file optimization) are also stored in `app/etc/config.php`. _Static file optimization_ means merging and minifying JavaScript and Cascading Style Sheets, and minifying HTML templates.

Sensitive values are _not_ stored in `app/etc/config.php`. Any sensitive configurations should export to `app/etc/config.php` during the `scd-dump` process. You can also create environment variables using CLI commands or the Project Web Interface.

<div class="bs-callout bs-callout-info" markdown="1">
You can set _any_ value using environment variables, but we recommend using environment variables for sensitive values.
</div>

For a list of configurable settings, see [Configuration settings you can change](#cloud-clp-settings) and [System settings reference]({{page.baseurl}}config-guide/prod/config-reference-var-name.html).

### Static content deployment performance {#cloud-confman-scd-over}
Depending on the size of your store, you may have a large amount of static content files to deploy. Normally, static content deploys during the [deploy phase]({{page.baseurl}}cloud/reference/discover-deploy.html#cloud-deploy-over-phases-hook), which is in Maintenance mode. To move the deployment of static content to the [build phase]({{page.baseurl}}cloud/reference/discover-deploy.html#cloud-deploy-over-phases-build), generate the configuration file.

If you generate `config.php`, the build and deploy hooks identify the file and deploy all static files during the build phase. This helps reduce the time spent in Maintenance mode during the deploy phase.

<div class="bs-callout bs-callout-info" markdown="1">
Before deploying static files, the build and deploy phases compress static content using `gzip`. Compressing static files reduces server loads and increases site performance. Refer to [Magento build options](http://devdocs.magento.com/guides/v2.2/cloud/env/environment-vars_magento.html#build) to learn about customizing or disabling file compression.
</div>

## Configuration selection flow
All system configurations are set during deployment according to the following override scheme:

1. If an environment variable exists, use the custom configuration and ignore the default configuration.
1. If an environment variable does not exist, use the configuration from a `MAGENTO_CLOUD_RELATIONSHIPS` name-value pair in the `.magento.app.yaml` file. Ignore the default configuration.
1. If an environment variable does not exist and `MAGENTO_CLOUD_RELATIONSHIPS` does not contain a name-value pair, remove all customized configuration and use the values from the default configuration.

## Configuration settings you can change {#cloud-clp-settings}
The following table shows the configuration settings affected by the `bin/magento magento-cloud:scd-dump` command. These are the configuration settings that you can manage in Git. If you use `php ./vendor/bin/ece-tools config:dump`, all settings are exported including default and modified settings.

The `config.php` file includes the following settings and configuration values:

* Configured values for settings entered through the Magento Admin (see the table below)
* Configured extension settings
* Scopes value for static content deployment (default is [`quick`](http://devdocs.magento.com/guides/v2.2/config-guide/cli/config-cli-subcommands-static-deploy-strategies.html#static-file-quick))

<table>
<tbody>
<tr>
<th style="width:250px;">Description</th>
<th>Path in Magento Admin: Stores > Configuration > ...</th>
</tr>
<tr>
<td>Store locale</td>
<td>General > General, Locale Options > Locale</td>
</tr>
<tr>
<td>Static asset signing</td>
<td>Advanced > Developer, Static Files Settings > Static Files Signing</td>
</tr>
<tr>
<td>Server-side or client-side LESS compilation</td>
<td>Advanced > Developer, Frontend Developer Workflow > Workflow type</td>
</tr>
<tr>
<td>HTML minification</td>
<td>Advanced > Developer, Template Settings > Minify Html</td>
</tr>
<tr>
<td>JavaScript minification</td>
<td>Advanced > Developer, JavaScript Settings > (several options)</td>
</tr>
<tr>
<td>CSS minification</td>
<td>Advanced > Developer, CSS Settings > Merge CSS Files and Minify CSS Files</td>
</tr>
<tr>
<td>Disable modules output</td>
<td>Advanced > Advanced > Disable Modules Output</td>
</tr>
</tbody>
</table>

## Recommended procedure to manage your settings {#cloud-config-specific-recomm}
Managing store configuration is a complex task that's mostly up to you. What locales do you want to use? What custom themes do you need? Only you can determine the answers to those questions. We can help you manage those settings more easily. For example, you may want to change the default locale and a store's static file optimization settings, with different settings in Staging and Production. Instead of making these changes in every environment, use `config.php`.

We **strongly recommend** using `scd-dump` to generate `config.php`. This file includes only the settings you configured without locking all default values. It also ensures all extensions used in Staging and Production do not break due to read-only configurations, especially Fastly.

To fully understand the process, please see [our extensive example]({{page.baseurl}}cloud/live/sens-data-initial.html).

The **Starter plan** environment high-level overview of this process:

![Overview of Starter configuration management]({{site.baseurl}}common/images/cloud_configmgmt-starter-2-2.png)

The **Pro plan** environment high-level overview of this process:

![Overview of Pro configuration management]({{site.baseurl}}common/images/cloud_configmgmt-pro-2-2.png)

### Step 1: Configure your store {#config-store}
Complete all configurations for your stores in the Admin console:

1. Log into the Magento Admin for one of the environments:

    * Starter: An active development branch
    * Pro: The `master` environment in Integration

2. Create and configure all store settings. These configurations do not include the actual products unless you plan on dumping the database from this environment to Staging and Production. Typically development databases don't include your full store data.
3. Open a terminal on your local and use an SSH command to generate `/app/etc/config.php` on the environment:

    `ssh <SSH URL> "<Command>"`

  For example for Pro, to run the `scd-dump` on Integration `master`:

    ssh itnu84v4m4e5k-master-ouhx5wq@ssh.us.magentosite.cloud "php vendor/bin/m2-ece-scd-dump"

**Important:** The `config.php` file includes the following settings and configuration values:

* Configured values for settings entered through the Magento Admin
* Configured extension settings
* Scopes value for static content deployment (default is [`quick`](http://devdocs.magento.com/guides/v2.2/config-guide/cli/config-cli-subcommands-static-deploy-strategies.html#static-file-quick))

### Step 2: Transfer and add the file to Git {#transfer-file}
Push `config.php` to Git. To push this file to the `master` Git branch, you need to complete a few extra steps because this environment is read-only.

1. Transfer `config.php` to your local system using `rsync` or `scp`. You can only add this file to the Git branch through your local.

    `rsync <SSH URL>:app/etc/config.php ./app/etc/config.php`

2. Add and push `config.php` to the Git master branch.

    `git add app/etc/config.php && git commit -m "Add system-specific configuration" && git push origin master`

When you add `config.php` to Git, all build and deploy processes move static content deployment to the build phase. The method for the deployment uses the scope. The default option is [`quick`](http://devdocs.magento.com/guides/v2.2/config-guide/cli/config-cli-subcommands-static-deploy-strategies.html#static-file-quick). You can change the strategy by setting an environment variable for [`SCD_STRATEGY`](http://devdocs.magento.com/guides/v2.2/cloud/env/environment-vars_magento.html#deploy).

<div class="bs-callout bs-callout-info" id="info" markdown="1">
Once this file is added to your code, you should not delete it. If you need to remove or edit settings, you must manually edit the file to make changes.
</div>

### Step 3 & 4: Push Git branch to Staging and Production {#push-git}
Log into the Magento Admin in those environments to verify the settings. If you used `scd-dump`, only configured settings display. You can continue configuring the environment if needed.

For Starter, when you push, the updated code pushes to the active environment. Merge the branch to Staging and finally `master` for Production. Complete any additional configurations in Staging and Production as needed.

For Pro, when you push to the Git branch, the Integration `master` environment updates. Deploy this branch to Staging and Production. Complete any additional configurations in Staging and Production as needed.

## Update configurations {#update}
If you need to modify or remove any existing configuration settings in `config.php`, you will need to modify the file manually with a text editor. After completing edits or removals, you can push it to Git to update.

To add new configurations, modify your environment through the Magento Admin panel and run the command again to generate the file. Any new configurations are appended to the code in the file. Push it to Git to update.

<div class="bs-callout bs-callout-warning" markdown="1">
While you can manually edit `config.php` in Staging and Production, we don't recommend it. The file helps keep all of your configurations consistent across all of your environments.

You should never delete `config.php` to rebuild it. Deleting the file can remove specific configurations and settings required for build and deploy processes.
</div>

## Migrate config.local.php to config.php {#migrate}
If you upgrade to {{site.data.var.ece}} 2.2 or later, you may want to migrate settings from `config.local.php` to your new `config.php` file. If the configuration settings in your Magento Admin match the contents of the file, you can follow the instructions to generate and add `config.php`.

If they differ, you can append content from `config.local.php` to your new `config.php` file:

1. Follow instructions to generate the `config.php` file using the [recommended method](#cloud-config-specific-recomm).
2. Open `config.php`and delete the last line.
3. Open `config.local.php`and copy the contents.
4. Paste the contents into `config.php`, save, and complete adding it to Git.
5. Deploy across your environments.

You only need to complete this migration once. When you need to update the file, you will always update the new `config.php`.