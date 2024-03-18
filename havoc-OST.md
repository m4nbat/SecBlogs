

# HTTP related artifacts that could be used for detection:
We searched for the "content-type" keyword to attempt to identify HTTP headers or rpofiles that could be used for detection.

## SMB profile
First we identified an SMB profile for the Havoc framework

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/61d517c4-e872-438f-bd45-cf4b2abbaa1e)

**Possible rule:**
Content-Type: application/json; charset=utf-8 Server: Microsoft-HTTPAPI/2.0 X-Content-Type-Options: nosniff x-ms-environment: North Europe-prod-3,_cnsVMSS-6_26 x-ms-latency: 40018.2038 Access-Control-Allow-Origin: https://teams.microsoft.com

## Havoc default HTTP header information
Within the GitHub repository we identified a number of hard coded values that can be used for detection. In particular the "X-Havoc" keyword in the header.

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/c6436e1d-0c00-48d7-991c-b16035e7eb85)


**Possible rules:**
"X-Havoc"
https://www.shodan.io/search?query=%22X-Havoc%22
<img width="797" alt="image" src="https://github.com/m4nbat/SecBlogs/assets/16122365/10a0bde2-c136-42cc-b451-5e752a7fdfa0">

Expanding on the keyword we can make a more specific detection using the full HTTP header or the HTTP header hash.

"HTTP/1.1 404 Not Found Content-Type: text/html Server: nginx X-Havoc: true Date:  Content-Length: 146"
https://www.shodan.io/search?query=HTTP%2F1.1+404+Not+Found+Content-Type%3A+text%2Fhtml+Server%3A+nginx+X-Havoc%3A+true+Date%3A++Content-Length%3A+146



![image](https://github.com/m4nbat/SecBlogs/assets/16122365/f06cf658-0177-43a2-b790-94bb5bad4868)








