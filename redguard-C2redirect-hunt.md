
# Writeup:

First I reviewed the certificate information on the RedGuard github repo. 

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/9787ea5d-2c35-49f2-949a-290455613459)


The below details were identified through searching the repo or reviewing screenshots of the RedGuard config. The info can be used to match the default certificate properties:


![image](https://github.com/m4nbat/SecBlogs/assets/16122365/33948c26-109c-48bd-9a8a-54f9b092c7b9)


## Cert:
Cert CommonName (default "*.aliyun.com")
Cert Country (default "CN")
Cert Locality (default "HangZhou")
Cert Organization (default "Alibaba (China) Technology Co., Ltd.")

We can start by using something like the below in shodan:

### SSL rule (starter from source code):
ssl:"CN" ssl:"HangZhou" ssl:"Alibaba (China) Technology Co., Ltd."

Next I reviewed the HTTP headers and HTML information from screenshots and config on github:

## HTTP:

### 301:
HTTP 301 Moved Permanently, length: 169
Proxy redirect URL (default "https://360.net")

360.com also present in a screenshot but when checking with the HTTP profile there were no results:


![image](https://github.com/m4nbat/SecBlogs/assets/16122365/caa0dc9d-25e2-4436-9502-7cbcd601ca76)


It appears the default ports are 80 and 443

## Port: 
80 or 443

### HTTP.HTML:

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/eca99554-8ea5-4dbb-8a47-68ffc059ba00)



### HTTP rule:
301:
http.html_hash:-618752581 http.headers_hash:-1625744203
https://www.shodan.io/search?query=http.html_hash%3A-618752581+http.headers_hash%3A-1625744203


![image](https://github.com/m4nbat/SecBlogs/assets/16122365/5b9360de-27ad-43f5-89a5-bd9c04339072)


### HTTP.HTML and SSL Header Rule:
301:
http.html_hash:-618752581 http.headers_hash:-1625744203 ssl:"Subject: C=CN L=HangZhou O=Alibaba (China) Technology Co., Ltd., CN=*.aliyun.com"
https://www.shodan.io/search?query=http.html_hash%3A-618752581+ssl%3A%22Subject%3A+C%3DCN%2C+L%3DHangZhou%2C+O%3DAlibaba+%28China%29+Technology+Co.%2C+Ltd.%2C+CN%3D*.aliyun.com%22


![image](https://github.com/m4nbat/SecBlogs/assets/16122365/06338e16-9756-42a8-ab71-d4385411e842)


### Initial Results:

**HTTP (Port 80)**

43.129.175.251
43.129.184.244
43.135.34.69
49.232.29.245

**SSL:**

42.[193.106.237
43.[129.175.251
43.[129.184.244
43.[135.34.69
**49.[232.29.245**
49.[234.165.142
69.[234.233.153
119[.27.176.138
123.[56.218.157
175.[24.254.64
218.[19.148.82

Checking the results in VT only 1 is flagged as bad and has links to Cobalt Strike. Lets refine our rule with SSL info.


## JARM:

301:
http.html_hash:-618752581 http.headers_hash:-1625744203
ssl.jarm:"3fd21b20d00000021c43d21b21b43d41226dd5dfc615dd4a96265559485910","2ad22b00000000022c43d22b22b43d3795b2a696610c3ae44909dcdcb797f2"
https://www.shodan.io/search?query=http.html_hash%3A-618752581+http.headers_hash%3A-1625744203+ssl.jarm%3A%223fd21b20d00000021c43d21b21b43d41226dd5dfc615dd4a96265559485910%22


![image](https://github.com/m4nbat/SecBlogs/assets/16122365/b6eb76dd-cbab-465b-a764-130579b87081)

## Results:

42.193.106.237
43.129.175.251
43.129.184.244
43.135.34.69
**49.232.29.245**

## Validating the HTTP 301 rule results:

These look better but still only 1 is flagged as malicious. 


![image](https://github.com/m4nbat/SecBlogs/assets/16122365/05c7f6c3-a4eb-418b-8b75-3c6684098477)


There are no communicating files for some of these IPs so this could mean that they are not yet used / detected.


# Another redirect possibility:

When looking at results in Shodan I also noticed that there were some results with a 307 response code. This was when I used the SSL info in the previous rules (ssl:"Subject: C=CN L=HangZhou O=Alibaba (China) Technology Co., Ltd., CN=*.aliyun.com") and added the keyword search for "https://360.net" 


![image](https://github.com/m4nbat/SecBlogs/assets/16122365/83489a8f-e874-4622-b089-5dad105b7feb)


## HTTP:

307 (Unsure whether this is a customisation to use 307 (not seen 307 in the github code):
HTTP/1.1 307 Temporary Redirect
Content-Type: text/html; charset=utf-8
Location: https://360.net
Date:
Content-Length: 51


![image](https://github.com/m4nbat/SecBlogs/assets/16122365/079d810e-ba16-4a96-bda7-7fa347795c79)


Sames rule using the hash value rather than the raw header text: 
307(Unsure whether this is a customisation to use 307 (not seen 307 in the github code)::
http.headers_hash:1926582344 http.html_hash:155817744
https://www.shodan.io/search?query=http.headers_hash%3A1926582344+http.html_hash%3A155817744


![image](https://github.com/m4nbat/SecBlogs/assets/16122365/d6c52b49-2fdd-4cd1-bab3-3fb040267f1b)


## HTTP & SSL:

307 (Unsure whether this is a customisation to use 307 (not seen 307 in the github code)::
http.headers_hash:1926582344 http.html_hash:155817744
ssl.jarm:"3fd21b20d00000021c43d21b21b43d41226dd5dfc615dd4a96265559485910"
https://www.shodan.io/search?query=http.headers_hash%3A1926582344+http.html_hash%3A155817744+ssl.jarm%3A%223fd21b20d00000021c43d21b21b43d41226dd5dfc615dd4a96265559485910%22


![image](https://github.com/m4nbat/SecBlogs/assets/16122365/930fa909-c2e0-4914-85ad-8baac9573095)


307 (Unsure whether this is a customisation to use 307 (not seen 307 in the github code)::
http.headers_hash:1926582344 http.html_hash:155817744
ssl:"Subject: C=CN L=HangZhou O=Alibaba (China) Technology Co., Ltd., CN=*.aliyun.com"
https://www.shodan.io/search?query=http.headers_hash%3A1926582344+http.html_hash%3A155817744+ssl%3A%22Subject%3A+C%3DCN+L%3DHangZhou+O%3DAlibaba+%28China%29+Technology+Co.%2C+Ltd.%2C+CN%3D*.aliyun.com%22


![image](https://github.com/m4nbat/SecBlogs/assets/16122365/0c5d1607-d8ff-4d02-81ab-2a208f3bd321)



## Validating the new 307response code rule results:

Review results in VT for the 301 and 307 related hosts to check:

### 307:

23[.105.197.219
43[.138.110.8
114[.115.145.188
121[.42.9.148
150[.158.137.47
45[.141.136.133
123[.60.164.87
142[.171.229.85
157[.245.222.152

The first is flagged as malicious

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/ac497fbf-06ff-4073-97c5-b08f0fc81575)

Looking at communicating files it appears to be linked to a cobalt strike beacon

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/86706066-0d73-4fff-930d-77887481c95a)

Our second IP appears to have historic links to cobalt strike:


![image](https://github.com/m4nbat/SecBlogs/assets/16122365/87e12408-4975-43da-b9a2-e453868074f3)


Our next check also shows Cobalt Strike so it would appear the rule is good: 

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/18208389-b0b6-4be4-bec8-165879cf9d16)

Looking at one of the US based IP addresses this has the redirector and it also has cobalt strike hosted on the same server: 

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/36f3ab21-649d-45b9-bce5-03c0a536f013)

The rule looks good and we have identified additional infrastructure :)

