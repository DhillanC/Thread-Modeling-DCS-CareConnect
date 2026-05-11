```mermaid

  sequenceDiagram

    participant Attacker

    participant DCSHealthApp

    participant CnCServer

    participant BackendServer

    participant User



    activate Attacker

    Attacker->>DCSHealthApp: Identify DCS Health 360 app

    DCSHealthApp->>Attacker: Application identified

    deactivate Attacker



    activate Attacker

    Attacker->>DCSHealthApp: Craft exploit for known vulnerabilities

    DCSHealthApp->>Attacker: Exploit crafted

    deactivate Attacker



    activate Attacker

    Attacker->>DCSHealthApp: Deploy phishing campaign targeting app users

    DCSHealthApp->>User: Phishing email sent

    activate User

    User->>DCSHealthApp: Clicks on malicious link/download attachment

    DCSHealthApp->>User: Malware downloaded

    deactivate User

    deactivate Attacker



    activate Attacker

    Attacker->>DCSHealthApp: Trick users into downloading malware

    DCSHealthApp->>BackendServer: Malicious payload executed

    BackendServer->>CnCServer: Communication established

    CnCServer->>BackendServer: Commands issued

    BackendServer->>CnCServer: Actions performed

    deactivate Attacker
```
