# ModernLoader Steeler (Avatar) C2 Hunt

# Source: 

- https://twitter.com/FalconFeedsio/status/1655466436729470977?s=19

# Hunt Flow

The initial pivot point was to use the C2 IP posted on the tweet 62[.]204[.]41[.]23.

I ran this against Shodan to identify the running services:

![image](https://user-images.githubusercontent.com/16122365/236936274-bb0fbe83-b6e0-4c9e-85e6-36f877374341.png)

Next check the SSH hash to see if its been reused on additional infrastructure. Unfortunately its no been reused.

![image](https://user-images.githubusercontent.com/16122365/236936432-0284a803-57d4-4bcb-8109-b24533b5f08f.png)

Next lets profile the web service runnning on port 443. At first glance the certificate information is interesting

![image](https://user-images.githubusercontent.com/16122365/236937664-bc817d6b-61c4-4654-8902-4611886108d3.png)

First I try the hash to see whether any additional results are returned but unfortuantely no additional servers are discovered:

![image](https://user-images.githubusercontent.com/16122365/236938008-94416b67-b2dc-4f83-8328-60165c605ece.png)

The SSL fingerprint returns nothing:

![image](https://user-images.githubusercontent.com/16122365/236939646-d0d60299-0887-4607-bb74-be7bc2edf79f.png)

Next I try using the header information but removing the date elements:

![image](https://user-images.githubusercontent.com/16122365/236938338-0bb6124e-cec1-4702-a2c8-2d090e5894d8.png)

This returns a lot of results but I think we may have something. Hmm what about that SSL certificate information:

![image](https://user-images.githubusercontent.com/16122365/236938704-6fcea094-d59c-49ee-a915-bf2b2b3eec80.png)

Okay so lets tighten up our query by including some of the certificate information as you can see below there are many ssl related filters but I'll just be a little bit lazy and use the more generic ssl: field syntax to search for some of the unique elements.

![image](https://user-images.githubusercontent.com/16122365/236940071-7a50283a-ed95-4fa5-98f1-698730244fd1.png)

Query:
ssl:O=Company ssl:OU=Department ssl:CN=www.example.com

Next we can combine the initial header information query with the ssl information query to refine our overall hunt query:
HTTP/1.1 403 Forbidden Server: nginx/1.18.0 Date: GMT Content-Type: text/html; charset=UTF-8 Content-Length: 0 Connection: keep-alive ssl:O=Company ssl:OU=Department ssl:CN=www.example.com

This is looking more interesting in terms of the results:

![image](https://user-images.githubusercontent.com/16122365/236944156-2d2b346a-cb84-4bbc-9437-a3bd885a1bc1.png)

Lets check some in VirusTotal and see what we get. As you can see our original IP from the tweet is flagged as ModernLoader C2

![image](https://user-images.githubusercontent.com/16122365/236943371-10d52587-acdf-4393-baac-ede441e74e03.png)

However some of the other results currently have no detections:

![image](https://user-images.githubusercontent.com/16122365/236943120-8e0a6f00-6c40-4695-8139-93af0bc2fefb.png)

![image](https://user-images.githubusercontent.com/16122365/236945326-dfd2f561-0b8d-4404-9bb9-d12c3e40716a.png)

![image](https://user-images.githubusercontent.com/16122365/236945744-7b03a5b4-84ac-4de6-99f2-2665b412efbf.png)

![image](https://user-images.githubusercontent.com/16122365/236945551-d32849d3-d53b-44b1-847d-f89348c79eaa.png)

Others appear to have links to Cobalt Strike:

![image](https://user-images.githubusercontent.com/16122365/236944934-048b7ae0-49e6-4936-8c47-dbc777d14528.png)

This was a brief example on how we can pivot off an initial network indicator and identify additional adversary infratsructure that can be used to inform defence. We could go further and start looking at the IP's and domains, file, URLs associated with them. Form this whois data may uncover further patterns that can be used to pivot to additional infrastructure. Each SSH key infromation can be checked to understand if keys have been reused.

For example the below hosts are reusing an SSH key which likely means they are connected. They are also on the same subnet and hosting provider.

![image](https://user-images.githubusercontent.com/16122365/236948280-6f78c774-4e24-4fd2-99fa-dd74b8701411.png)

Although not fully exhaustive I hope this gives you a flavour of hunting for potentially linked C2 infrastructure based on an initial ModernLoader malware network indicator. From further analysis of the results their appears there could be a connection between some of the results and Android based malware.

Combined query:
https://www.shodan.io/search?query=HTTP%2F1.1+403+Forbidden+Server%3A+nginx%2F1.18.0+Date%3A+GMT+Content-Type%3A+text%2Fhtml%3B+charset%3DUTF-8+Content-Length%3A+0+Connection%3A+keep-alive+ssl%3AO%3DCompany+ssl%3AOU%3DDepartment+ssl%3ACN%3Dwww.example.com
