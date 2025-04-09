# Using DQS with Rspamd

This repository contains configuration files for Rspamd, (https://rspamd.com/) for use with the Spamhaus Technology Data Query Service (DQS) product.

***

### Table of contents
- [What is DQS](#what-is-dqs)?
- [What zones are available with DQS](#what-zones-are-available-with-dqs)?
- [What are the advantages of DQS](#what-are-the-advantages-of-dqs)?
- [How does DQS Perform](#how-does-dqs-perform)?
	- [HBL performance boost](#hbl-performance-boost)
- [What is the licensing for DQS](#what-is-the-licensing-for-dqs)?
- [What is the difference between paid-for and free DQS](#what-is-the-difference-between-paid-for-and-free-dqs)?
- [How do I register a DQS key](#how-do-i-register-a-dqs-key)?
- [Prerequisites](#prerequisites)
- [Conventions](#conventions)
- [Installation instructions](#installation-instructions)
- [Testing your setup](#testing-your-setup)
- [Final recommendations](#final-recommendations)
- [Support and feedback](#support-and-feedback)

***

#### What is DQS?

Data Query Service (DQS) is a set of DNSBLs with real-time updates operated by Spamhaus Technology ([https://www.spamhaus.com](https://www.spamhaus.com)).

***

#### What zones are available with DQS?

All zones, their definitions, and all possible return codes are documented [here](https://docs.spamhaus.com/10-data-type-documentation/datasets/030-datasets.html)

***

#### What are the advantages of DQS?

With DQS, Spamhaus provides real time updates instead of the one-minute-delayed updates that are used by the public mirrors and the RSYNC feed.
Sixty seconds doesn't seem much, but when dealing with hailstormers they are *crucial*. The increase in catch rate between public mirrors and DQS is mostly due to the real time updates.

Along with the above advantage, free DQS users will also get two new zones to query, Zero Reputation Domains (ZRD) and AuthBL. Paid-for DQS users will also get access to the Hash BlockList (HBL).

ZRD automatically adds newly-registered as well as previously-dormant domains to a block list for 24 hours. It also gives return codes that indicate the age of the domain (in hours) since first detection.

AuthBL is primarily designed for use by anyone operating a submission SMTP server. It is a list of IPs that are known to host bots that use stolen credentials to spam. If one of your customers gets their credentials stolen, AuthBL greatly mitigates the ability of botnets to abuse the account, and keeps your MTAs safe from collateral damage.

HBL is a zone dedicated to deal with sextortions/scam cryptowallets, dropbox emails and malicious files.

***

#### How does DQS perform?

You can [see it yourself](https://www.virusbulletin.com/testing/results/latest/vbspam-email-security). We are independently tested by Virus Bulletin, a company that tests both DQS and public mirror performances. The difference between them is that DQS catches up to 42% more spam than our public mirrors.
NOTE: Results on VBSpam are achieved by using *only* the DQS dataset, meaning that if you just add an antivirus to your email filtering setup you can potentially reach the same performance as other commercial antispam products.

#### HBL performance boost

While we know that every scenario is different, our in the field observations made using the Virus Bulletin spam feed shows that including HBL in your antispam setup could roughly boost spam detection from 0,3% up to slightly more than 1%

***

#### What is the licensing for DQS?

The usage terms are [the same](https://www.spamhaus.org/organization/dnsblusage/) as the ones for our public mirrors, meaning that if you already use our public mirrors you are entitled to a free DQS key.

***

#### What is the difference between paid-for and free DQS?

With free DQS you have access to ZRD and AuthBL, and you must abide by the [free usage policy limits](https://www.spamhaus.org/organization/dnsblusage/) 

With a paid subscription there is no query limit, and access to HBL (the new zone that deals with cryptovalues, emails and malware) is included. 

All the technical information about HBL is available [here](https://docs.spamhaustech.com/10-data-type-documentation/datasets/030-datasets.html#hbl)

If you have a free DQS subscription and would like to trial HBL, please send an email to [sales@spamhaus.com](mailto:sales@spamhaus.com) including your customer ID, and you will be contacted by one of our representative to activate a 30 day trial.

***

#### How do I register a DQS key?

Just go [here](https://www.spamhaus.com/dqs/) and complete the registration procedure. After you register an account, go to [this](https://portal.spamhaus.com/manuals/dqs/) page and you'll find the DQS key under section "1.0 Datafeed Query Service".

***

#### Prerequisites

You need a DQS key along with an existing Rspamd 1.9.1+ (old rules, unsupported), Rspamd 2.x (old rules, unsupported) or Rspamd 3.x  (currently supported) installation on your system. These instructions do not cover the initial Rspamd installation. To correctly install Rspamd, please refer to instructions applicable to your distribution or see the documentation on the [Rspamd site](https://rspamd.com/).

***

#### Conventions

We are going to use some abbreviations and placeholders:

 * SH: Spamhaus
 * *configuration directory*: whenever you'll find these italic words, we will refer to Rspamd's configuration directory. Depending on your distribution it may be `/etc/rspamd` or other
 * Whenever you find the box below, it means that you need to enter the command on your shell:
```
	$ command
```
 * Whenever you find the box below, it means that you need to enter the command on a shell with *root privileges*:
```
	# command
```

***

#### Warning! Warning! Understand what follows!

Since Rspamd is a fast changing and evolving project, we are going to support *only* the latest version, that is 3.x at the time of writing.

However you'll also find two directories, 1.9 and 2.x that contain the rules for those respective Rspamd versions. Those rules are there only for historic purposes and will not be supported. The installation instructions below may not work for older versions. Specifically - you need to put your DQS key in the `rspamd.local.lua` file for version `2.x`, not just in `*.conf` files.

The HBL subset has only been tested on versions 2.4 and higher, so you are strongly encouraged to run the latest Rspamd version before using these rules.

The HBL URL component has only been tested on 3.6 and higher, and we know it *does not work* on version 3.5 and below, so upgrade to the latest version to get access to all the latest features!

***

## Installation instructions

First of all, please note that we consider these configuration files to be *beta*. We did some limited field tests, but you are encouraged to keep an eye on the logfiles to spot any possible problems we may have missed. See the [support and feedback](#support-and-feedback) section below for how to reach us in case of need.

Start with downloading all the needed files:

```
	$ git clone https://github.com/spamhaus/rspamd-dqs
	Cloning into 'rspamd-dqs'...
	remote: Enumerating objects: 10, done.
	remote: Counting objects: 100% (10/10), done.
	remote: Compressing objects: 100% (8/8), done.
	remote: Total 10 (delta 0), reused 10 (delta 0), pack-reused 0
	Unpacking objects: 100% (10/10), done.
```

A subdirectory called `rspamd-dqs` will be created. Within it you will find the following files:

- `README.md`. This is just a pointer to this document.
- `Changelog.md`. The changes log file.
- `hbltest.sh`. A script that helps you know if your DQS key is HBL enabled.
- `1.9`. Directory that contains config files for Rspamd 1.9.1+ (unsupported)
- `1.9/rbl.conf`. This file contains lookup redefinitions for the IP-based lists.
- `1.9/surbl.conf`. This file contains lookup redefinitions for the domain-based lists.
- `1.9/emails.conf`. This file contains lookup redefinitions for email addresses.
- `1.9/rbl_group.conf`. This file contains scores redefinitions.
- `2.x`. Directory that contains config files for Rspamd 2.x (unsupported)
- `2.x/rbl.conf`. This file contains lookup redefinitions and more for all SH lists.
- `2.x/rbl_group.conf`. This file contains scores redefinitions.
- `2.x/rspamd.local.lua`. This file contains functions used only with HBL.
- `2.x/sh_rbl_group_hbl.conf`. This file contains scores for HBL.
- `2.x/sh_rbl_hbl.conf`. This file contains definitions for HBL.
- `3.x`. Directory that contains config files for Rspamd 3.x
- `3.x/rbl.conf`. This file contains lookup redefinitions and more for all SH lists.
- `3.x/rbl_group.conf`. This file contains scores redefinitions.
- `3.x/rspamd.local.lua`. This file contains functions used only with HBL.
- `3.x/sh_rbl_group_hbl.conf`. This file contains scores for HBL.
- `3.x/sh_rbl_hbl.conf`. This file contains definitions for HBL.
- `3.x/settings.conf`. This file contains settings for the HBL URL and CW matches.

In these installation instructions, we are assuming you are running Rspamd version 3.x. You install this module by using these commands:

```
	$ cd rspamd-dqs/3.x
```

Next, configure your DQS key. Assuming your key is `aip7yig6sahg6ehsohn5shco3z`, execute the following command:

```
	$ sed -i -e 's/your_DQS_key/aip7yig6sahg6ehsohn5shco3z/g' *.conf
```

If you are on FreeBSD then the command slightly changes:

```
	$ sed -i "" -e 's/your_DQS_key/aip7yig6sahg6ehsohn5shco3z/g' *.conf
```

There will be no output, but the key will be inserted in all the needed places. 

We provide a simple script to help you verify whether your DQS key is HBL enabled or not. Use this script to understand what files to copy in your Rspamd config directory. You only need to run the script and input your DQS key.

Assuming the example key ```aip7yig6sahg6ehsohn5shco3z``` *is* HBL enabled, run the script and the output will confirm whether your key is HBL enabled:

```
	$ sh ../hbltest.sh 
	Please input your DQS key: aip7yig6sahg6ehsohn5shco3z
	Looking up test record for HBL... done
	Your DQS key aip7yig6sahg6ehsohn5shco3z is enabled for HBL
	You can copy sh_rbl_hbl.conf, sh_rbl_group_hbl.conf, settings.conf and rspamd.local.lua if you
	want HBL enabled detection.

```

If your key is not HBL enabled (meaning that you registered a FREE DQS key and did not use a paid subscription) the output will be the following:

```
	$ sh ../hbltest.sh 
	Please input your DQS key: aip7yig6sahg6ehsohn5shco3z
	Looking up test record for HBL... done
	Your DQS key aip7yig6sahg6ehsohn5shco3z is -=NOT=- enabled for HBL
	Please *do not* copy sh_rbl_hbl.cf, sh_rbl_group_hbl.cf and rspamd.local.lua
```

Next, move the configuration files in your Rspamd *configuration directory*. 

If your DQS key is HBL enabled and, assuming your configuration directory is `/etc/rspamd` , execute the following command:

```
	# umask 022
	# cp -i *.conf /etc/rspamd/local.d
	# cp rspamd.local.lua /etc/rspamd
```

If the `cp` command tells you that you are overwriting files, you might have an older installation of this module. You can overwrite the `sh_rbl*` and `rbl*` files. However, if you already have a `settings.conf`, make sure to merge the spamhaus settings with any existing settings you may have in that file. Simply appending the spamhaus `settings.conf` to the existing one will usually do just fine.

Or, if your DQS key is not HBL enabled, use this:

```
	# umask 022
	# cp rbl.conf rbl_group.conf /etc/rspamd/local.d
```

It is not recommended to copy the HBL files if your key is not HBL enabled, as the query timeouts will likely increase the time Rspamd uses to scan an email.

Next, run:

```
	# rspamadm configtest
```

If the output ends with the line:

```
	syntax OK
```

then you are done! Just restart Rspamd and you'll have the updated configuration up and running. The HBL `_url` component needs the `URL_normalization.yaml` file to properly operate. That file is automatically downloaded and kept up to date in your `$DBDIR` (usually `/var/lib/rspamd`). The file is checked periodically by the plugin itself, but you can turn this off by setting `download_check` to `0` in `settings.conf`.
By default the file is downloaded from spamhaus itself, from https://docs.spamhaus.com/download/URL_normalization.yaml. If you have an outgoing firewall that may block downloads, make sure to allow downloads from that location. You can also specify an alternative download location by setting `download_url` in the `settings.conf` file. Or download the file in some other way, install it in your `$DBDIR` yourself, and set `download_check` to `0` to disable automatic downloads.

The plugin only checks upto 100 URLs per email. This can be configured in `settings.conf`.

## Testing your setup

Once you succesfully installed the plugin, you could head to [https://blt.spamhaus.com](https://blt.spamhaus.com) and test if you have correctly installed everything.

**Please read the docs carefully**, as a "delivered" response with a red flag **doesn't always mean you missed something**; it depends on your setup. You should always check all the headers of any email that the BLT sends and look for spam headers, usually, but not always: "X-Spam-Flag: Yes" or "X-Spam: Yes".

## Final recommendations
 
The configuration in the VBSpam survey makes exclusive use of our data, since our goal was to certify their quality, and to keep an eye on how we perform in the field.

While the results are reasonably good, the malware/phishing scoring can certainly be improved by employing some additional actions that we recommend.

- Install an antivirus software on your mailserver;
- The modern rule of thumb for receiving email should be to "stay defensive", which is why we recommend doing basic attachment filtering by dropping all emails that contains potentially hazardous attachments, at *minimum* all file extensions that match this regex:

```
(exe|vbs|pif|scr|bat|cmd|com|cpl|dll|cpgz|chm|js|jar|wsf)
```

- You should also drop, by default, all Office documents with macros.

## Support and feedback

We would be happy to receive your feedback! If you notice any problems with this installation, please open a Github issue and we'll do our best to help you.

Remember that we are only going to support the latest version, so before opening an issue, please be sure to be running the up to date code from this Github repository.

