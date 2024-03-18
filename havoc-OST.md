

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




**Possible rule:**






