# Attack 1 Summary: AI Generated External phishing email utilising admin credentials



## Stages of the Attack



### Origins

The attack is initiated by an attacker leveraging a long history of cyber attack techniques. The attacker identifies DCS Health 360 application as a target due to its sensitive health data and public-facing nature.



### Reconnaissance

The attacker conducts research to identify vulnerabilities in the DCS Health 360 application. This includes gathering information about the application's architecture, technologies used, and potential weaknesses in its security measures.



### Weaponization

Exploit payloads are crafted specifically to target vulnerabilities identified in the DCS Care Connect application. Additionally, the attacker creates sophisticated phishing emails tailored to appear legitimate and enticing to users of the application.



### Delivery

Phishing emails containing malicious links or attachments are sent to users of the DCS Health 360 application. These emails may appear to come from trusted sources or mimic official communication from the application itself, increasing the likelihood of successful exploitation.



### Exploitation

The attacker exploits vulnerabilities in the DCS Health 360 application, such as SQL injection or cross-site scripting (XSS) vulnerabilities, to gain unauthorized access. By exploiting these vulnerabilities, the attacker can execute arbitrary code or extract sensitive information from the application's database.



### Installation

Once access is gained, the attacker establishes a foothold within the DCS Health 360 application's infrastructure. This may involve creating backdoor accounts or implanting malware to maintain persistent access to the system.



### Actions on Objectives

With access to the DCS Health 360 application, the attacker can exfiltrate sensitive health data stored within the application's database. Additionally, the attacker may manipulate patient records, tamper with medical information, or disrupt the application's functionality for malicious purposes.


```mermaid
flowchart LR
    style Reconnaissance fill:#0037b8,stroke:#000,stroke-width:2px,color:#fff
    style Weaponization fill:#F5B041,stroke:#000,stroke-width:2px
    style Delivery fill:#EB984E,stroke:#000,stroke-width:2px
    style Exploitation fill:#E59866,stroke:#000,stroke-width:2px
    style Installation fill:#DC7633,stroke:#000,stroke-width:2px
    style Command_Control fill:#CA6F1E,stroke:#000,stroke-width:2px,color:#fff
    style Actions_Objectives fill:#BA4A00,stroke:#000,stroke-width:2px,color:#fff

    Reconnaissance[Reconnaissance] -->|Identify admin targets| Weaponization[Weaponization]
    Weaponization[Weaponization] -->|Craft malicious link or attachment| Delivery[Delivery]
    Delivery[Delivery] -->|User clicks link, opens attachment| Exploitation[Exploitation]
    Exploitation[Exploitation] -->|Valid AWS or app admin creds| Installation[Installation]
    Installation[Installation] -->|Outbound C2 channel established| Command_Control[Command and Control]
    Command_Control[Command and Control] -->|Issue commands, lateral movement| Actions_Objectives[Actions on Objectives]
```
