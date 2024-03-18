

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

To do this we can check the facet analysis section and understand how many unique headers make up the data set:

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/759dcc3e-01de-40c7-85e6-8a7dc5c05f1a)

Using these headers we can now track specific Havoc header configs seperately which can in some cases allow us to track specific versions of the framework or in some cases specific activity groups or threat actors.

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/73d20de2-3175-4ecd-8b72-3cbfea0581aa)

The new query leveraging the unique header hashes, would look like the below image:

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/04399ff6-63f5-4343-a208-41e244c8c14e)

## Certificate based detection
In addition to headers we can also look for default certificate configuration to facilitate detetcion of the framework online.








