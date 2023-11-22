# RisePro Stealer Panel Hunt

# Source:
Viriback c2 tracker is a great resource for doing a little e-crime infrastructure hunting https://tracker.viriback.com/index.php?q=Risepro

# Notes
I had a little dig into Risepro stealer panels using Fofa and there were some intriguing results.

First up lets grab a C2 IP from Virback

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/f8cd2315-bdbe-46dc-83d5-a0aa0430f4e8)

Now we have the IP lets check out the results in FOFA

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/b246442f-e1ee-4d2d-8822-8276d1c1c5f1)

Looking at those results I can already see that the HTTP header, server name, port, and JARM can be used to construct a hunting rule

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/8b6e805b-9835-4e85-9c72-5d1ba8ab0659)

27 results returned that all look like stealer panels 

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/2cacd995-d28c-460e-a221-0b3456e128f7)

Lets verify a few in a secure Tor browser:

![image](https://github.com/m4nbat/SecBlogs/assets/16122365/8e858d59-ad3f-47ff-b0aa-d8adbc09f2be)

It looks like they are all operational. The threat actor may have modified the page title on several of the panels to avoid searching for "RisePro"

#threathunting #malicious #infrastructure #malware #risepro #infostealer #informationstealer #stealer #fofa #threatinformed #threatintelligence #cti



