# Attackers Leveraging Google Cloud Services for Command and Control

Attackers come up with creative ways to abuse cloud services for their own purposes. This blog will cover how attackers have leveraged Google cloud services in the form of the Google API, Drive and Sheets services to provide a command and control infrastructure to interact with the victim. The attack flow of an attack that leverages Google cloud infrastructure is illustrated below:

![image](https://user-images.githubusercontent.com/16122365/235530695-58e2b2cc-1550-480f-867d-186380492599.png)

Utilising popular cloud infrastructure provides several benefits to the attacker:

- Low cost infrastructure
- Traffic tends to blend in with normal personal / business internet usage
- Not many organisations can/will block Google cloud services 
- Out of the box not many security tools pick up the network communication as malicious



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

However, its worth noting that calls to .googleapis.com will be quite noisy if we don't include some process exclusions. For example browsers and some desktop apps may cause false positives. Toi start with we will want to exclude common browser process filenames, updaters for chrome or legitimate other google related applications.



### Kusto query to detect this network behaviour

The DeviceNetworkEvents table in MDE advanced hunting would seem an obvious choice to start with. First I will execute a query to identify connections to the *.googleapis.com domain.

![image](https://user-images.githubusercontent.com/16122365/235533462-2ae64e9e-4ac6-4514-8189-ea5e49aca5c8.png)


The final query can be seen below:

```
let excludedProcessFileNames = datatable (browser:string)["teams.exe","GoogleUpdate.exe","outlook.exe","msedge.exe","chrome.exe","iexplorer.exe","brave.exe","firefox.exe"]; //add more browsers or mail clients where needed for exclusion 
DeviceNetworkEvents 
| where not(InitiatingProcessFileName has_any (excludedProcessFileNames))
| where RemoteUrl has_any ("oauth2.googleapis.com","sheets.googleapis.com","drive.googleapis.com") and isnotempty(InitiatingProcessFileName)
| summarize visitedURLs=make_list(RemoteUrl) by ActionType, DeviceName, InitiatingProcessAccountName, InitiatingProcessParentFileName, InitiatingProcessFileName
| project ActionType, DeviceName, InitiatingProcessAccountName, InitiatingProcessParentFileName, InitiatingProcessFileName, visitedURLs, Connections=array_length(visitedURLs)
//| where visitedURLs contains "oauth2.googleapis.com" and visitedURLs has_any ("sheets.googleapis.com","drive.googleapis.com") // may allow for higher fidelity as the GC2 go application communicates to both the google drive folder and sheets API.
```![image](https://user-images.githubusercontent.com/16122365/235533287-488b4d14-e2d6-4e07-aee1-62ea11cebefe.png)

As you can see the activity has been picked up based on the telemetry forwraded from the victim host

![image](https://user-images.githubusercontent.com/16122365/235533237-9b3a5ca2-cb43-4c88-aac5-7653e08537ba.png)
