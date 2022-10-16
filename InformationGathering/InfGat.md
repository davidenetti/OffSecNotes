The information gathering phase is the first step in every penetration test where we need to simulate external attackers without internal information from the target organization. This phase is crucial as poor and rushed information gathering could result in missing flaws that otherwise thorough enumeration would have uncovered.

During this process, our objective is to identify as much information as we can from the following areas:

- **Domains and Subdomains**: Often, we are given a single domain or perhaps a list of domains and subdomains that belong to an organization. Many organizations do not have an accurate asset inventory and may have forgotten both domains and subdomains exposed externally. This is an essential part of the reconnaissance phase. We may come across various subdomains that map back to in-scope IP addresses, increasing the overall attack surface of our engagement (or bug bounty program). Hidden and forgotten subdomains may have old/vulnerable versions of applications or dev versions with additional functionality (a Python debugging console, for example). Bug bounty programs will often set the scope as something such as *.inlanefreight.com, meaning that all subdomains of inlanefreight.com, in this example, are in-scope (i.e., acme.inlanefreight.com, admin.inlanefreight.com, and so forth and so on). We may also discover subdomains of subdomains. For example, let's assume we discover something along the lines of admin.inlanefreight.com. We could then run further subdomain enumeration against this subdomain and perhaps find dev.admin.inlanefreight.com as a very enticing target. There are many ways to find subdomains (both passively and actively) which we will cover later in this module;
- **IP ranges**: Unless we are constrained to a very specific scope, we want to find out as much about our target as possible. Finding additional IP ranges owned by our target may lead to discovering other domains and subdomains and open up our possible attack surface even wider;
- **Infrastructure**: We want to learn as much about our target as possible. We need to know what technology stacks our target is using. Are their applications all ASP.NET? Do they use Django, PHP, Flask, etc.? What type(s) of APIs/web services are in use? Are they using Content Management Systems (CMS) such as WordPress, Joomla, Drupal, or DotNetNuke, which have their own types of vulnerabilities and misconfigurations that we may encounter? We also care about the web servers in use, such as IIS, Nginx, Apache, and the version numbers. If our target is running outdated frameworks or web servers, we want to dig deeper into the associated web applications. We are also interested in the types of back-end databases in use (MSSQL, MySQL, PostgreSQL, SQLite, Oracle, etc.) as this will give us an indication of the types of attacks we may be able to perform;
- **Virtual Hosts**: Lastly, we want to enumerate virtual hosts (vhosts), which are similar to subdomains but indicate that an organization is hosting multiple applications on the same web server. We will cover vhost enumeration later in the module as well.


We can break the information gathering process into two main categories:
- **Passive information gathering**: We do not interact directly with the target at this stage. Instead, we collect publicly available information using search engines, whois, certificate information, etc. The goal is to obtain as much information as possible to use as inputs to the active information gathering phase;
- **Active information gathering**: We directly interact with the target at this stage. Before performing active information gathering, we need to ensure we have the required authorization to test. Otherwise, we will likely be engaging in illegal activities. Some of the techniques used in the active information gathering stage include port scanning, DNS enumeration, directory brute-forcing, virtual host enumeration, and web application crawling/spidering.

It is crucial to keep the information that we collect well-organized as we will need various pieces of data as inputs for later phasing of the testing process. Depending on the type of assessment we are performing, we may need to include some of this enumeration data in our final report deliverable (such as an External Penetration Test). When writing up a bug bounty report, we will only need to include details relevant specifically to the bug we are reporting (i.e., a hidden subdomain that we discovered led to the disclosure of another subdomain that we leveraged to obtain remote code execution (RCE) against our target.


# WHOIS 

We can consider WHOIS as the "white pages" for domain names. It is a TCP-based transaction-oriented query/response protocol listening on TCP port 43 by default. We can use it for querying databases containing domain names, IP addresses, or autonomous systems and provide information services to Internet users. The protocol is defined in RFC 3912. The first WHOIS directory was created in the early 1970s by Elizabeth Feinler and her team working out of Stanford University's Network Information Center (NIC). Together with her team, they created domains divided into categories based upon a computer's physical address.

The WHOIS domain lookups allow us to retrieve information about the domain name of an already registered domain. The Internet Corporation of Assigned Names and Numbers (ICANN) requires that accredited registrars enter the holder's contact information, the domain's creation, and expiration dates, and other information in the Whois database immediately after registering a domain. In simple terms, the Whois database is a searchable list of all domains currently registered worldwide.

# DNS

The Domain Name System (DNS) is an excellent place to look for this kind of information. But first, let us take a look at what DNS is.

The DNS is the Internet's phone book. Domain names such as hackthebox.com and inlanefreight.com allow people to access content on the Internet. Internet Protocol (IP) addresses are used to communicate between web browsers. DNS converts domain names to IP addresses, allowing browsers to access resources on the Internet.

Each Internet-connected device has a unique IP address that other machines use to locate it. DNS servers minimize the need for people to learn IP addresses like 104.17.42.72 in IPv4 or more sophisticated modern alphanumeric IP addresses like 2606:4700::6811:2b48 in IPv6. When a user types www.facebook.com into their web browser, a translation must occur between what the user types and the IP address required to reach the www.facebook.com webpage.

There is a hierarchy of names in the DNS structure. The system's root, or highest level, is unnamed.

TLDs nameservers, the Top-Level Domains, might be compared to a single shelf of books in a library. The last portion of a hostname is hosted by this nameserver, which is the following stage in the search for a specific IP address (in www.facebook.com, the TLD server is com). Most TLDs have been delegated to individual country managers, who are issued codes from the ISO-3166-1 table. These are known as country-code Top-Level Domains or ccTLDs managed by a United Nations agency.

There are also a small number of "generic" Top Level Domains (gTLDs) that are not associated with a specific country or region. TLD managers have been granted responsibility for procedures and policies for the assignment of Second Level Domain Names (SLDs) and lower level hierarchies of names, according to the policy advice specified in ISO-3166-1.

A manager for each nation organizes country code domains. These managers provide a public service on behalf of the Internet community. Resource Records are the results of DNS queries and have the following structure:

- **Resource record**: A domain name, usually a fully qualified domain name, is the first part of a Resource Record. If you don't use a fully qualified domain name, the zone's name where the record is located will be appended to the end of the name;
- **TTL**: In seconds, the Time-To-Live (TTL) defaults to the minimum value specified in the SOA record;
- **Record class**: Internet, Hesiod, or Chaos;
- **Start of authority (SOA)**: It should be first in a zone file because it indicates the start of a zone. Each zone can only have one SOA record, and additionally, it contains the zone's values, such as a serial number and multiple expiration timeouts;
- **Home server (NS)**: The distributed database is bound together by NS Records. They are in charge of a zone's authoritative name server and the authority for a child zone to a name server;
- **IPv4 addresses (A)**: The A record is only a mapping between a hostname and an IP address. 'Forward' zones are those with A records;
- **Pointer (PTR)**: The PTR record is a mapping between an IP address and a hostname. 'Reverse' zones are those that have PTR records;
- **Canonical Name (CNAME)**: An alias hostname is mapped to an A record hostname using the CNAME record;
- **Mail Exchange (MX)**: The MX record identifies a host that will accept emails for a specific host. A priority value has been assigned to the specified host. Multiple MX records can exist on the same host, and a prioritized list is made consisting of the records for a specific host.


# Passive subdomain enumeration

Subdomain enumeration refers to mapping all available subdomains within a domain name. It increases our attack surface and may uncover hidden management backend panels or intranet web applications that network administrators expected to keep hidden using the "security by obscurity" strategy. At this point, we will only perform passive subdomain enumeration using third-party services or publicly available information. Still, we will expand the information we gather in future active subdomain enumeration activities.


# Passive infrastructure identification

Netcraft can offer us information about the servers without even interacting with them, and this is something valuable from a passive information gathering point of view. We can use the service by visiting https://sitereport.netcraft.com and entering the target domain.

Some interesting details we can observe from the report are:

- Background: General information about the domain, including the date it was first seen by Netcraft crawlers;
- Network: Information about the netblock owner, hosting company, nameservers, etc;
- Hosting history: Latest IPs used, webserver, and target OS.

We need to pay special attention to the latest IPs used. Sometimes we can spot the actual IP address from the webserver before it was placed behind a load balancer, web application firewall, or IDS, allowing us to connect directly to it if the configuration allows it. This kind of technology could interfere with or alter our future testing activities.


# Wayback Machine

The Internet Archive is an American digital library that provides free public access to digitalized materials, including websites, collected automatically via its web crawlers.

We can access several versions of these websites using the Wayback Machine to find old versions that may have interesting comments in the source code or files that should not be there. This tool can be used to find older versions of a website at a point in time. Let's take a website running WordPress, for example. We may not find anything interesting while assessing it using manual methods and automated tools, so we search for it using Wayback Machine and find a version that utilizes a specific (now vulnerable) plugin. Heading back to the current version of the site, we find that the plugin was not removed properly and can still be accessed via the wp-content directory. We can then utilize it to gain remote code execution on the host and a nice bounty.

We can check one of the first versions of facebook.com captured on December 1, 2005, which is interesting, perhaps gives us a sense of nostalgia but is also extremely useful for us as security researchers.

We can also use the tool waybackurls to inspect URLs saved by Wayback Machine and look for specific keywords. Provided we have Go set up correctly on our host, we can install the tool as follows:

    go get github.com/tomnomnom/waybackurls

To get a list of crawled URLs from a domain with the date it was obtained, we can add the -dates switch to our command as follows:

    waybackurls -dates https://facebook.com > waybackurls.txt
    cat waybackurls.txt

# Active infrastructure identification

A web application's infrastructure is what keeps it running and allows it to function. Web servers are directly involved in any web application's operation. Some of the most popular are Apache, Nginx, and Microsoft IIS, among others.

If we discover the webserver behind the target application, it can give us a good idea of what operating system is running on the back-end server. For example, if we find out the IIS version running, we can infer the Windows OS version in use by mapping the IIS version back to the Windows version that it comes installed on by default. Some default installations are:

- IIS 6.0: Windows Server 2003
- IIS 7.0-8.5: Windows Server 2008 / Windows Server 2008R2
- IIS 10.0 (v1607-v1709): Windows Server 2016
- IIS 10.0 (v1809-): Windows Server 2019

Although this is usually correct when dealing with Windows, we can not be sure in the case of Linux or BSD-based distributions as they can run different web server versions in the case of Nginx or Apache. This kind of technology could interfere with or alter our future testing activities.

## Web servers

We need to discover as much information as possible from the webserver to understand its functionality, which can affect future testing. For example, URL rewriting functionality, load balancing, script engines used on the server, or an Intrusion detection system (IDS) in place may impede some of our testing activities.

The first thing we can do to identify the webserver version is to look at the response headers.

    curl -I "http://${TARGET}"

There are also other characteristics to take into account while fingerprinting web servers in the response headers. These are:

- X-Powered-By header: This header can tell us what the web app is using. We can see values like PHP, ASP.NET, JSP, etc.
- Cookies: Cookies are another attractive value to look at as each technology by default has its cookies. Some of the default cookie values are:
    1. .NET: ASPSESSIONID<RANDOM>=<COOKIE_VALUE>
    2. PHP: PHPSESSID=<COOKIE_VALUE>
    3. JAVA: JSESSION=<COOKIE_VALUE>

Other available tools analyze common web server characteristics by probing them and comparing their responses with a database of signatures to guess information like web server version, installed modules, and enabled services. Some of these tools are:

1. WhatWeb
2. WafW00f
3. aquatone

# VHosts
    
A virtual host (vHost) is a feature that allows several websites to be hosted on a single server. This is an excellent solution if you have many websites and don't want to go through the time-consuming (and expensive) process of setting up a new web server for each one. Imagine having to set up a different webserver for a mobile and desktop version of the same page. There are two ways to configure virtual hosts:

- IP-based virtual hosting
- Name-based virtual hosting
    
## IP-based Virtual Hosting
For this type, a host can have multiple network interfaces. Multiple IP addresses, or interface aliases, can be configured on each network interface of a host. The servers or virtual servers running on the host can bind to one or more IP addresses. This means that different servers can be addressed under different IP addresses on this host. From the client's point of view, the servers are independent of each other.
    
## Name-based Virtual Hosting

The distinction for which domain the service was requested is made at the application level. For example, several domain names, such as admin.inlanefreight.htb and backup.inlanefreight.htb, can refer to the same IP. Internally on the server, these are separated and distinguished using different folders. Using this example, on a Linux server, the vHost admin.inlanefreight.htb could point to the folder /var/www/admin. For backup.inlanefreight.htb the folder name would then be adapted and could look something like /var/www/backup.

During our subdomain discovering activities, we have seen some subdomains having the same IP address that can either be virtual hosts or, in some cases, different servers sitting behind a proxy.
    
# Crawling
Crawling a website is the systematic or automatic process of exploring a website to list all of the resources encountered along the way. It shows us the structure of the website we are auditing and an overview of the attack surface we will be testing in the future. We use the crawling process to find as many pages and subdirectories belonging to a website as possible.

Zed Attack Proxy (ZAP) is an open-source web proxy that belongs to the Open Web Application Security Project (OWASP). It allows us to perform manual and automated security testing on web applications. Using it as a proxy server will enable us to intercept and manipulate all the traffic that passes through it.

## Sensitive informations disclosure
It is typical for the webserver and the web application to handle the files it needs to function. However, it is common to find backup or unreferenced files that can have important information or credentials. Backup or unreferenced files can be generated by creating snapshots, different versions of a file, or from a text editor without the web developer's knowledge. There are some lists of common extensions we can find in the raft-[ small | medium | large ]-extensions.txt files from SecLists.

We will combine some of the folders we have found before, a list of common extensions, and some words extracted from the website to see if we can find something that should not be there. The first step will be to create a file with the following folder names and save it as folders.txt.
