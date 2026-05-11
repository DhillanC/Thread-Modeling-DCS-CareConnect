<p align="left">
  <img src="Archive/dcs_logo.png" alt="DCS Logo" width="500"/>
</p>

# Project Summary


This repository contains the results and findings of a comprehensive threat modeling workshop focused on DCS Care Connect team an their Health 360 solution, a web-facing healthcare application. The workshop, conducted over three hours, explored various AI-driven attack scenarios and provided a detailed runbook for analyzing and mitigating potential cyber threats. Attendees included key members from the DCS Care Connect engineering team, product management, DevEx, and DevSecOps. The goal was to identify critical vulnerabilities and strengthen the application's security posture through the application of industry-standard frameworks such as the MITRE ATT&CK and STRIDE models.





# Threat Modelling Workshop Summary



## Introduction

A 3-hour threat modeling workshop took place to detail the runbook scenario of multiple AI attacks against the web-facing healthcare application DCS Care Connect health 360.



## Attendees

DCS Care Connect Engineering team, Product Managers, DevEx Engineers, and the DevSecOps Team.



## Scope

4 Scenarios were run covering: (1) an AI-generated external phishing email utilizing admin credentials, (2) an Attack against Machine Processes and the data lake, (3) an SQL Injection attack, and (4) an Insider attack taking Quant algorithms.



## Methodology

All scenarios were run against the cyber attack kill chain, utilizing the Mitre Att&ack framework and STRIDE for control gap assessments. Culminating in identified risks. 



## Conclusion

A total of 4 high risks and 1 medium risk were found during the threat modeling workshop.



## Controls Required



- Regular security audits using ASVS specifically targeting the DCS Health 360 application to detect vulnerabilities and weaknesses in its security measures.

- Patch management to ensure the DCS Health 360 application and its underlying technologies are up-to-date and protected against known vulnerabilities.

- Comprehensive employee training on phishing awareness to educate users of the DCS Health 360 application about the risks of phishing attacks and how to identify and report suspicious emails.

- Implementation of a Web Application Firewall (WAF) tailored to the DCS Health 360 application's traffic to monitor and filter incoming requests for malicious activity.

- Deployment of Multi-factor Authentication (MFA) to enhance authentication security and prevent unauthorized access to the DCS Care Connect Health 360 application.

- Continuous network traffic monitoring to detect and respond to suspicious activity within the DCS Health 360 application's infrastructure.

- Implement Role-based Access Control (RBAC) within the DCS Health 360 application to limit access to sensitive health data and functionalities based on user roles and permissions.



# Threat Modelling Process Summary



```mermaid

mindmap
  root((Attack Begins))
    STRIDE/MITRE ATT&CK/Kill Chain
      Conduct Inherent Risk Assessment
      Create a Critical Asset List
        Schedule and Scope Threat Modelling Workshop
    Controls Required
      Risks and Mitigations
      Risk Summary
        Remediation workflow
          Slack
          JIRA
    Attack Scenarios
      Attack 1
      Attack 2
      Attack 3
      Attack 4
```
