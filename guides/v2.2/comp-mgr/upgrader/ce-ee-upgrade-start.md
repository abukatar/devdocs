---
group: compman
title: Upgrade from Open Source to Commerce
version: 2.2
github_link: comp-mgr/upgrader/ce-ee-upgrade-start.md
functional_areas:
  - Upgrade
---

## Overview of {{site.data.var.ce}} to {{site.data.var.ee}} upgrade {#compman-overview}

This section discusses how to upgrade {{site.data.var.ce}} to {{site.data.var.ee}}.

<div class="bs-callout bs-callout-info" id="info">
You must be authorized for {{site.data.var.ee}} to perform the tasks discussed in this topic.
</div>

## Prerequisites {#compman-prereq}

Before continuing, complete all tasks discussed in [Prerequisites].

In addition, you might need to install the {% glossarytooltip bf703ab1-ca4b-48f9-b2b7-16a81fd46e02 %}PHP{% endglossarytooltip %} [`bcmath`] extension, which is required by {{site.data.var.ee}}. Examples follow:

*	CentOS (using the `webtatic` repository): `yum -y install php56w-bcmath`
*	Ubuntu (using the `ppa:ondrej/php5-5.6` repository): `apt-get -y install php5-bcmath`

<div class="bs-callout bs-callout-info" id="info">
Make sure you are authorized for {{site.data.var.ee}} access before you continue. Contact [Magento Support] if you have questions.
</div>

## Start System Upgrade from the Magento Admin {#compman-access}

To run System Upgrade:

1.	Log in to the {% glossarytooltip 18b930cf-09cc-47c9-a5e5-905f86c43f81 %}Magento Admin{% endglossarytooltip %} as an administrator.
2.	Click **System** > **Web Setup Wizard**.
	The following page displays.

    ![Specify whether to manage components or upgrade Magento]

3.	Click **System Upgrade**.

    Magento begins searching for core module updates immediately. To also search for component updates, click **Yes**. A sample follows:

    ![Magento begins searching for upgrades right away]

    The page displays similar to the following when we find components to upgrade.

    ![Magento finds software to upgrade]

       From the list, click the version to which to upgrade. Typically, you'll choose the most recent version (indicated by **(latest)**.)

After the upgrade completes, restart Varnish if you use it for page caching.

	service varnish restart

#### Errors

*	The following error can indicate one of several issues, including that you haven't entered your [authentication keys] in the Magento Admin:

	![Sorry we can't take that action right now]

    For suggested solutions to other causes indicated by this message, see [troubleshooting].

*	The following error might display:

		[2016-01-19 23:33:24 UTC] An error occurred while executing job
		"setup:upgrade {"command":"setup:upgrade"}": Could not complete
		setup:upgrade {"command":"setup:upgrade"} successfully: Source
		class "\Cybersource" for "CybersourceLogger" generation does not exist.

	For more information, see [Error upgrading from CE to EE].

## Continue your upgrade {#ce-ee-continue}

From here, your upgrade is the same as any other upgrade. Continue with [Step 1. Select versions to upgrade].

<!-- Link Definitions -->
[Prerequisites]: {{ page.baseurl }}/comp-mgr/prereq/prereq_compman.html
[`bcmath`]: http://php.net/manual/en/book.bc.php
{:target="_blank"}
[Magento Support]: http://support.magentocommerce.com
{:target="_blank"}
[authentication keys]: {{ page.baseurl }}/install-gde/prereq/connect-auth.html
[troubleshooting]: {{ page.baseurl }}/comp-mgr/trouble/cman/were-sorry.html
[Error upgrading from CE to EE]: {{ page.baseurl }}/comp-mgr/trouble/cman/ce-ee-upgrade.html
[Step 1. Select versions to upgrade]: {{ page.baseurl }}/comp-mgr/upgrader/upgrade-main-pg.html

<!-- Image Definitions -->
[Specify whether to manage components or upgrade Magento]: {{ site.baseurl }}/common/images/cman_upgr_initial.png
{:width="650px"}
[Magento begins searching for upgrades right away]: {{ site.baseurl }}/common/images/upgr_initial-pg.png
{:width="650px"}
[Magento finds software to upgrade]: {{ site.baseurl }}/common/images/upgr-ee-version-list.png
{:width="750px"}
[Sorry we can't take that action right now]: {{ site.baseurl }}/common/images/upgr-sorry.png
{:width="600px"}
