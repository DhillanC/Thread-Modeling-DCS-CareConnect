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
    A[Reconnaissance<br/>OSINT on DCS staff,<br/>LinkedIn, GitHub repos] -->|Identify admin targets| B[Weaponization<br/>AI-generated phishing<br/>+ payload]
    B -->|Craft malicious link / attachment| C[Delivery<br/>Spear-phishing email<br/>to DCS admin]
    C -->|User clicks link / opens attachment| D[Exploitation<br/>Credential harvest<br/>or browser RCE]
    D -->|Valid AWS / app admin creds| E[Installation<br/>Backdoor user,<br/>persistence on backend]
    E -->|Outbound C2 channel established| F[Command & Control<br/>HTTPS beacon to attacker C2]
    F -->|Issue commands<br/>lateral movement| G[Actions on Objectives<br/>Exfil PHI,<br/>manipulate records,<br/>deploy ransomware]

    classDef recon fill:#0037b8,stroke:#000,color:#fff;
    classDef weap fill:#F5B041,stroke:#000;
    classDef deliv fill:#EB984E,stroke:#000;
    classDef expl fill:#E59866,stroke:#000;
    classDef install fill:#DC7633,stroke:#000;
    classDef c2 fill:#CA6F1E,stroke:#000,color:#fff;
    classDef act fill:#BA4A00,stroke:#000,color:#fff;
    class A recon;
    class B weap;
    class C deliv;
    class D expl;
    class E install;
    class F c2;
    class G act;
