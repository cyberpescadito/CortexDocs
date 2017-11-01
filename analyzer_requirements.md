# DRAFT -- WIP
Please do not make corrections or submit PRs at this time as the document is 
undergoing changes.

# Cortex Analyzer Requirements
This document outlines the different information that needs to be provided for 
each analyzer, such as API keys, usernames, instances, etc. It will also 
specify whether the service is paid, free, or requires special access. This 
will assist users in deciding what analyzers they want to enable and what 
information they need to gather in order to enable them.

## Table of Contents

  * [Introduction](#introduction)
  * [Free Analyzers](#free-analyzers)
    * [Abuse\_Finder](#abuse_finder)
    * [CuckooSandbox](#cuckoosandbox)
    * [File\_Info](#file_info)
    * [FireHOLBlocklists](#fireholblocklists)
    * [Fortiguard](#fortiguard)
    * [GoogleSafeBrowsing](#googlesafebrowsing)
    * [Hippocampe](#hippocampe)
    * [MaxMind](#maxmind)
    * [MISP](#misp)
    * [Msg\_Parser](#msg_parser)
    * [OTXQuery](#otxquery)
    * [PhishTank](#phishtank)
    * [PhishingInitiative](#phishinginitiative)
    * [Virusshare](#virusshare)
    * [WOT](#wot)
    * [Yara](#yara)
    * [Yeti](#yeti)
  * [Paid](#paid)
    * [JoeSandbox](#joesandbox)
    * [PassiveTotal](#passivetotal)
    * [DNSDB](#dnsdb)
    * [DomainTools](#domaintools)
    * [Nessus](#nessus)
    * [VMRay](#vmray)
    * [VirusTotal](#virustotal)
  * [Special Access](#special-access)
    * [CERTatPassiveDNS](#certatpassivedns)
    * [CIRCLPassiveDNS](#circlpassivedns)
    * [CIRCLPassiveSSL](#circlpassivessl)

## Introduction
All configuration settings must be made in the global Cortex 
configuration 
file (`/etc/cortex/application.conf` by default), in the `config` section.

By default, all analyzers are enabled. If you want to disable some of them, 
add a 
`disabled` list to `/etc/cortex.application.conf` with the full name of the 
analyzer, including its version. For example, if you'd like to disable the 
Abuse Finder (version 2.0 as of this writing) and DNSDB DomainName (same 
version as of this writing), add the following line in the `analyzer` section:
```text
analyzer {
  # Absolute path where you have pulled the Cortex-Analyzers repository.
  path = "/opt/Cortex-Analyzers/analyzers"

  disabled = ["Abuse_Finder_2_0", "DNSDB_DomainName_2_0"]

  # Sane defaults. Do not change unless you know what you are doing.
  fork-join-executor {
  [...]
  }
  [...]
}
```

If you have troubles finding out the exact name of the analyzer you'd like to
 disable, you can use a command like the following to list the full names of 
 all the analyzers:

```commandline
$ grep "Register analyzer" /var/log/cortex/application.log \
| sed -e 's/.*(\([0-9a-zA-Z_]\+\))$/\1/' | sort -u
```

The output should look like the following:
```commandline
Abuse_Finder_2_0
CERTatPassiveDNS_2_0
CIRCLPassiveDNS_2_0
CIRCLPassiveSSL_2_0
CuckooSandbox_File_Analysis_Inet_1_0
CuckooSandbox_Url_Analysis_1_0
DNSDB_DomainName_2_0
DNSDB_IPHistory_2_0
DNSDB_NameHistory_2_0
DomainTools_ReverseIP_2_0
DomainTools_ReverseNameServer_2_0
DomainTools_ReverseWhois_2_0
DomainTools_WhoisHistory_2_0
[...]
```
## Free Analyzers

### Abuse_Finder
Use CERT-SG's [Abuse Finder](https://github.com/certsocietegenerale/abuse_finder) to fin abuse contacts associated with domain 
names, 
URLs, IPs and email addresses.

The analyzer comes in only one flavor.

The analyzer has no entry in the `config` section. It can be used out 
of the box.

### CuckooSandbox
Analyze URLs and files using [Cuckoo Sandbox](https://cuckoosandbox.org/).

The analyzer comes in two flavors:

- CuckooSandbox_**File_Analysis_Inet**: submit files for analysis to a Cuckoo 
Sandbox instance with Internet access.
- CuckooSandbox_**Url_Analysis**: submit URLs for analysis to a Cuckoo Sandbox 
instance.

#### Requirements
The CuckooSandbox analyzer requires you to have a local instance
 of Cuckoo Sandbox deployed.
It is an open source tool that is free for use but needs to be manually 
deployed in your environment. Please go to 
[https://cuckoosandbox.org/](https://cuckoosandbox.org/) 
for more information on setting it up.

To configure the analyzer you need to supply the URL of your local instance 
using the `url` keyword.

#### Example:
```text
CuckooSandbox {
    url = "http://my.cuckoo.sandbox"
}
```

### File_Info
Parse files in several formats such as OLE and OpenXML to detect VBA macros, 
extract their source code, generate useful information on PE, PDF files and much more.

The analyzer comes in only one flavor.

No configuration is required. The analyzer has no entry in the `config` section. It can be used out 
of the box.

### FireHOLBlocklists 
Check IP addresses against the [FireHOL blocklists](https://firehol.org/).

The analyzer comes in only one flavor.

#### Requirements
This analyzer needs you to download the FireHOL block lists first to a 
directory. Use `git` for that  purpose:
```commandline
$ mkdir /path/to/firehol
$ cd /path/to/firehol
$ git clone https://github.com/firehol/blocklist-ipsets 
```

We advise you to keep the lists fresh  by  adding  a  cron  entry  to 
regularly download them for example (using `git pull`).

Specify the directory where the lists have been downloaded using the 
`blocklistpath` paramater and  an optional `ignoreolderthandays` parameter to
 ignore all lists that have not been updated in the last N days.

#### Example
```text
    FireHOLBlocklists {
      blocklistpath = "/opt/firehol/blocklists"
      ignoreolderthandays="7" # ignore all lists not updated in the last 7d
    }
```

### Fortiguard
Check the [Fortiguard](https://fortiguard.com/webfilter) category of a URL or
 a domain.

The analyzer comes in only one flavor called *Fortiguard_URLCategory*.

No configuration is required. The analyzer has no entry in the `config` section. It can be used out 
of the box.

### GoogleSafeBrowsing
Check URLs against [Google Safebrowsing](https://www.google.com/transparencyreport/safebrowsing/).

The analyzer comes in only one flavor. 

#### Requirements
You need to [obtain an API key](https://developers.google.com/safe-browsing/)
 from Google.

Provide your API key as a value of the `key` parameter.

#### Example
```text
    GoogleSafebrowsing {
      key = "MYKEY"
    }
```

### Hippocampe
Query threat feeds through [Hippocampe](https://github.com/CERT-BDF/Hippocampe), 
a FOSS tool from TheHive Project that centralizes feeds and allows you to 
associate a confidence level to each one of them (that can be changed over time)
 and get a score indicating the data quality.
 
The analyzer comes in two flavors:
- HippoMore: get the Hippocampe detailed report for an IP address, a domain or
 a URL.
- Hipposcore: get the Hippocampe Score report associated with an IP address, a
 domain or a URL.

#### Requirements
The Hippocampe analyzer requires you to have a local instance of Hippocampe 
deployed/configured. It is an open source tool that is free for use but needs
 to be manually deployed in your environment. Please go to [https://github.com/CERT-BDF/Hippocampe](https://github.com/CERT-BDF/Hippocampe)
 for more in information on setting it up.

To configure the analyzer you need to supply the URL of your local instance 
using the `url` keyword.

#### Example
```text
Hippocampe {
    url = "http://my.hippocampe.instance"
}
```

### MaxMind
Geolocate an IP Address via [MaxMind](https://www.maxmind.com/en/home) 
GeoLite2 **free** City and Country databases.

Cortex does not refresh those databases automatically. It is up to you to 
create a cron job to refresh them at the frequency you want. The files to 
update are:

- `MaxMind/GeoLite2-City.mmdb`
- `MaxMind/GeoLite2-Country.mmdb`

You can fetch up-to-date versions from [https://dev.maxmind.com/geoip/geoip2/geolite2/](https://dev.maxmind.com/geoip/geoip2/geolite2/).

The analyzer comes in only one flavor.

No configuration is required. The analyzer has no entry in the `config` section. It can be used out 
of the box.

### MISP
Query multiple MISP (Malware Information Sharing Platform )instances for 
events containing an observable.

[MISP](http://www.misp-project.org/) is an open source threat sharing 
platform and considered 
the *de facto* standard in the field. You'd benefit greatly from using it in 
conjunction to Cortex and TheHive as these 3 FOSS products make an 
interesting Threat Intelligence, Incident Response and Digital Forensics 
ecosystem.

The analyzer comes in only one flavor. 

#### Requirements
The MISP analyzer requires you to have access to one or several [MISP](http://www.misp-project.org/) 
 instances. You can also deploy your own instance.

Four parameters are required:
- `url`
- `key`
- `certpath`
- `name`

You need the URL for each MISP instance you'd like to search. Those URLs go
in the `url` dict. You'll also need the authentication key associated with 
your account on each of those instances. To obtain the key, log into the MISP
 instance's Web UI, click on your username on the top navigation bar and 
 retrieve the value of the `Authkey` parameter. Each `Authkey` must be added,
  in the same order as the URLs to the `key` dict. 

Another important parameter is the `certpath` dict. For each MISP instance:

- Use `""` if you don't want to validate the instance's X.509 certificate or 
if the instance use old plain HTTP.
- Use `"/etc/ssl/certs"` or another file to validate the instance's X.509 
certificate.

Last but not least, give each instance a name and add it in the order you 
specified URLs and keys above to the `name` dict.

#### Example
The example below shows the configuration of the MISP analyzer which will 
search two MISP instances called MY-OWN-MISP and REMOTE-MISP. Note that the 
first one is accessed through HTTP while the second has HTTPS enabled.

```text
    MISP {
      url=["http://my.own.misp", "https://remote-misp.peercert.org"]
      key=["my-own-misp-account-authkey", "remote-misp-account-authkey"]
      certpath=["","/etc/ssl/certs"]
      name=["MY-OWN-MISP","REMOTE-MISP"]
```

### Msg_Parser
Parse Outlook message files automatically and show the key information it 
contains such as headers, attachments etc. Please note that the analyzer 
doesn't extract attachments.

The analyzer comes in only one flavor.

No configuration is required. The analyzer has no entry in the `config` section. It can be used out 
of the box.

### OTXQuery
Query AlienVault's [Open Threat Exchange](https://otx.alienvault.com/) for IPs, 
domains, URLs, or file hashes.

The analyzer comes in only one flavor.

#### Requirements
You need to sign up for an [OTX](https://otx.alienvault.com/) account or use 
an existing one.

Log in to your OTX account, click on your username on the top 
navigation bar then on *Settings* and retrieve your OTX key and use it as the 
value of the `key` parameter.

#### Example
```text
    OTXQuery {
      key="MYUBERSEKRETOTXQUERYKEY"
    }
```

### PhishTank
Query [PhishTank](https://www.phishtank.com/) to assess whether a URL has 
been flagged a phishing site.

The analyzer comes in only one flavor called *PhishTank_CheckURL*.

#### Requirements
You need to sign up for a [PhishTank](https://www.phishtank.com/register.php)
 account or use an existing one.

Log in to your PhishTank account, click on the *Developers* tab then on 
*Manage Applications*, register an application by giving it a name and 
entering a CAPTCHA code. You'll obtain an API key that you'll need to supply 
as the value to the `key` configuration parameter for this analyzer to work.

#### Example
```text
    PhishTank {
      key="MYPHISHTANKAPIKEYGOESHERE"
    }
```

### PhishingInitiative
Query [Phishing Initiative](https://phishing-initiative.fr/contrib/) to 
assess whether a URL has been flagged a phishing site.

This analyzer comes in only one flavor called *PhishingInitiative_Lookup*.

#### Requirements
You need to sign up for a [Phishing Initiative](https://phishing-initiative.fr/register)
 account or use an existing one.

Log in to your Phishing Initiative account, click on the icon representing 
your account details then on *API*. Retrieve the API key value and supply 
it as the value to the `key` configuration parameter.

#### Example
```text
    PhishingInitiative {
      key="MYPHISHINGINITIATIVEAPIKEYGOESHERE"
    }
```

### Virusshare
Check whether a file or hash is available on [VirusShare.com](https://virusshare.com/).

This analyzer comes in only one flavor.

#### Requirements
Prior to using the analyzer, you need to retrieve the Virusshare hash lists 
using the `download_hashes.py` script that is located in the same directory 
as the analyzer. To keep your lists fresh, you may want to regularly  
download them using a cron entry or a similar system.

Indicate the path where you have downloaded the hash lists using the `path` 
parameter.

#### Example
```text
    Virusshare {
      path = "/path/to/virusshare/lists"
    }
```

### WOT
Check a domain against [Web of Trust](https://www.mywot.com/), a website 
reputation service.

This analyzer comes in only one flavor called *WOT_Lookup*.

#### Requirements
An account with Web of Trust is required to get an API key, which is 
necessary to configure the analyzer. You can sign up for an account at
[https://www.mywot.com/en/signup?destination=profile/api](https://www.mywot.com/en/signup?destination=profile/api).

Supply the API key you'll find under [https://www.mywot.com/en/signup?destination=profile/api](https://www.mywot.com/en/signup?destination=profile/api)
 as the value for the `key` parameter 

#### Example
```text
    WOT {
      key="myWOTAPIkey"
    }
```

### Yara
Check files against [YARA](https://virustotal.github.io/yara/) rules using 
[yara-python](https://github.com/VirusTotal/yara-python).

The analyzer comes in only one flavor.

#### Requirements
You need to point your analyzer to multiple files and/or directories 
containing your YARA rules. If you supply a directory, the analyzer expects to
find an *index.yar* or *index.yas* file. The index file can include other rule 
files. An example can be found in the [Yara-rules](https://github.com/Yara-Rules/rules/blob/master/index.yar)
repository.

Add each file and/or directory containing YARA rules to the `rules` dict. 

#### Example
In the example shown below, the first two locations are directories. As such,
 the analyzer will expect to find an *index.yar* or *index.yas* file in these
  directories:

```text
Yara {
    rules=["/path/dirA", "/path/dirB", "/path/my/rules.yar"]
}
```

### Yeti
[YETI](https://yeti-platform.github.io/) is a FOSS platform meant to organize
 observables, indicators of compromise, TTPs, and knowledge on threats in a 
 single, unified repository. The analyzer for this platform lets you make API 
 calls to YETI and retrieve all available information pertaining to a domain, 
 a fully qualified domain name, an IP address, a URL or a hash.

This analyzer comes in only one flavor.

#### Requirements
The Yeti analyzer requires you to have a local instance of [YETI](https://yeti-platform.github.io/) 
deployed/configured. It is an open source tool that is free for use but needs
 to be manually deployed in your environment.

Provide the URL of your YETI instance as a value for the `url` parameter.

#### Example
```text
Yeti {
    url = "http://my.yeti.instance:5000"
}
```

## Analyzers Requiring Special Access
### CERTatPassiveDNS
Check CERT.at Passive DNS Service for a given domain.

This analyzer comes in only one flavor.

#### Requirements
Access to the CERT.at service is allowed to trusted partners only. If you 
think you qualify, please contact [CERT.at](http://www.cert.at/index_en.html).

No configuration is required. The analyzer has no entry in the `config` section. It can be used out 
of the box if CERT.at positively answers your access request.

### CIRCLPassiveDNS
Check [CIRCL's Passive DNS](https://www.circl.lu/services/passive-dns/) for a
 given domain.

This analyzer comes in only one flavor.
 
#### Requirements
Access to CIRCL Passive DNS is only allowed to trusted partners in Luxembourg
and abroad. [Contact CIRCL](https://www.circl.lu/contact/) if you would like
access. Include your affiliation and the foreseen use of the Passive DNS 
data.

If the CIRCL positively answers your access request, you'll obtain a username
 and password which are needed to make the analyzer work.

supply your username as the value for the `user` parameter and your password 
as the value for the `password` parameter.

#### Example
```text
    CIRCLPassiveDNS {
      user= "myusername"
      password= "mypassword"
    }
```

### CIRCLPassiveSSL
Check [CIRCL's Passive SSL](https://www.circl.lu/services/passive-ssl/) 
service for a given IP address or certificate hash.

This analyzer comes in only one flavor.

#### Requirements
Access to CIRCL Passive SSL is allowed to partners including security 
researchers or incident analysts worldwide. [Contact CIRCL](https://www.circl.lu/contact/)
if you would like access.

If the CIRCL positively answers your access request, you'll obtain a username
 and password which are needed to make the analyzer work.

supply your username as the value for the `user` parameter and your password 
as the value for the `password` parameter.

#### Example
```text
    CIRCLPassiveSSL {
      user= "myusername"
      password= "mypassword"
    }
```

**PROGRESS_MARK**

## Paid  

- DNSDB
- DomainTools
- JoeSandbox
- Nessus
- PassiveTotal
- VirusTotal
- VMRay

### JoeSandbox
**Configuration Parameters**: _TBD_

JoeSandbox has both a free and a paid version. 

### PassiveTotal
**Configuration Parameters**: Username, API Key

An account with PassiveTotal is required to get an API key. You can sign up for an account [here](https://community.riskiq.com/registration).
### DNSDB
**Configuration Parameters**: Server name, API key

This is a paid service that can be purchased from [here](https://www.farsightsecurity.com/order-services/). 

### DomainTools
**Configuration Parameters**: Username, API key

This is a paid service that can be purchased from [here](https://www.domaintools.com/products/api-integration/). There is also a free API that can be used (needs to be set to free in analyzer config) and does not require a username/API key. 


### Nessus
**Configuration Parameters**: Url, login, password, policy

This is a paid service that can be purchased from [here](https://www.tenable.com/products/nessus-vulnerability-scanner). 

### VMRay
**Configuration Parameters**: Url, API key

This is a paid service that can be purchased from [here](https://www.vmray.com/). 

### VirusTotal
**Configuration Parameters**: API Key

An account with VirusTotal is required to get an API key. You can sign up for an account [here](https://www.virustotal.com/en/).