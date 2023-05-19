# Attackers Leveraging Google Cloud Services for Command and Control

Attackers are coming up with new and creative ways to abuse cloud services for their own purposes. This blog will cover how attackers have leveraged Google cloud services to provide a command and control infrastructure to interact with their targets. The attack flow of an attack that leverages Google cloud infrastructure is illustrated below:

![image](https://user-images.githubusercontent.com/16122365/235530695-58e2b2cc-1550-480f-867d-186380492599.png)

Utilising popular cloud infrastructure provides several benefits to the attacker:

- Low cost infrastructure
- Traffic tends to blend in with normal personal / business internet usage
- Not many organisations can/will block Google cloud services 
- Out of the box not many security tools pick up the network communication as malicious

## Threat intelligence / Why we care!

"In Google's April 2023 Threat Horizons Report, TAG (Threat Analysis) attributed a campaign to APT41 in which they were found abusing the GC2 (Google Command and control) tool in data theft attacks. APT41 is a Chinese state sponsored hacking group which has been around since 2012. They are also financially motivated and have been seen targeting Telecommunication, Healthcare and Technology sectors.

GC2 (Google Command and Control) is a Command and Control application that allows an attacker to execute commands on the target machine using Google Sheet and exfiltrates data using Google Drive.

In this campaign they used a phishing email with links to a password protected file which was hosted on a google drive. This automatically incorporates the GC2 tool that reads commands from google sheets and exfiltrate data using the cloud storage service. GC2 also enables the attacker to download additional files from Drive into the Victim system.

APT41 is known for deploying a wide variety of malware on compromised systems although it's not clear which malwares was used in this campaign. They have been observed using cobalt strike and deploying Winnti in past campaigns.

The use of GC2 by APT41 demonstrates the groups tendency to use open-source tools to increase chances of evading detection and also hinder activities such as attribution during post-incident investigations. Threat groups already utilise legitimate tools for this purpose during intrusions, such as Remote Management Software like TeamViewer, Atera etc. and this move by APT41 is another move in this direction."

## Install pre-requistes and compile the go program

First we need to grab the project from Github:

`git clone https://github.com/looCiprian/GC2-sheet`

Make sure go is installed, if it isn't download from: 

`https://go.dev/doc/install`

On Linux you can download the package and then install using:

`sudo tar -C /usr/local -xzf go1.20.3.linux-amd64.tar.gz`

Then add go to your PATH:

`export PATH=$PATH:/usr/local/go/bin`

Check your version and were ready to go (excuse the pun :))

`go version`

Next we need to compile the code using build.

`go build gc2-sheet.go`

To generate a windows compatible executable we ensure we are in the project folder and set OS and architecture variables:

```
export GOOS=windows
export GOARCH=amd64
```

Then we build the Windows binary using:

`go build -o gc2-sheet.exe`

Before restoring the operating system and architecture variables:

```
unset GOOS
unset GOARCH
```

## Configure the Google cloud components

1. Create a folder on Google Drive that can be used to upload and download data from/to the victim.

![image](https://user-images.githubusercontent.com/16122365/235531473-a5b1f0e7-1269-4f39-943c-7e5abf292e83.png)

2. Create a Google Sheet that will be used to send commands to the victim and to receive and record their output in the Google Sheet.

images.githubusercontent.com/16122365/235531490-d27d8dc9-17c4-4560-aaa1-4468971d9d8d.png)

3. Create a service account at [https://console.cloud.google.com/](https://console.cloud.google.com/)

![image](https://user-images.githubusercontent.com/16122365/235531719-f275eca6-7d95-4854-a4b8-e7586b695deb.png)

![image](https://user-images.githubusercontent.com/16122365/235531746-38a527db-eb21-4209-8a91-e5ebdc147244.png)

4. Once created, create and download a JSON key file for the service account.

![image](https://user-images.githubusercontent.com/16122365/235531768-3d943c72-dd1a-4bf3-bc5c-85df14607297.png)

5. Add the service principal as an editor on the google drive folder and google sheets document.

![image](https://user-images.githubusercontent.com/16122365/235531871-59d6aa73-27a4-47a4-a667-81b27b6c8b9a.png)

## Testing the C2 Server

Transfer the windows binary created initial and the key file to the test victim host. It's possible to hardcode these to the binary however as its bank holiday weekend and we are just demonstrating capability we will use the command line to specify our keys and credentials.

![image](https://user-images.githubusercontent.com/16122365/235531953-562ddc40-ea6e-4991-8e0e-bffa1f324cf8.png)

Once run, you should see a new tab created in your google sheets.

![image](https://user-images.githubusercontent.com/16122365/235531991-e72cc064-fa68-44d8-9f6f-e101ce9bc65f.png)

### Example commands and output:

#### Discovery comamnds

The image illustrates the commands that can be run from the Google sheet. These are processed by the agent running on the victim, executed on the host, before the result/output is written back to the google sheet.

![image](https://user-images.githubusercontent.com/16122365/235532039-96d9a6ec-2cdc-4da1-be68-e2a19b581679.png)

#### Data ingress and exfiltartion commands

In addition to enumeration it is also possible to download file to the victim and exfiltrate data to a google drive.

![image](https://user-images.githubusercontent.com/16122365/235532779-82d85004-7da2-4ffe-8053-de099de750f8.png)

As you can see ien the below image the files are uploaded to Google drive

![image](https://user-images.githubusercontent.com/16122365/235532833-f20185cf-2b1e-4f90-8cb4-715c35292649.png)

## Detection

In this scenario we understand that the C2 framework will programmatically interact with the endpoints on the *.googleapis.com domain. In particular we will want to monitor for oauth2.googleapis.com, sheets.googleapis.com, and drive.googleapis.com.

However, its worth noting that calls to .googleapis.com will be quite noisy if we don't include some process exclusions. For example browsers and some desktop apps may cause false positives. To start with we will want to exclude common browser process filenames, updaters for chrome or legitimate other google related applications.

### Kusto query to detect this network behaviour

The DeviceNetworkEvents table in MDE advanced hunting would seem an obvious choice to start with. First I will execute a query to identify connections to the *.googleapis.com domain.

![image](https://user-images.githubusercontent.com/16122365/235533640-15f12acd-5e36-46ea-b65f-d50f099492d1.png)

Based on what we see above we can build out the start of our query. The below line of code can be used to exclude common browser process filenames but their are likely many more that need to be added based on evaluation of the operating environment.

![image](https://user-images.githubusercontent.com/16122365/235533734-fe856df1-e468-40ba-9ca7-0fd89fb722c8.png)

Next we can make sure were only looking at connections to the endpoints we care about. The query below works well as it is and can be used to detect the behaviour related to the GC2 offensive tool.

![image](https://user-images.githubusercontent.com/16122365/235533844-8c48d4a1-fe58-47c3-9542-1984844b7ee8.png)

In the end I decided to create an additional field called visitedURLs that would hold a dynamic set of the RemoteUrl's visited in the time period of the query. This can be used to do analysis and filtering in a noisy environment.

![image](https://user-images.githubusercontent.com/16122365/235534156-c7883783-0ee3-4ce0-b2c3-8124656f8ac5.png)

I then used project to produce a more clear field output for the analyst and to add a count of the number of connections identified in the array.

![image](https://user-images.githubusercontent.com/16122365/235534084-90a42f90-8e85-4fbe-adf0-d63ca57dc86d.png)

### Final KQL query

```
let excludedProcessFileNames = datatable (browser:string)["teams.exe","GoogleUpdate.exe","outlook.exe","msedge.exe","chrome.exe","iexplorer.exe","brave.exe","firefox.exe"]; //add more browsers or mail clients where needed for exclusion 
DeviceNetworkEvents 
| where not(InitiatingProcessFileName has_any (excludedProcessFileNames))
| where RemoteUrl has_any ("oauth2.googleapis.com","sheets.googleapis.com","drive.googleapis.com") and isnotempty(InitiatingProcessFileName)
| summarize visitedURLs=make_list(RemoteUrl) by ActionType, DeviceName, InitiatingProcessAccountName, InitiatingProcessParentFileName, InitiatingProcessFileName
| project ActionType, DeviceName, InitiatingProcessAccountName, InitiatingProcessParentFileName, InitiatingProcessFileName, visitedURLs, Connections=array_length(visitedURLs)
//| where visitedURLs contains "oauth2.googleapis.com" and visitedURLs has_any ("sheets.googleapis.com","drive.googleapis.com") // may allow for higher fidelity as the GC2 go application communicates to both the google drive folder and sheets API.
```

As you can see the activity has been picked up based on the telemetry forwarded from the victim host

![image](https://user-images.githubusercontent.com/16122365/235533237-9b3a5ca2-cb43-4c88-aac5-7653e08537ba.png)

## Identifying files created by the suspicious process making connections to Google APIs

Below I will cover off the ability to download files (T1544: Ingress Tool Transfer) from the C2. In this instance, we have downloaded an enumeration script that can be used by the attacker to perform discovery of the local system and connected network.

The below query is an example of how we can use our initial query to identify suspicious process filenames that are communicating with Google APIs and to use the distinct list of names as a filter to search for files created on the host which could indicate a tool being transferred from the C2 to the victim.

```
let excludedProcessFileNames = datatable (browser:string)["teams.exe","GoogleUpdate.exe","outlook.exe","msedge.exe","chrome.exe","iexplorer.exe","brave.exe","firefox.exe"]; //add more browsers or mail clients where needed for exclusion 
let processComWithGoogleAPI = DeviceNetworkEvents 
| where not(InitiatingProcessFileName has_any (excludedProcessFileNames))
| where RemoteUrl has_any ("oauth2.googleapis.com","sheets.googleapis.com","drive.googleapis.com") and isnotempty(InitiatingProcessFileName)
| distinct InitiatingProcessFileName;
DeviceFileEvents
| where ActionType == "FileCreated" and InitiatingProcessFileName in~ (processComWithGoogleAPI)
```

An example output from the query is shown below

![image](https://user-images.githubusercontent.com/16122365/235535924-11e36956-fc39-4504-adea-7e15a46d92f7.png)

The below query can be used to pivot for processes and commandlines associated with the processes that had performed the initial network connections.

```
let excludedProcessFileNames = datatable (browser:string)["teams.exe","GoogleUpdate.exe","outlook.exe","msedge.exe","chrome.exe","iexplorer.exe","brave.exe","firefox.exe"]; //add more browsers or mail clients where needed for exclusion 
let processComWithGoogleAPI = DeviceNetworkEvents 
| where not(InitiatingProcessFileName has_any (excludedProcessFileNames))
| where RemoteUrl has_any ("oauth2.googleapis.com","sheets.googleapis.com","drive.googleapis.com","www.googleapis.com") and isnotempty(InitiatingProcessFileName)
| distinct InitiatingProcessFileName;
DeviceFileEvents
| where ActionType == "FileCreated" and InitiatingProcessFileName in~ (processComWithGoogleAPI)
```

An example output from the query is shown below

![image](https://user-images.githubusercontent.com/16122365/235536726-4e29af44-d7df-41d8-b513-797400daebaa.png)


## Identifying data exfiltration

GC2 provides the ability to exfiltrate data (T1567.002:Exfiltration Over Web Service: Exfiltration to Cloud Storage) from the endpoint to the Google Drive. The image below illustrates how the data appears in the specified Google drive.

![image](https://user-images.githubusercontent.com/16122365/235534782-46f06ea9-c89a-48a6-affb-7bd86332d440.png)

Defender for Cloud Apps is pretty good at this and could be used to identify and alert on a high volume of data uploaded to Google. This could prove tricky if G Suite is being used in the organisation.

![image](https://user-images.githubusercontent.com/16122365/235534607-f73f210c-72e1-4cf7-b795-2513eaa80ee7.png)

## Conclusion

In conclusion, attackers are constantly adapting and finding new ways to leverage legitimate services for malicious purposes. In this case, we've demonstrated how Google Cloud Services can be used as a command and control infrastructure. By understanding the tools, procedures, and operations used by current threat groups we can perform micro emulation and understand what telemetry is available to improve our ability to detect, prevent, and respond to adversarial operations. This provides a mechanism for improving overall security posture and awareness in the faceof new threats.

A big thank you to Tom Wood, Dominic Mortimer, and Joshua Penny for contributing and supporting the development of this blog!


## SIGMA Rule:

https://github.com/SigmaHQ/sigma/blob/2a2a4d9cd0ad187607c49ea815a1c96da853b41f/rules/windows/network_connection/net_connection_win_google_api_non_browser_access.yml
