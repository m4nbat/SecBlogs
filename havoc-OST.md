# Intro
This blog provides a intriduction to reviewing github repoitories of offensive security tools (OST) to help build comamnd and control (C2) framework detection rules for Shodan.

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

<img width="904" alt="image" src="https://github.com/m4nbat/SecBlogs/assets/16122365/9f75b466-a459-45b0-acd2-5fe4793c9440">

Unfortunately, when looking at the facet analysis in Shodan there is limited uniqueness in some values to be able to use for a standalone detection. 

**Subject Name:**

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/dfec1577-3cc6-4a80-9f1c-5731d9c1aa40)

**Issuer**

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/2662c2bb-d88e-464b-bae7-8a34ba1999ee)

## JARM
The JARM however does allow us some detection opportunities:

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/23830bc7-53a3-4410-88b9-d34112025ed9)

"X-Havoc" ssl.jarm:"3fd21b20d00000021c43d21b21b43de0a012c76cf078b8d06f4620c2286f5e","3fd21b20d00000021c43d21b21b43d76e1f79b8645e08ae7fa8f07eb5e4202","2ad2ad16d2ad2ad0002ad2ad2ad2ad13962a56ecbfc3caaf51829946ab7fbe","40d1db40d0000001dc43d1db1db43d76e1f79b8645e08ae7fa8f07eb5e4202","40d1db40d0000001dc43d1db1db43de0a012c76cf078b8d06f4620c2286f5e","07d19d1ad21d21d00042d43d00000076e5b3c488a88e5790970b78ffb8afc2"



