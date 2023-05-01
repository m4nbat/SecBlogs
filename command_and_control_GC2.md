# Attackers Leveraging Google Cloud Services for Command and Control

Attackers come up with creative ways to abuse cloud services for their own purposes. This blog will cover how attackers have leveraged Google cloud services in the form of the Google API, Drive and Sheets services to provide a command and control infrastructure to interact with the victim. The attack flow of an attack that leverages Google cloud infrastructure is illustrated below:




Utilising popular cloud infrastructure provides several benefits to the attacker:

- Low cost infrastructure
- Traffic tends to blend in with normal personal / business internet usage
- Not many organisations can/will block Google cloud services 
- Out of the box not many security tools pick up the network communication as malicious



## Setting Up the Environment

First we need to grab the project from Github:

```
git clone https://github.com/looCiprian/GC2-sheet
```
