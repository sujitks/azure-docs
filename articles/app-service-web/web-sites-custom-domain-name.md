﻿---
title: Map a custom domain name to an Azure app
description: Learn how to map a custom domain name (vanity domain) to your app in Azure App Service.
services: app-service
documentationcenter: ''
author: cephalin
manager: wpickett
editor: jimbe
tags: top-support-issue

ms.assetid: 48644a39-107c-45fb-9cd3-c741974ff590
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/27/2016
ms.author: cephalin

---
# Map a custom domain name to an Azure app
[!INCLUDE [web-selector](../../includes/websites-custom-domain-selector.md)]

This article shows you how to manually map a custom domain name to your web app, mobile app backend, or API app in [Azure App Service](../app-service/app-service-value-prop-what-is.md). 

Your app already comes with a unique subdomain of azurewebsites.net. For example, if the name of your app is **contoso**, then its domain name is 
**contoso.azurewebsites.net**. However, you can map a custom domain name to app so that its URL, such
as `www.contoso.com`, reflects your brand.

> [!NOTE]
> Get help from Azure experts on the [Azure forums](https://azure.microsoft.com/support/forums/). 
> For even higher level of support, go to the [Azure Support site](https://azure.microsoft.com/support/options/) and click **Get Support**.
> 
> 

[!INCLUDE [introfooter](../../includes/custom-dns-web-site-intro-notes.md)]

## Buy a new custom domain in Azure portal
If you haven't already purchased a custom domain name, you can buy one and manage it directly in your app's settings in the 
[Azure portal](https://portal.azure.com). This option makes it easy to map a custom domain to your app, whether your app 
uses [Azure Traffic Manager](web-sites-traffic-manager-custom-domain-name.md) or not. 

For instructions, see [Buy a custom domain name for App Service](custom-dns-web-site-buydomains-web-app.md).

## Map a custom domain you purchased externally
If you have already purchased a custom domain from [Azure DNS](https://azure.microsoft.com/services/dns/) or from a third-party provider, 
there are three main steps to map the custom domain to your app:

1. [*(A record only)* Get app's IP address](#vip).
2. [Create the DNS records that map your domain to your app](#createdns). 
   * **Where**: your domain registrar's own management tool (e.g. Azure DNS, GoDaddy, etc.).
   * **Why**: so your domain registrar knows to resolves the desired custom domain to your Azure app.
3. [Enable the custom domain name for your Azure app](#enable).
   * **Where**: the [Azure portal](https://portal.azure.com).
   * **Why**: so your app knows to respond to requests made to the custom domain name.
4. [Verify DNS propagation](#verify).

### Types of domains you can map
Azure App Service lets you map the following categories of custom domains to your app.

* **Root domain** - the domain name that you reserved with the domain registrar (represented by the `@` host record, typically). 
  For example, **contoso.com**.
* **Subdomain** - any domain that's under your root domain. For example, **www.contoso.com** (represented by the `www` host record).  You can 
  map different subdomains of the same root domain to different apps in Azure.
* **Wildcard domain** - [any subdomain whose leftmost DNS label is `*`](https://en.wikipedia.org/wiki/Wildcard_DNS_record) 
  (e.g. host records `*` and `*.blogs`). For example, **\*.contoso.com**.

### Types of DNS records you can use
Depending on your need, you can use two different types of standard DNS records to map your custom domain: 

* [A](https://en.wikipedia.org/wiki/List_of_DNS_record_types#A) - maps your custom domain name to the Azure app's virtual 
  IP address directly. 
* [CNAME](https://en.wikipedia.org/wiki/CNAME_record) - maps your custom domain name to your app's Azure domain name, 
  **&lt;*appname*>.azurewebsites.net**. 

The advantage of CNAME is that it persists across IP address changes. If you delete and recreate your app, or change from a higher pricing 
tier back to the **Shared** tier, your app's virtual IP address may change. Through such a change,
a CNAME record is still valid, whereas an A record requires an update. 

The tutorial shows you steps for using the A record and also for using the CNAME record.

> [!IMPORTANT]
> Do not create a CNAME record for your root domain (i.e. the "root record"). For more information, see 
> [Why can't a CNAME record be used at the root domain](http://serverfault.com/questions/613829/why-cant-a-cname-record-be-used-at-the-apex-aka-root-of-a-domain).
> To map a root domain to your Azure app, use an A record instead.
> 
> 

<a name="vip"></a>

## Step 1. *(A record only)* Get app's IP address
To map a custom domain name using an A record, you need your Azure app's IP address. If you will map using a CNAME record
instead, skip this step and move onto the next section.

1. Log in to the [Azure portal](https://portal.azure.com).
2. Click **App Services** on the left menu.
3. Click your app, then click **Custom domains**.
4. Take note of the IP address above Hostnames section..
   
   ![Map custom domain name with A record: Get IP address for your Azure App Service app](./media/web-sites-custom-domain-name/virtual-ip-address.png)
5. Keep this portal blade open. You will come back to it once you create the DNS records.

<a name="createdns"></a>

## Step 2. Create the DNS record(s)
Log in to your domain registrar and use their tool to add an A record or CNAME record. Every registrar’s UI is slightly 
different, so you should consult your provider's documentation. However, here are some general guidelines.

1. Find the page for managing DNS records. Look for links or areas of the site labeled **Domain Name**, **DNS**, or 
   **Name Server Management**. Often, you can find the link by viewing your account information, and then looking for a link 
   such as **My domains**.
2. Look for a link that lets you add or edit DNS records. This might be a **Zone file** or **DNS Records** link, or 
   an **Advanced** configuration link.
3. Create the record and save your changes.
   * [Instructions for an A record are here](#a).
   * [Instructions for a CNAME record are here](#cname).

<a name="a"></a>

### Create an A record
To use an A record to map to your Azure app's IP address, you actually need to create both an A record and a TXT record. 
The A record is for the DNS resolution itself, and the TXT record is for Azure to verify that you own the custom domain 
name. 

Configure your A record as follows (@ typically represents the root domain):

<table cellspacing="0" border="1">
  <tr>
    <th>FQDN example</th>
    <th>A Host</th>
    <th>A Value</th>
  </tr>
  <tr>
    <td>contoso.com (root)</td>
    <td>@</td>
    <td>IP address from <a href="#vip">Step 1</a></td>
  </tr>
  <tr>
    <td>www.contoso.com (sub)</td>
    <td>www</td>
    <td>IP address from <a href="#vip">Step 1</a></td>
  </tr>
  <tr>
    <td>\*.contoso.com (wildcard)</td>
    <td>\*</td>
    <td>IP address from <a href="#vip">Step 1</a></td>
  </tr>
</table>

Your additional TXT record takes on the convention that maps from &lt;*subdomain*>.&lt;*rootdomain*> to 
&lt;*appname*>.azurewebsites.net. Configure your TXT record as follows:

<table cellspacing="0" border="1">
  <tr>
    <th>FQDN example</th>
    <th>TXT Host</th>
    <th>TXT Value</th>
  </tr>
  <tr>
    <td>contoso.com (root)</td>
    <td>@</td>
    <td>&lt;<i>appname</i>>.azurewebsites.net</td>
  </tr>
  <tr>
    <td>www.contoso.com (sub)</td>
    <td>www</td>
    <td>&lt;<i>appname</i>>.azurewebsites.net</td>
  </tr>
  <tr>
    <td>\*.contoso.com (wildcard)</td>
    <td>\*</td>
    <td>&lt;<i>appname</i>>.azurewebsites.net</td>
  </tr>
</table>

<a name="cname"></a>

### Create a CNAME record
If you use a CNAME record to map to your Azure app's default domain name, you don't need an additional TXT record
like you do with an A record. 

> [!IMPORTANT]
> Do not create a CNAME record for your root domain (i.e. the "root record"). For more information, see 
> [Why can't a CNAME record be used at the root domain](http://serverfault.com/questions/613829/why-cant-a-cname-record-be-used-at-the-apex-aka-root-of-a-domain).
> To map a root domain to your Azure app, use an [A record](#a) instead.
> 
> 

Configure your CNAME record as follows (@ typically represents the root domain):

<table cellspacing="0" border="1">
  <tr>
    <th>FQDN example</th>
    <th>CNAME Host</th>
    <th>CNAME Value</th>
  </tr>
  <tr>
    <td>www.contoso.com (sub)</td>
    <td>www</td>
    <td>&lt;<i>appname</i>>.azurewebsites.net</td>
  </tr>
  <tr>
    <td>\*.contoso.com (wildcard)</td>
    <td>\*</td>
    <td>&lt;<i>appname</i>>.azurewebsites.net</td>
  </tr>
</table>

<a name="enable"></a>

## Step 3. Enable the custom domain name for your app
Back in the **Custom Domains** blade in the Azure portal (see [Step 1](#vip)), you need to add the fully-qualified
domain name (FQDN) of your custom domain to the list.

1. If you haven't done so, log in to the [Azure portal](https://portal.azure.com).
2. In the Azure portal, click **App Services** on the left menu.
3. Click your app, then click **Custom domains** > **Add hostname**.
4. Add the FQDN of your custom domain to the list (e.g. **www.contoso.com**).
   
   ![Map a custom domain name to an Azure app: add to list of domain names](./media/web-sites-custom-domain-name/add-custom-domain.png)
   
   > [!NOTE]
   > Azure will attempt to verify the domain name that you use here. Be sure that it is the same domain name
   > for which you created a DNS record in [Step 2](#createdns). 
   > 
   > 
5. Click **Validate**.
6. Upon clicking **Validate** Azure will kick off Domain Verification workflow. This will check for Domain ownership as well as Hostname availability and report success or detailed error with prescriptive guidence on how to fix the error.    
7. Upon successful validation **Add hostname** button will become active and you will be able to the assign hostname. 
8. Once Azure finishes configuring your new custom domain name, navigate to your custom domain name in a browser. The browser should open your Azure app, which means that your custom domain name is configured properly.

## Migrate an active domain with no downtime 

When you migrate a live site and its domain name to App Service, that domain name is already serving live traffic, and you don't want any downtime in DNS resolution during
the migration process. In this case, you need to preemptively bind the domain name to your Azure app for domain verification. To do this, follow the modified steps below:

1. First, create a verification TXT record with your DNS registry by following the steps at [Step 2. Create the DNS record(s)](#createdns).
Your additional TXT record takes on the convention that maps from &lt;*subdomain*>.&lt;*rootdomain*> to &lt;*appname*>.azurewebsites.net.
See the following table for examples:  
 
    <table cellspacing="0" border="1">
    <tr>
    <th>FQDN example</th>
    <th>TXT Host</th>
    <th>TXT Value</th>
    </tr>
    <tr>
    <td>contoso.com (root)</td>
    <td>awverify.contoso.com</td>
    <td>&lt;<i>appname</i>>.azurewebsites.net</td>
    </tr>
    <tr>
    <td>www.contoso.com (sub)</td>
    <td>awverify.www.contoso.com</td>
    <td>&lt;<i>appname</i>>.azurewebsites.net</td>
    </tr>
    <tr>
    <td>\*.contoso.com (wildcard)</td>
    <td>awverify.\*.contoso.com</td>
    <td>&lt;<i>appname</i>>.azurewebsites.net</td>
    </tr>
    </table>

2. Then, add your custom domain name to your Azure app by following the steps at [Step 3. Enable the custom domain name for your app](#enable).

    Your custom domain is now enabled in your Azure app. The only thing left to do is to update the DNS record with your domain registrar.

3. Finally, update your domain's DNS record to point to your Azure app as is shown in [Step 2. Create the DNS record(s)](#createdns). 

    User traffic should be redirected to your Azure app immediately after DNS propagation happens.

<a name="verify"></a>

## Verify DNS propagation
After you finish the configuration steps, it can take some time for the changes to propagate, depending on your DNS provider. You can verify that the DNS propagation is working as expected by using [http://digwebinterface.com/](http://digwebinterface.com/). After you browse to the site, specify the hostnames in the textbox and click **Dig**. Verify the results to confirm if the recent changes have taken effect.  

![Map a custom domain name to an Azure app: verify DNS propagation](./media/web-sites-custom-domain-name/1-digwebinterface.png)

> [!NOTE]
> The propagation of the DNS entries can take up to 48 hours (sometimes longer). If you have configured everything correctly, you still need to wait for the propagation to succeed.
> 
> 

## Next steps
Learn how to secure your custom domain name with HTTPS by [buying an SSL certificate in Azure](web-sites-purchase-ssl-web-site.md) or [using an SSL certificate from elsewhere](web-sites-configure-ssl-certificate.md).

> [!NOTE]
> If you want to get started with Azure App Service before signing up for an Azure account, go to [Try App Service](http://go.microsoft.com/fwlink/?LinkId=523751), where you can immediately create a short-lived starter web app in App Service. No credit cards required; no commitments.
> 
> 

[Get started with Azure DNS](../dns/dns-getstarted-create-dnszone.md)  
[Create DNS records for a web app in a custom domain](../dns/dns-web-sites-custom-domain.md)  
[Delegate Domain to Azure DNS](../dns/dns-domain-delegation.md)

<!-- Images -->
[subdomain]: media/web-sites-custom-domain-name/azurewebsites-subdomain.png
