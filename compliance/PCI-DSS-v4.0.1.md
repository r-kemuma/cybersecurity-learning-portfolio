# PCI DSS v4.0.1: Learning Summary

My learning notes after completing the **PCI DSS v4.0.1 course on Codebashing**, focusing on how the standard protects payment card data and what it means for secure software development.

![Course](https://img.shields.io/badge/Course-Completed-brightgreen)
![Standard](https://img.shields.io/badge/PCI%20DSS-v4.0.1-blue)
![Topic](https://img.shields.io/badge/Topic-Cybersecurity-orange)
![Learning](https://img.shields.io/badge/Focus-Payment%20Security-purple)

---

##  Contents

1. [What is PCI DSS?](#-what-is-pci-dss)
2. [The 6 Goals and 12 Requirements](#-the-6-goals-and-12-requirements)
3. [Scope: What Is In and What Is Out](#-scope-what-is-in-and-what-is-out)
4. [Cardholder Data: What You Can and Cannot Store](#-cardholder-data-what-you-can-and-cannot-store)
5. [Secure Software Under Requirement 6](#-secure-software-under-requirement-6)
6. [Key Changes in v4.x](#-key-changes-in-v4x)
7. [Key Takeaways](#-key-takeaways)
8. [References](#-references)

---

##  What Is PCI DSS?

The **Payment Card Industry Data Security Standard (PCI DSS)** is a set of security requirements designed to protect payment account data. It applies to organizations that store, process, or transmit payment account data, as well as systems that can affect the security of the cardholder data environment.

The standard is maintained by the **PCI Security Standards Council (PCI SSC)**. Version 4.0.1, published in June 2024, clarified and refined aspects of version 4.0 without introducing new requirements.

### How a card payment works

```mermaid
flowchart LR
    A["Cardholder"] -->|Card payment| B["Merchant"]
    B --> C["Payment Processor"]
    C --> D["Card Network"]
    D --> E["Issuing Bank"]

    style B fill:#ffe0b2,stroke:#e0a050,color:#222
    style C fill:#ffe0b2,stroke:#e0a050,color:#222
```

**Key lesson:** Organizations must understand how payment data flows through their systems and determine their PCI DSS responsibilities.

---

##  The 6 Goals and 12 Requirements

PCI DSS organizes its 12 principal requirements into six security goals.

```mermaid
mindmap
  root((PCI DSS v4.0.1))
    1. Build and Maintain Secure Networks
      R1 Network security controls
      R2 Secure configurations
    2. Protect Account Data
      R3 Protect stored account data
      R4 Protect data in transit
    3. Maintain a Vulnerability Management Program
      R5 Protect against malware
      R6 Develop and maintain secure systems
    4. Implement Strong Access Control Measures
      R7 Restrict access by business need
      R8 Identify and authenticate users
      R9 Restrict physical access
    5. Regularly Monitor and Test Networks
      R10 Log and monitor access
      R11 Test security regularly
    6. Maintain an Information Security Policy
      R12 Support security with policies and programs
```

### The 12 requirements in plain language

| # | Requirement | Practical meaning |
|---|---|---|
| 1 | Install and maintain network security controls | Use firewalls, network segmentation, and appropriate traffic controls. |
| 2 | Apply secure configurations | Remove default passwords and unnecessary services. |
| 3 | Protect stored account data | Minimize storage and render stored PAN unreadable. |
| 4 | Protect data in transit with strong cryptography | Protect account data transmitted over open, public networks. |
| 5 | Protect against malicious software | Prevent, detect, and address malware threats. |
| 6 | Develop and maintain secure systems and software | Apply secure coding, patching, and vulnerability management practices. |
| 7 | Restrict access by business need to know | Grant access according to job responsibilities and least privilege. |
| 8 | Identify users and authenticate access | Use unique user IDs, strong authentication, and MFA where required. |
| 9 | Restrict physical access | Protect facilities, payment devices, and physical media. |
| 10 | Log and monitor all access | Maintain audit trails and monitor suspicious activity. |
| 11 | Test security regularly | Conduct appropriate vulnerability scans, penetration tests, and change detection. |
| 12 | Support security with policies and programs | Establish governance, risk management, awareness, and third-party oversight. |

**Practical connection:** Access reviews, privileged access management, least privilege, segregation of duties, audit evidence, and remediation tracking all support important security control objectives.

---

##  Scope: What Is In and What Is Out?

The **Cardholder Data Environment (CDE)** consists of the people, processes, and technologies that store, process, or transmit cardholder data or sensitive authentication data.

Systems connected to the CDE, or that can affect its security, may also be in scope. Scope exclusions must be justified through appropriate controls and validation.

```mermaid
flowchart TB
    subgraph OUT["⬜ Potentially Out of Scope"]
        X["Corporate Email"]
        Y["HR Systems"]
    end

    subgraph CONNECTED["🟨 Connected-to or Security-Impacting Systems"]
        J["Jump Servers"]
        L["Logging and SIEM"]
        AD["Directory Services"]
    end

    subgraph CDE["🟥 Cardholder Data Environment"]
        P["Payment Application"]
        DB[("Card Database")]
    end

    J <--> P
    L <--> CDE
    AD <--> CDE
    P <--> DB
    OUT -. "Must be appropriately segmented and validated" .- CONNECTED

    style OUT fill:#eeeeee,stroke:#999,color:#222
    style CONNECTED fill:#fff2cc,stroke:#d6b656,color:#222
    style CDE fill:#f4cccc,stroke:#cc0000,color:#222
```

### How can organizations reduce scope?

- **Network segmentation:** Separate the CDE from other networks using properly implemented and validated controls.
- **Tokenization:** Replace PAN with a token to reduce the exposure of the original card number.
- **Payment outsourcing:** Use a qualified service provider where appropriate, while retaining responsibility for applicable merchant controls.
- **Scope validation:** Confirm the scope at least once every 12 months and after significant changes, as required.

 **Key lesson:** Reducing unnecessary exposure to cardholder data can reduce risk and compliance effort, but it does not automatically eliminate PCI DSS responsibilities.

---

##  Cardholder Data: What You Can and Cannot Store

Understanding the difference between cardholder data and sensitive authentication data is essential.

```mermaid
flowchart TD
    AD["Account Data"] --> CHD["Cardholder Data"]
    AD --> SAD["Sensitive Authentication Data"]

    CHD --> PAN["Primary Account Number (PAN)"]
    CHD --> NAME["Cardholder Name"]
    CHD --> EXP["Expiration Date"]
    CHD --> SC["Service Code"]

    SAD --> TRACK["Full Track Data"]
    SAD --> CVV["Card Verification Code"]
    SAD --> PIN["PIN or PIN Block"]

    PAN --> PROTECT["May be stored when permitted, with required protection"]
    TRACK --> NEVER["Never store after authorization"]
    CVV --> NEVER
    PIN --> NEVER

    style PROTECT fill:#c8e6c9,stroke:#4caf50,color:#222
    style NEVER fill:#ffcdd2,stroke:#e57373,color:#222
```

*Note: Cardholder name, expiration date, and service code are cardholder data when associated with a PAN. The PAN must be rendered unreadable wherever stored as required by PCI DSS. Sensitive authentication data must not be stored after authorization, even if encrypted.*

### Common data protection techniques

| Technique | What it does | Example |
|---|---|---|
| **Masking** | Conceals digits when displaying a PAN. | `4111 11•• •••• 1111` |
| **Truncation** | Permanently removes or omits portions of a PAN from stored or displayed data. | Retaining only permitted digits. |
| **Tokenization** | Replaces a PAN with a token that has no exploitable mathematical relationship to the original PAN. | `tok_8f3a...` |
| **Hashing** | Uses a cryptographic hash; PCI DSS specifies keyed cryptographic hashes for rendering PAN unreadable through hashing. | A suitable keyed hash. |
| **Encryption** | Transforms data into ciphertext using cryptographic keys. | Strong encryption with secure key management. |

 **Rule of thumb:** If you do not need the data, do not store it.

---

## Secure Software Under Requirement 6

Requirement 6 is especially relevant to secure software development, application security, and vulnerability management.

```mermaid
flowchart LR
    A["📝 Requirements and Threat Modeling"] --> B["👩‍💻 Secure Coding"]
    B --> C["🔍 Code Review and SAST"]
    C --> D["🧪 Security Testing and DAST"]
    D --> E["🚀 Controlled Release"]
    E --> F["🛡️ Production Protection"]
    F --> G["🩹 Patching and Monitoring"]
    G --> A
```

### Important secure development practices

| Area | What it means in practice |
|---|---|
| Secure development | Follow secure software development practices throughout the lifecycle. |
| Developer training | Train software personnel in security at least once every 12 months, as required. |
| Common attacks | Protect against injection, cross-site scripting, and broken access control. |
| Code review | Review custom code before release to production, using qualified personnel and the required independence. |
| Vulnerability management | Identify, risk-rank, and remediate vulnerabilities according to applicable requirements. |
| Software inventory | Maintain an inventory of bespoke and custom software and third-party components. |
| Public-facing web applications | Protect against attacks using an appropriate solution, such as a web application firewall, or other permitted methods. |
| Payment-page scripts | Authorize, assure the integrity of, and maintain an inventory of payment-page scripts; implement tamper detection as required. |

### Common attack patterns and defenses

| Attack | Risk | Defense |
|---|---|---|
| SQL injection | Unauthorized access to payment or customer data. | Parameterized queries and input validation. |
| Cross-site scripting (XSS) | Session theft or malicious page modification. | Context-aware output encoding and an appropriate Content Security Policy. |
| Broken access control | Users access data or functions they are not authorized to use. | Server-side authorization checks and least privilege. |
| Sensitive data in logs | PAN or other sensitive information is exposed. | Exclude sensitive data or apply required masking and protections. |
| Hardcoded secrets | Credentials and keys are exposed in source code. | Use a secrets manager and rotate exposed credentials. |
| Payment-page skimming | Malicious scripts steal payment data from the browser. | Script authorization, integrity checks, and tamper detection. |

### Example: SQL injection

** Vulnerable code**

```python
# User input is concatenated directly into the SQL query.
cursor.execute("SELECT * FROM cards WHERE user = '" + user + "'")
```

** Safer approach**

```python
# Use a parameterized query.
cursor.execute("SELECT * FROM cards WHERE user = %s", (user,))
```

Parameterized queries help prevent user input from being interpreted as SQL syntax. The exact parameter syntax depends on the database driver.

**Key lesson:** Security must be integrated into development, testing, deployment, and maintenance—not treated as a final check before release.

---

##  Key Changes in PCI DSS v4.x

```mermaid
timeline
    title PCI DSS Evolution
    2018 : PCI DSS v3.2.1
    Mar 2022 : PCI DSS v4.0 published
    Mar 2024 : PCI DSS v3.2.1 retired
    Jun 2024 : PCI DSS v4.0.1 published
    Mar 2025 : Future-dated v4.x requirements become effective
```

Important themes in version 4.x include:

- 🔄 **Customized approach:** Allows eligible requirements to be met through alternative controls that achieve the stated security objectives, with appropriate validation and documentation.
- 🔑 **Stronger authentication:** Strengthens authentication expectations, including MFA requirements for applicable access.
- 📊 **Targeted risk analysis:** Requires documented risk analysis to determine certain control frequencies where permitted.
- 🌐 **Payment-page protection:** Strengthens protections against unauthorized script changes and e-skimming.
- 👥 **Defined responsibilities:** Requires clear assignment of responsibilities for applicable security controls.
- 🔁 **Continuous security:** Emphasizes ongoing control operation, monitoring, and improvement rather than compliance as a once-a-year activity.

**Important date:** Future-dated requirements in PCI DSS v4.0 became effective on 31 March 2025. Version 4.0.1 did not restart that timeline.

---

## Key Takeaways

1. **PCI DSS is a security baseline.** Compliance is important, but organizations must also manage threats beyond the standard's minimum requirements.
2. **Scope matters.** Understand where payment data flows, which systems can affect its security, and how exposure can be reduced.
3. **Protect sensitive authentication data.** Never store it after authorization.
4. **Secure software throughout its lifecycle.** Apply secure coding, code review, testing, patching, and monitoring.
5. **Access control is fundamental.** Least privilege, appropriate authentication, and regular access reviews help prevent unauthorized access.
6. **Evidence matters.** Documented controls, assigned responsibilities, monitoring, and remediation support effective security governance.
7. **Compliance is continuous.** Controls must remain effective as applications, infrastructure, threats, and business processes change.

### My learning reflection

Completing the PCI DSS v4.0.1 course helped me connect payment card security requirements with practical cybersecurity concepts, including access governance, vulnerability management, secure software development, and audit readiness.

This is knowledge I can continue building on as I grow in information security and cybersecurity governance.

---

##  References

- [PCI Security Standards Council — Document Library](https://www.pcisecuritystandards.org/document_library/)
- [PCI Security Standards Council — Official Website](https://www.pcisecuritystandards.org/)
- Codebashing — PCI DSS v4.0.1 course.

> **Disclaimer:** These are personal learning notes written in my own words following course completion. They are intended for educational purposes and do not replace the official PCI DSS standard, professional assessment, or formal certification.

---

**Course completed:** 29 October 2026  
**Learning platform:** Codebashing  
**Topic:** Payment Card Security and Secure Software Development
