## Support Operations Application Security Assessment

**Target:** Support Operations Platform  
**Assessment Type:** Web Application Penetration Test  
**Assessor:** Nsuku Minyuku  
**Assessment Date:** 5 - 11 October 2026  

## Executive Summary

A security assessment was conducted against the Support web application to identify vulnerabilities that could allow an attacker to gain unauthorised access, disclose sensitive information, or execute commands on the underlying server. Overall, the application's current security posture presents a high risk to the organisation.

The assessment identified seven security findings ranging from Informational to High risk. The most significant findings were remote code execution through command injection and insecure session management. Successful exploitation of these vulnerabilities could allow an attacker to bypass intended access controls, access privileged functionality and execute operating-system commands with the privileges of the web application process. This could expose sensitive information, compromise application data and disrupt services.

Additional weaknesses included local file inclusion, broken object-level authorisation, insufficient protection against automated authentication attempts, unencrypted HTTP communication and information disclosure. The assessment demonstrated that several of these vulnerabilities could be chained together, allowing an attacker to progress from initial access to administrative account compromise and operating-system command execution.

Immediate remediation should prioritise command injection and insecure session management, followed by the access control, file handling and authentication weaknesses that facilitate privilege escalation. The application should also be configured to enforce encrypted communications and minimise unnecessary information disclosure to reduce its overall exposure to attack.






## Summary of Vulnerabilities


| **Reference** | **Vulnerability** | **Risk Rating** |
|---|---|---|
| 4.1 | Remote Code Execution via Command Injection | High |
| 4.2 | Insecure Session Management — Cookie Forgery | High |
| 4.3 | Local File Inclusion (LFI) | Medium |
| 4.4 | Broken Object Level Authorization (BOLA) | Medium |
| 4.5 | Lack of Rate Limiting | Medium |
| 4.6 | Unencrypted Communication (HTTP) | Low |
| 4.7 | Information Disclosure — Email Address and Server Version | Informational |


## Attack Chain and Combined Impact

The assessment demonstrated how multiple vulnerabilities could be chained to progress from unauthenticated access to privileged administrative access and, ultimately, operating-system command execution.

### Stage 1: Initial Access Through Brute-Force Authentication

The attacker visits the Support application's login page and identifies the publicly displayed support email address, `help@support.thm`. This provides a valid username for authentication attempts. The login endpoint does not adequately restrict repeated authentication attempts, allowing the attacker to conduct a brute-force attack against the account. The attacker successfully identifies the password and authenticates to the application as the IT support user.

**Related finding:** Finding 4.5 — Lack of Rate Limiting (Medium).

**Impact:** Unauthorised access to the application using compromised credentials.

### Stage 2: Privilege Escalation Through Cookie Forgery

After logging in, the attacker inspects the application's cookies and identifies the `isITUser` cookie. The application uses this client-controlled value to determine whether the user has IT support privileges. The attacker discovers that the cookie contains an MD5 hash of a predictable boolean value. By replacing the hash for `false` with the hash for `true`, the attacker bypasses the intended privilege restriction and gains access to the IT support interface.

**Related finding:** Finding 4.2 — Insecure Session Management (High).

**Impact**: Unauthorised access to functionality intended for IT support users.

### Stage 3: Administrative Account Discovery Through BOLA

The elevated interface exposes an API viewing function. Upon interacting with this feature, the attacker intercepts the baseline HTTP request and manually injects the URI parameter id=1 into the request line. Because the API does not adequately enforce object-level authorisation, it returns profile information belonging to the requested user. The response reveals the email address associated with the `specialadmin` account.

**Related finding:** Finding 4.4 — Broken Object Level Authorization (Medium).

**Impact:** Disclosure of privileged account information, enabling the attacker to identify the administrative account for subsequent exploitation.

### Stage 4: Master Password Disclosure Through Local File Inclusion

The application provides a theme-selection feature that loads local files based on the `skin` parameter. The attacker modifies the parameter from a legitimate theme selection, such as `skin=red`, to a traversal payload, `skin=../config`.

The application processes the modified parameter without adequately restricting access to the intended theme files. The attacker accesses the configuration file and discovers the master password `Support@110`.

The disclosed password provides the credential required for the next stage of the attack.

**Related finding** Finding 4.3 — Local File Inclusion / File Disclosure (Medium).

**Impact:** Exposure of sensitive configuration information and a password that enables further privilege escalation.

**Stage 5:** Administrative Access Through Credential Reuse

Having obtained the administrative email address through BOLA and the master password through local file disclosure, the attacker attempts to authenticate as specialadmin using the discovered credentials.

To successfully bypass the application's authentication logic, the attacker must slightly modify the extracted password by stripping the special character, transforming the discovered `Support@110` into `Support110`. The application accepts these manipulated credentials, granting the attacker full access to the administrative account and its privileged functionality.

**Related findings:** Finding 4.3 — Local File Inclusion / File Disclosure (Medium), and Finding 4.4 — Broken Object Level Authorization (Medium).

**Impact:** Compromise of the administrative account through the combination of information disclosure and exposed credentials.

### Stage 6: Operating-System Command Execution

After obtaining administrative access, the attacker navigates to the date and time functionality. The application passes user-supplied input to the underlying operating system without adequate protection against command injection.

The attacker modifies the submitted input to append an operating-system command. The application executes the injected command, allowing the attacker to retrieve information from the host, including the contents of `/etc/passwd`.

**Related finding:** Finding 4.1 — Remote Code Execution via Command Injection (High).

**Impact:** Arbitrary operating-system command execution with the privileges of the web application process. The attacker may be able to access or modify additional resources available to that process.

### Overall Assessment

The attack chain demonstrates that weaknesses in authentication, session management, authorisation and file handling can combine to compromise an application's administrative boundary. The publicly disclosed support email assists with identifying a valid account, the lack of rate limiting facilitates password guessing, cookie forgery exposes IT support functionality, BOLA reveals the administrative account, and local file disclosure exposes a password that enables administrative login.

Once administrative access is obtained, command injection provides a route from application-level compromise to operating-system command execution.

The combined attack path presents a High overall risk because it culminates in arbitrary command execution on the host. This overall rating does not require every individual finding to be rated High; each finding retains its own severity based on its demonstrated impact and exploitation requirements.


## Vulnerability Descriptions
### Vulnerability Grading
The vulnerabilities identified in this report have been classified according to the potential risk they present to the Support Operations Platform.  
**High:** A vulnerability that could allow an attacker to gain unauthorised access to a system or sensitive data, or to control, alter or delete electronic assets.  
**Medium:** A vulnerability that could cause significant harm when combined with other weaknesses or when specific conditions are met.  
**Low:** A vulnerability with limited impact or an extremely low likelihood of successful exploitation.  
**Informational:** A finding that does not directly enable exploitation but may provide information useful for further attacks or deviate from security best practice.  

### 4.1 Remote Code Execution via Command Injection
**Risk Rating:** HIGH  

**Description**  
The application's administrative interface contains a date and time functionality that fails to properly validate user-supplied input before passing it to the underlying operating system. This allows an authenticated attacker to append arbitrary system commands to the expected input. The vulnerability arises from unsafe handling of user input when executing operating-system commands.


**Business Impact** 
Successful exploitation allows an attacker to execute arbitrary operating-system commands with the privileges of the web application process. Depending on those privileges, an attacker may access or modify application data, read accessible files, and compromise the confidentiality, integrity or availability of the host system. Additional privileges or weaknesses could enable further compromise.


**Reproduction Steps**  
1. Authenticate to the administrative dashboard using the valid credentials `specialadmin`.
2. Navigate to the date and time functionality.
3. Intercept the HTTP request submitting the date format.
4. Modify the input to include a command separator and an additional command, such as `; whoami`.
5. Forward the request and inspect the HTTP response for evidence of command execution.


**Remediation**  
Avoid passing user-controlled input directly to an operating-system shell. Use built-in programming language functions or libraries to handle date and time formatting. Where operating-system commands are unavoidable, use safe process-execution APIs that do not invoke a shell unnecessarily. Apply strict allowlist validation to input and ensure the application runs with the minimum privileges required.

### 4.2 Insecure Session Management — Cookie Forgery
**Risk Rating: HIGH**  

**Description**  
The application uses a client-controlled cookie named isITUser to determine whether a user has IT support privileges. The cookie value is a predictable MD5 hash of a boolean string rather than a securely protected session attribute. Because the application does not adequately protect or validate this privilege indicator, an attacker can calculate the hash corresponding to true, replace the cookie and obtain access to functionality intended for IT support users. In the context of this assessment, this privilege escalation was the critical pivot point that allowed the discovery of the administrative credentials, ultimately leading to total system compromise.


**Business Impact**  
Successful exploitation allows a lower-privileged user to bypass the intended privilege boundary and access the internal IT support interface and ticketing functionality. This may expose sensitive support information and enable further attacks against the application.
The finding is rated High because the demonstrated privilege escalation provides unauthorised access to read confidential information.


**Reproduction Steps**  
1. Authenticate using the low-privileged account `help@support.thm`.
2. Inspect the browser's cookies and identify the `isITUser` cookie.
3. Determine that the cookie contains the MD5 hash of the string false.
4. Calculate the MD5 hash of true, which is `b326b5062b2f0e69046810717534cb09`.
5. Replace the existing cookie value with the calculated hash.
6. Refresh the application and observe whether the IT support dashboard becomes accessible.


**Remediation**  
Do not store or trust privilege levels in client-modifiable cookies. Maintain authentication state and user roles on the server, using securely generated session identifiers. Enforce server-side authorisation checks on all privileged operations, irrespective of cookie contents. Set appropriate cookie attributes, including HttpOnly, Secure and SameSite, as additional session protections.

### 4.3 Local File Inclusion (LFI)
**Risk Rating:** MEDIUM  

**Description**  
The dashboard's theme selector uses a URL parameter, skin, to determine which local file is loaded. The application does not adequately restrict the parameter to approved theme files, potentially allowing directory traversal and access to unintended files outside the expected web directory. During testing, this weakness was used to access a configuration file containing the master password Support@110, which was subsequently used with the administrator email address disclosed through Finding 3 to obtain administrative access.


**Business Impact**  
Unauthorised file access may expose application configuration details, credentials or other sensitive information. In this case, the disclosed information contributed to the wider attack path. The severity is assessed as Medium because the significant escalation impact depends on the surrounding vulnerabilities and the information obtained, rather than being attributed solely to file inclusion.


**Reproduction Steps**  
1. Authenticate to the application using the available IT support credentials.
2. Intercept the GET request responsible for loading the dashboard theme.
3. Modify the skin parameter to attempt directory traversal, using the tested payload `?skin=../config`.
4. Forward the request and render the resulting page.
5. View the page source code and observe that the contents of the internal configuration file, including the master password `Support@110`, are exposed in the HTML.


**Remediation**  
Do not use user-supplied parameters as direct filesystem paths. Implement a server-side allowlist that maps approved theme identifiers to fixed filenames. Restrict file access to the intended directory and ensure the application process cannot read unnecessary sensitive files. Directory traversal filtering should be treated as a supplementary measure rather than the primary defence.

### 4.4 Broken Object Level Authorization (BOLA)
**Risk Rating:** MEDIUM  

**Description**  
The application's internal API, exposed through `/api.php`, accepts a user identifier as a request parameter. The endpoint does not adequately verify whether the authenticated user is authorised to access the requested profile. As a result, a user with lower privileges may retrieve information belonging to another account by modifying the identifier supplied in the request. This information was used in conjunction with the password obtained through Finding 4.3 to authenticate as the administrative user.


**Business Impact**  
This weakness allows unauthorised access to user profile information and may expose internal account details. During testing, the endpoint disclosed information associated with the privileged specialadmin account, including its email address. This information could assist an attacker in identifying privileged accounts and planning further attacks. The risk is assessed as Medium because the demonstrated disclosure facilitates further compromise, while the extent of additional impact depends on other vulnerabilities and the sensitivity of the exposed data.


**Reproduction Steps**  
1. Authenticate as the standard IT support user.
2. Navigate to the application's API viewing functionality.
3. Intercept the request sent to `/api.php`.
4. Modify the request to include a different user identifier, for example `GET /api.php?id=1`.
5. Forward the request and inspect the JSON response for unauthorised profile information.
6. Observe that the response returns the profile data and email address for the `specialadmin` account instead of the currently authenticated user's details.


**Remediation**
Implement server-side authorisation checks for every API endpoint that accesses user-specific resources. The application must verify that the authenticated user has permission to access the requested object before returning its data. User identifiers supplied by the client must not be treated as proof of authorisation. Apply least-privilege access controls and test all object-level access paths.

### 4.5 Lack of Rate Limiting
**Risk Rating:** MEDIUM  

**Description**
The application's login endpoint does not adequately restrict repeated authentication attempts. The absence of effective throttling or other automated attack protections allows repeated password guesses to be submitted within a short period.


**Business Impact**  
An attacker may automate password-guessing attempts against known accounts. When combined with weak or reused passwords, this can lead to account compromise and unauthorised access to application functionality. The finding is rated Medium because the weakness increases the likelihood of account compromise when combined with susceptible credentials.


**Reproduction Steps**  
1. Identify the application's login portal and target the known valid account `help@support.thm`.
2. Use an automated HTTP request tool (such as Burp Suite Intruder or Hydra) to submit rapid, repeated login attempts using a standard password wordlist.
3. Observe that the server processes all requests continuously without introducing throttling, CAPTCHA challenges, or temporary account lockouts.
4. Note that the tool successfully identifies the correct password, evidenced by the server returning a successful authentication response (e.g., HTTP 302 redirect to the dashboard), granting unauthorized access to the account.


**Remediation**  
Implement server-side rate limiting and monitoring for repeated authentication failures. Consider progressive delays, temporary restrictions and risk-based additional verification. Apply controls at appropriate account and network levels, and monitor suspicious login patterns. Ensure protective measures do not unnecessarily expose legitimate users to account-lockout denial-of-service attacks.

### 4.6 Unencrypted Communication (HTTP)
**Risk Rating:** LOW  

**Description**  
The web application communicates over HTTP without transport-layer encryption. Information exchanged between the browser and the application may therefore be transmitted in plaintext. Where sensitive data is sent over this connection, an attacker with an appropriate network position may be able to intercept or modify traffic.


**Business Impact**  
An attacker capable of intercepting network traffic may obtain login credentials, session cookies or internal support information transmitted without encryption. Captured credentials or session tokens could subsequently be used to impersonate legitimate users. The finding is rated Low because exploitation generally requires a suitable network position or another means of intercepting the communication.


**Reproduction Steps**
1. Navigate to the application and verify that the URL utilizes the unencrypted `http://` scheme.
2. Utilize a packet analyzer (e.g., Wireshark or tcpdump) to capture the network traffic generated during an active login attempt.
3. Observe that the captured HTTP POST request transmits the submitted username and password in cleartext over the network, making it visible to anyone intercepting the traffic.

**Remediation** 
Deploy a valid TLS certificate and configure the web server to serve the application exclusively over HTTPS. Redirect HTTP requests to HTTPS and ensure that credentials, session identifiers and sensitive application data are never transmitted over unencrypted connections. Configure HTTP Strict Transport Security (HSTS) after HTTPS is correctly deployed, and review cookie settings to ensure session cookies are marked Secure.

### 4.7 Information Disclosure — Email Address and Server Version
**Risk Rating:** INFORMATIONAL  

**Description**
The application's unauthenticated homepage exposes the support email address help@support.thm. Additionally, HTTP response headers disclose the web server software and version through the header Server: Apache/2.4.58.
These disclosures provide information about the application's support contact and underlying technology.


**Business Impact**  
The disclosed email address may provide a target for phishing or password-guessing attempts, while the server version may help an attacker identify potentially relevant vulnerabilities for further investigation.
Neither disclosure necessarily permits direct exploitation on its own. The finding is therefore classified as Informational under the defined grading criteria.


**Reproduction Steps**
1. Navigate to the unauthenticated homepage.
2. Observe the internal support email address `help@support.thm` exposed in plaintext within the page content.
3. Intercept an HTTP response from the application.
4. Observe the Server: `Apache/2.4.58` header in the raw HTTP response, exposing the exact web server software version.


**Remediation**
Review whether the internal support email address needs to be displayed publicly. Where appropriate, use a dedicated public contact address or contact form. Configure the web server to minimise unnecessary version information in response headers. For Apache, the ServerTokens Prod directive can reduce the detail exposed in the Server header. This should be treated as a defence-in-depth measure rather than a replacement for patching and maintaining the server.

## Conclusion
The assessment identified significant weaknesses in input handling, access control, session management, authentication protections, and transport security, culminating in a High overall risk to the Support Operations Platform.
The most significant findings were command injection and cookie forgery. Command injection enabled unauthorised operating system command execution, while cookie forgery allowed privilege escalation to the IT support interface. When combined with weaknesses in authentication and local file access, these vulnerabilities formed an attack chain that could enable an attacker to obtain administrative access and execute commands on the server.
Remediation should prioritise eliminating command injection, enforcing server-side authorisation, and implementing secure session management. Further improvements should address authentication rate limiting, restrictions on local file access, HTTPS enforcement, and unnecessary information disclosure to reduce the likelihood of similar attack chains.
