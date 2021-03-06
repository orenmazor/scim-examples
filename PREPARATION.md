# Preparing to deploy your 1Password SCIM Bridge

This guide will help you prepare to deploy your 1Password SCIM Bridge.

## Decide on URL and email address

There are a few pieces of information you'll want to decide on before beginning the setup process:

* Your SCIM bridge domain name. (example: `op-scim-bridge.example.com`)
* An email to use for the automatically-created Provision Manager user. (example: `op-scim@example.com`)


## High-Level Overview

The SCIM bridge relies on the [SCIM protocol](http://www.simplecloud.info/), and acts as an intermediary between your Identity Provider - Azure Active Directory, Okta, and others - and your 1Password instance.

It allows for automatic provisioning and deprovisioning of your 1Password user accounts and groups based on what accounts and groups you have assigned in your Identity Provider, providing a way to centralize your organization's 1Password account with other services you may be using.


### Technical Components

For general deployment, the SCIM bridge requires two services to function correctly:

* the `op-scim` service itself
* a [Redis](https://redis.io/) cache


### DNS record

You will need to be able to create a DNS record with the SCIM bridge domain name decided. However, you'll need to have the IP address of the host, which necessitates deploying the SCIM bridge first, unless you have a static IP already assigned. Follow the steps in each respective deployment guide on when to finish setting up your DNS record.


### SSL Certificates

SSL certificates are handled through the [https://letsencrypt.org/](LetsEncrypt) service which automatically generates and renews an SSL certificate based on the domain name you've decided on. On your firewall, you should ensure that the service can access Port 80 and Port 443, as Port 80 is required for the LetsEncrypt service to complete its domain challenge and issue your SCIM bridge an SSL certificate. Note that a TLS connection is mandatory for connecting to the 1Password service.


## Clone this repository

You should clone this repository to ensure you have all the files needed to begin deployment. You should also familiarize yourself with the contents of the deployment method you've selected to ensure you have a full idea of what the deployment process will do.

From the command line:

```
git clone https://github.com/1Password/scim-examples.git
```

Alternatively, you can download a .zip of the project by clicking the "Clone or download" button.


## Caveats

There are a few common issues that pop up when deploying the SCIM Bridge.

* Do not create the Provision Manager user manually. Let the setup process create the Provision Manager user for you **automatically.**
* When the Provisioning setup asks you for an email address for the new Provision Manager user it creates for you automatically, use a **dedicated email address** (for example: `op-provision-manager@example.com`) to handle this account. It is _not advised_ to use any personal email address, and additionally, this account should be accessible by whomever will manage the 1Password service for your organization, be it a single individual or a group.
* You should **never** need to log into this Provision Manager account manually. Please refrain from doing so.
* You should only run one instance of the SCIM Bridge online at a time. The SCIM Bridge is not considered a high-availablity service. Running multiple SCIM Bridges is also not supported.
* Do not attempt to perform a provisioning sync until the setup has been completed.
* Once set up, your Identity Provider becomes the _authoritative source_ of information for your 1Password accounts. With Provisioning enabled, the ability to change the _display name_ and _account status_ are not possible through the 1Password Web UI, and must be done through your Identity Provider. You can, however, continue to issue Account Recovery requests through the 1Password Web UI with Provisioning enabled.
* With v1.6.0+ of the SCIM Bridge, you can enforce e-mail address changes through your Identity Provider. Users will be required to confirm those e-mail changes the next time they log in, as their e-mail address is used when generating their encryption keys.

For more information on our security model, you can read our [security whitepaper](https://1password.com/files/1Password-White-Paper.pdf).

## Prepare your 1Password Account

Log in to your 1Password account [using this link](https://start.1password.com/settings/provisioning/setup). It will take you to the setup page for the SCIM bridge. Follow the instructions there.

During this process, the setup will guide you through the following process:

* Automatically creating a Provision Managers group
* Automatically creating a Provision Manager user
* Generating your SCIM bridge credentials


### Security (IMPORTANT)

There are a few specific considerations with respect to security.

All SCIM requests must be secured via TLS using an API gateway (self-configured web server) or the provided load balancer.

You will be provided with two separate secrets:

* a `scimsession` file
* a bearer token

The `scimsession` file contains the credentials for the new Provision Manager user the setup process automatically created for you. This user will create, confirm, and suspend users, and create and manage access to groups. These secrets are required to connect to the 1Password service.

**Do not share these secrets!**

The bearer token must be provided to your Identity Provider, but beyond that it should be kept safe and **not shared with anyone else.** The `scimsession` file should only be shared with the SCIM bridge itself.

These secrets can be used to authenticate as the Provision Manager user. It is a major security concern if they're not kept safe.

**IMPORTANT:** To reiterate, please keep these secrets in a secure location (such as within 1Password), and **don't share them** with anyone unless absolutely necessary.
