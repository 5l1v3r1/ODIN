# ODIN
### Observe, Detect, and Investigate Networks

[![Python Version](https://img.shields.io/badge/Python-3.6-brightgreen.svg)]() [![license](https://img.shields.io/github/license/mashape/apistatus.svg)]()

![ODIN](https://github.com/chrismaddalena/viper/raw/master/ODIN.png)

`Current version: v2.0.0 "Huginn"`

A Python tool for automating intelligence gathering, asset discovery, and reporting. ODIN is still in active development, so check the dev branch for the bleeding edge. Feedback is welcome!

**Note:** ODIN is designed to be run on Linux. 99% of it will absolutely work on Windows or MacOS with Python 3 and a copy of urlcrazy, but the `extract` tool, used for pulling metadata from non-PDF files (i.e. Office documents), is exclusive to Linux. You'll be fine using an OS without access to `extract`, but you'll see some warnings and get less information. You can always extract the metadata yourself later from the donwloaded files!

You may also want a SQLite3 database viewer/browser (to run your own custom queries against the OSINT database) and Neo4j installed (to view graphs of the external perimeter). Neo4j is *strongly* recommended! The graphs and tables you can create are powerful tools for analysis and presentations. See the NEO4J.md documentation in the docs folder for examples and additional information.

## First Things First
ODIN is made possible through the help, input, and work provided by others. Therefore, this project is entirely open source and available to all to use/modify. Have fun :D

## What Can ODIN Do?
ODIN is still very much in development, but it aims to automate many of the common recon tasks carried out by penetration testers and red teamers.

### Phase 1 - Asset Discovery
* Collect basic organization information from sources like the Full Contact marketing database.
* Check DNS Dumpster, Netcraft, and TLS certificates to discover subdomains for the provided domains.
* Resolve domain and subdomains to IP addresses via socket connections and DNS records.
* Collect information for all IP addresses, such as ownership and organization data, from RDAP, whois, and other data sources.
* Lookup domains and search for IP addresses on Shodan to collect additional data, such as operating systems, service banners, and open ports.
* Check for the possibility of domain takeovers or domain fronting with the domains and subdomains.

### Phase 2 - Employee Discovery
* Harvest email addresses and employee names for the target organization.
* Link employees to social media profiles via search engines and the Twitter API.
* Cross check discovered email addresseswith Have I Been Pwned.

### Phase 3 - Cloud and Web Services
* Hunt for Office files and PDFs under the target domain, download them, and extract metadata.
* Search for AWS S3 buckets and Digital Ocean Spaces using keywords related to the organization.
* Take screenshots of discovered web services for a quick, early review of services.

### Phase 4 - Reporting
* Save all data to a SQLite3 database to allow the data to be easily queried.
* Generate an HTML report using default SQL queries to make it simple to peruse the data in a web browser.
* Create a Neo4j graph database that ties all of the discovered entities (IP addresses, domains, subdomains, ports, and certificates) together with relationships (e.g. RESOLVES_TO, HAS_PORT).

At the end of all of this you will have multiple ways to browse and visualize the data. A simple Neo4j query like `MATCH (n) RETURN n` (display everything) can create a fascinating graph of the organization's external perimeter and make it simple to see how assets are linked.

## Getting Started
### Installing ODIN
ODIN requires **Python 3**. Using `pipenv` for managing the required libraries is the best option to avoid Python installations getting mixed-up. Do this:

1. Run: `pip3 install --user pipenv` or `python3 -m pip install --user pipenv`
2. Run: `git clone https://github.com/chrismaddalena/ODIN.git`
3. Run: `cd ODIN && pipenv install`
4. Start using ODIN by running: `pipenv shell`

**Optional:** Install PhantomJS -- `brew install phantomjs` or `apt install phtanomjs`

This may go away soon due to PhantomJS no longer being actively supported and developed with Selenium.)

**Note 1:** On MacOS you may get an error about `pew` not being in your PATH after installing `pipenv` and attempting to install ODIN. To fix it, follow these steps in order:
* Uninstall virtualenv, pipenv, and pew.
* Install virtualenv
* Install pew
* Install pipenv

**Note 2:** If you're running as root on something like Kali Linux, you'll want to drop the `--user` portion of the `pip` commands above. That seems to create issues for actually using `pipenv` commands later, at least on Kali.

### Setup API Keys
1. Review the keys.config.sample file to fill-in your API keys and create a keys.config file.
2. `cd` into the /setup directory and run `setup_check.py` to make sure your keys.config file is in order.
3. Install awscli, run `aws configure`, and follow the prompts to enter your credentials.

## The APIs and Services
ODIN uses several APIs to gather information. Some of these require an API key, but most of the APIs are free. That is to say, you can get a free key and then pay for more requests/day. Shodan is a good example of this. You may prefer to not use APIs at all for one reason or another. You can still use ODIN, but a few of the APIs are just really fantastic and you should consider using them, specifically Censys and Shodan.

### Whois and RDAP
Both of these services are used to collect data on domains and IP addresses. This includes attaching domains to IP addresses, identifying the network CIDRs for these addresses, and pulling information about the owners.

No API key is needed.

### Robtex
The Robtex free REST API is used to collect domain names tied to IP addresses. This information is displayed alongside the RDAP information for IP addresses, so you can see what else is hosted at that IP address.

No API key is needed.

### Shodan
Shodan is used to search for domains and lookup hosts (IP addresses). This pulls in information like open ports, banners, hostnames, and location data. Shodan also flags hosts for well known vulnerabilities like Heartbleed. This data is recorded as well, if it exists, but does tend to be outdated (or just wrong) a lot of the time.

Sign-up for an account to get your API key: [shodan.io](https://www.shodan.io/)

### Censys
Censys is very much like Shodan, except less information about open ports/services is available. However, Censys provides a way to search for certificates tied to a domain. This can be a *lot* of data, but you may find new hosts, like those tied to an employee's email address and used for a VPS in the cloud.

Sign-up for an account to get your API key: [censys.io](https://www.censys.io/)

### Twitter
If you setup a Twitter app for ODIN, the tokens can be used with Tweepy to collect account data (e.g. real name, location, follower count, and user description) from Twitter profiles ODIN has linked to the target organization.

In the future, this may be used to collect analytics from Twitter to help you find very active users or get a profile of them.

Become a Twitter developer by going to [dev.twitter.com](https://dev.twitter.com/) and then create an app on [apps.twitter.com](https://apps.twitter.com/).

### Cymon
eSentire's Cymon is used to check domains and IP addresses to see if the target appears in any of Cymon's collected threat intelligence feeds. This is used for reputation checks, but also used in combination with urlcrazy to check similar, registered domains to see if the domain or the domain's A-record IP addresses have been reported.

Note that appearing in a threat feed doesn't mean something is wrong or that Cymon has bad data. A domain may have been used for phishing, been detected and seized, and is now dormant with the old malicious A records. Then you have things like cloud service IPs that change hands often. Events like that can lead to a domain or IP being used for malicious activities one day and safe the next. Always investigate these findings before crying wolf to your client.

Sign-up for an account to get your API key: [cymon.io](https://www.cymon.io/)

### URLVoid
URLVoid offers reputation data for domains, including Alexa and Google rankings, domain age, and location data. It also keeps track of domains that have been flagged for malicious activity by various entities (e.g. Fortinet, Avira).

Like Cymon, this may help you identify typosqautted domains (identified via URCrazy) that are/have been linked to malicious activity.

Sign-up for an account to get your API key: [urlvoid.com/api](http://api.urlvoid.com/)

### HaveIBeenPwned
Email addresses are checked against HIBP to determine if any email addresses for the organization have been mentioned in any pastes or been involved in any security breaches.

No API key is needed.

### DNS Dumpster
DNS Dumpster is a cool project you can find at dnsdumpster.com. Subdomain information is collected from DNS Dumpster, including a neat domain map image!

No API key is needed.

### NetCraft
ODIN will check NetCraft for domain history and known subdomains. This does require a web driver for Selenium. If you download a driver and provide the path to it in your keys.config file (Yes, this isn't really a key, but so be it), NetCraft searches will be kicked off automatically when you perform domain OSINT.

The Chrome web driver is recommended, but the Firefox/Gecko driver should work just fine, too.

[Chrome Web Driver](https://github.com/SeleniumHQ/selenium/wiki/ChromeDriver)
[Gecko Driver](https://github.com/mozilla/geckodriver/releases)

### EmailHunter
Meant for marketing folks to find leads and contacts at a company, this service offers free API keys for harvesting their contact information organizaed by company/domain. Hunter will return names, email addresses, phone numbers, Twitter handles, LinkedIn profile links, and job titles.

Sign-up for an account to get your API key: [hunter.io](https://hunter.io/)

### Full Contact
Full Contact support is implemented only for their Company API at the moment, but support for the People API may come in the future. For now, this is used to build a company profile based on a target domain, such as the client's primary domain used for email and their website. Full Contact catalogues everything from website info and company logo(s) to website blurbs and social media profiles.

It's likely Full Contact will get some things wrong, such as number of employees. In my experience, the data is usually not too far off the mark, but the profile is only meant to act as a snapshot to get you started.

Sign-up for an account to get your API key: [app.fullcontact.com](https://app.fullcontact.com/start/welcome)

### AWS
Yes, Amazon Web Services. ODIN will perform recon against AWS to find things like S3 buckets and accounts names and aliases. Account names are strings of numbers, so you will need some idea of what you're looking for there. Aliases, however, can be anything, like a company name, and those can be validated as existing or not.

By default, ODIN uses the client (`-c`) name and domain (`-d`) for searches. ODIN will search for the name with spaces stripped out, the domain with the TLD, and the domain without the TLD. Then ODIN will add some common suffices and prefixes, like "downloads-" or "-apps" to these keywords.

Optional wordlists can be provided for additional keywords and 'fixes. Keywords can be anything, really. Consider assembling a list of related words, alternate client names, etc.

An Amazon and awscli are required.

### Digital Ocean
ODIN will search for Digital Ocean Spaces just like it searches for S3 buckets. Spaces follows the same standards as S3, so it is simple to verify existing Spaces.

No API key is needed.

### WhoXY
WhoXY offers a nice fallback for a regular whois lookup. When whois fails to return any contact information, that can mean domain privacy is enabled or that the registrar is keeping that information behind their own whois tool. Querying individual registrars is not feasible (there's too many and not all make it remotely possible for automated queries), but WhoXY assists by cataloguing a lot of the available whois data out there. The WhoXY whois API is used as a fallback if whois fails to return anything. It's only a fallback to reduce API calls and credit usage when it's not necessary.

Also, WhoXY's database can be searched with a company name, email address, or a keyword to find related domains. ODIN will take the provided organization name and any organization names returned from whois lookups and perform the reverse lookups through WhoXY for domains tied to the organization.

A WhoXY API key is required. [whoxy.com](https://www.whoxy.com/)

Note: A WhoXY account is free, but does eventually cost some money. It's inexpensive and they do offer a program for free API access, but you get a good number fo API credits for free with an account. Then, once you run out, whois credits are 2 for 1,000 credits and reverse whois credits are $2 for 250 credits. Just $4 will reload your account and set you up for a good long time.

## FAQ
**I get this syntax error. What's the deal?**

Please make sure you are using Python 3, not Python 2.7 or earlier. I recommend using `pipenv`.

**I get an error when ODIN tries to import a library. What's wrong?**

Like above, please make sure you are using Python 3. ODIN must be run in Python 3 and the requirements must be installed using `pip` or `pip3` for Python 3. To make sure all required libraries are installed for Python 3, use `pipenv` and the provided Pipfile. The Pipfile enforces Python 3, so you should be good to go.

See the installation instructions at the top.

**I don't have X API key, can I still use ODIN?**

Absolutely. If an API key is missing from the keys.config file, any checks using those keys will be skipped. You are strongly encouraged to go get the free API keys to get the most out of ODIN, but you can skip any you don't want.

API access for Shodan, Censys, and EmailHunter are defnitely worth it if you want to use the bare minimum.

**Why do you not like "why not" questions?**

If you ask "why not use X API" or "why not do Y like this," that's not very helpful. Presumably, the question is meant to convey the idea that X would be a good addition or Y is a bad way to accomplish a task and you want to know the reason it is not currently supported. The answer is most likely "I wasn't aware of this." That also means I don't know anything about it. :\)

If you have a suggestion for a change, service, or API, please explain what it does and provide some details explaining why you think it would be a good addition.

**Why not add support for the Clearbit API?**

Clearbit looks useful for OSINT, but the free tier is restricted to 20 API calls in a month. That may even be 20 API calls for the life of the account. The details are unclear. Either way, that's very restrictive and I want ODIN to be as simple and free to use as possible. The paid tiers are quite expensive.

**Why not use Wappalyzer?**

Wappalyzer is useful, but it's very difficult to automate fetching the results from Wappalyzer. Some tools can do this, but they use an unmaintained package called wappalyzer-python (https://github.com/scrapinghub/wappalyzer-python). This package still works, as far as I know, but there are several problems with it. The package has not been updated in three years, the developers have stated they have no plans to change that or support wappalyzer-python, and the package is Python 2. It could be used until it breaks one day, but the Python 2 bit is the real sticking point because ODIN is written in Python 3 and the two just are not compatible.

**Why not add support for the BuiltWith API?**

Like Clearbit, BuiltWith is a neat resource and some interesting details can be reviewed on the website. The API, however, is not free. The free version of the API won't give you any details, so at best it can be used to highlight a domain you may want to then review on the BuiltWith website. Scraping the website search results is certainly possible, but that could easily break and/or be unreliable.

Adding support for BuiltWith hasn't been ruled out, but the goal is to make ODIN entirely free to use.

**Why not use Full Contact's People API?**

Currently only the Company API is used. There are plans to incorproate the People API in the future.

**Does ODIN perform DNS brute forcing?**

No. Brute forcing can take a long time and there are many tools that take care of this quite well. Those tools are not so easy to incorporate into ODIN without just running the commands for those tools. For subdomain discovery via guessing, it's hard to beat Aquatone right now and there's alwas Fierce and DNSRecon.

For now, ODIN leverages DNS Dumpster, Netcraft, and SSL/TLS certificate data to collect subdomains to get you started. That should get you a good number of subdomains to get started.

### Special Thanks
A big thank you to a few contributors who gave me the OK to re-use some of their code:

* Ninjasl0th - Creator of the original scope verification script and all around cool dude!
* 0xF1 - Architect behind Cymon and a great guy to have on your team!
* GRC_Ninja - For providing great feedback regarding HTTP requests and RDAP.
* Troy Hunt - For giving me permission to use HaveIBeenPwned's REST API in this way.

And to these folks who have created/maintained some of the tools integrated into ODIN:

* Alton Johnson (altjx) - The creator of the original and very cool [PyFOCA](https://github.com/altjx/ipwn) that exists here in its new Python 3 form as a part of ODIN.
* Laramies - Creator of the awesome [TheHarvester](https://github.com/laramies/theHarvester).
* PaulSec - Creator of the [unofficial API for the DNS Dumpster](https://github.com/PaulSec/API-dnsdumpster.com).
* Daniel Grzalek (Dagrz) - Creator of [aws_pwn](https://github.com/dagrz/aws_pwn) and the reason why I was able to build out AWS recon options.