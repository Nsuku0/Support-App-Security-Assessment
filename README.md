## Support Operations Application Security Assessment

A web application security assessment of the TryHackMe Support Operations Platform, focusing on identifying vulnerabilities, evaluating their impact, and demonstrating how multiple weaknesses can be chained to achieve unauthorised access and operating-system command execution.

## Project Overview

| Detail | Description |
|---|---|
| **Assessment Type** | Web Application Penetration Test |
| **Target** | TryHackMe Support Operations Platform |
| **Assessment Period** | 5–11 October 2026 |
| **Assessor** | Nsuku Minyuku |
| **Overall Risk** | High |

## Objectives

- Identify security vulnerabilities in the web application.
- Assess authentication, session management, and access control mechanisms.
- Test input handling and local file access.
- Evaluate the potential impact of identified vulnerabilities.
- Document reproducible findings and recommend remediation measures.

## Key Findings

The assessment identified seven findings across four risk levels.

| Vulnerability | Risk Rating |
|---|---|
| Remote Code Execution via Command Injection | High |
| Insecure Session Management — Cookie Forgery | High |
| Local File Inclusion (LFI) | Medium |
| Broken Object Level Authorization (BOLA) | Medium |
| Lack of Rate Limiting | Medium |
| Unencrypted Communication (HTTP) | Low |
| Information Disclosure | Informational |

## Attack Chain Analysis

The assessment demonstrated how multiple weaknesses could be combined to progress from initial access to administrative account compromise and operating-system command execution.

The attack path involved:

1. Brute-force authentication against a valid account.
2. Privilege escalation through cookie forgery.
3. Administrative account discovery through broken object-level authorisation.
4. Password disclosure through local file access.
5. Administrative login using the discovered credentials.
6. Operating-system command execution through command injection.

This demonstrated the importance of assessing vulnerabilities both individually and in combination.

## Skills Demonstrated

- Web application penetration testing
- Authentication and session management testing
- Broken access control testing
- Local file inclusion and directory traversal testing
- Command injection testing
- HTTP request and response analysis
- Vulnerability risk assessment
- Technical reporting and remediation recommendations

## Report

See [`report.md`](report.md) for the detailed assessment report, including vulnerability descriptions, risk ratings, reproduction steps, business impact, and remediation recommendations.

## Disclaimer

This assessment was conducted against a designated TryHackMe lab environment for educational and portfolio purposes. Findings and conclusions apply to the assessed lab environment and should not be interpreted as an assessment of TryHackMe's production infrastructure.

All testing was limited to the authorised lab scope.
