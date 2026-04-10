---
title: CAN Bus Security
published: true
---

# CAN Bus Security

## Introduction: The Unseen Threats in Connected Vehicles

As today’s vehicles transition into highly computerized and interconnected systems, securing their digital infrastructure has become a critical priority. At the core of in-vehicle communication lies the Controller Area Network (CAN) bus—a technology originally engineered to streamline how Electronic Control Units (ECUs) exchange data.

Although the CAN bus excels in reliability and real-time performance, it was never designed with modern cybersecurity challenges in mind. This absence of native security controls has made it an attractive entry point for attackers. Threat actors increasingly take advantage of these weaknesses, putting essential vehicle functions, passenger safety, and sensitive data at risk.

With connected features, over-the-air updates, and autonomous driving technologies becoming standard, the attack surface of vehicles continues to expand. As a result, identifying CAN bus vulnerabilities and deploying advanced defensive measures has shifted from being an optional enhancement to an absolute requirement for safeguarding the next generation of automotive technology.

## Why CAN Bus Security is Critical

Securing the CAN bus has become a pivotal concern in modern automotive engineering, as this communication protocol manages critical vehicle operations such as engine performance, braking, steering, infotainment, and advanced driver assistance systems. When attackers gain access to the CAN network, the consequences can be severe—from uncontrolled acceleration and brake disruption to the possibility of full remote vehicle takeover.

One of the most notable wake-up calls was the 2015 Jeep Cherokee incident, where researchers exposed the feasibility of remotely influencing vehicle behavior. Their findings highlighted that CAN bus exploitation is not a theoretical scenario but a genuine operational risk. Miller and Valasek’s research the same year reinforced this reality by showing how steering and braking functions could be altered through CAN-level attacks.

As vehicles increasingly integrate technologies like V2X communication, cloud-based fleet management, and OTA software updates, their exposure to cyber threats grows rapidly. Earlier studies, including work by Petit and Shladover (2014), revealed that wireless interfaces greatly expand the avenues through which attackers can reach and compromise CAN networks, rendering many previous security assumptions inadequate.

These developments emphasize the urgent need for stronger protection strategies to secure the communications backbone of today’s connected vehicles.

## CAN Bus Vulnerabilities

### Lack of Authentication and Encryption

One of the fundamental weaknesses of the CAN bus is its absence of authentication and encryption. Unlike modern communication systems, the CAN protocol allows any ECU connected to the network to send or receive data without verifying the identity or legitimacy of the sender. This design flaw opens the door for attackers—whether they gain access remotely or through a physical connection—to inject fraudulent messages capable of overriding essential vehicle functions.

> Research has shown that this vulnerability is more than theoretical. Woo and Kim (2015) demonstrated that incorporating message authentication codes (MACs) could significantly reduce spoofing attempts on the CAN bus. However, deploying such protections remains difficult due to the limited bandwidth and timing constraints inherent in the CAN protocol, making widespread adoption a complex challenge.

### Broadcast Communication Model

The CAN bus operates on a broadcast model in which every ECU connected to the network receives all messages transmitted across the bus. While this design simplifies communication, it also creates a significant security exposure: once a single ECU is compromised, an attacker can inject or alter messages and influence the behavior of other critical systems.

> Research by Checkoway et al. (2011) demonstrated how this weakness can be exploited in practice. Their work showed that breaching a seemingly low-risk component—such as the infotainment unit—can provide a pathway for attackers to move laterally through the network and interact with safety-critical modules. This interconnected nature means that even non-critical systems can become entry points for severe vehicle-wide manipulation.

### Denial-of-Service (DoS) Attacks

CAN bus systems are also highly susceptible to Denial-of-Service (DoS) attacks, where an attacker overwhelms the network by continuously sending high-priority frames. Because CAN prioritizes messages based on arbitration ID, this flood of dominant traffic can prevent legitimate signals from being transmitted, effectively crippling normal vehicle operations.

> Research by Choi et al. (2018) highlighted the severity of this threat. Their findings showed that sustained DoS conditions on the CAN bus could disrupt essential safety features—triggering failures in airbag deployment, interfering with braking functions, and causing erratic behavior in dashboard indicators. These results underscore how even a non-invasive attack can compromise the reliability of vital vehicle systems.

### Replay Attacks

Replay attacks pose a significant risk to CAN networks, as attackers can intercept legitimate messages and retransmit them at a later time. Since the CAN protocol lacks inherent mechanisms to verify message freshness, vehicles may accept these repeated messages as valid, potentially triggering unsafe actions or system malfunctions.

> Groll and Rieke (2019) suggested the use of cryptographic timestamps as a countermeasure to distinguish old messages from real-time commands. While this approach can enhance security, practical deployment is complicated by the limited computational resources of typical ECUs, making it a challenging solution for real-world automotive environments.

### Remote Exploits and Wireless Attack Surfaces

As modern vehicles increasingly incorporate wireless technologies such as Wi-Fi, Bluetooth, and cellular networks, the potential attack surface extends far beyond physical access points. These wireless interfaces provide new avenues for attackers to interact with a vehicle’s internal systems remotely.

> Research by Koscher et al. (2010) demonstrated that vulnerabilities in wireless connectivity could be exploited to inject malicious commands directly onto the CAN bus. This work highlighted the critical need for robust security at network gateways and reinforced the importance of protecting vehicles against remote exploitation.

<img width="1151" height="757" alt="image" src="https://github.com/user-attachments/assets/9c34834d-67d6-4c8c-87e4-0fccb983c9c5" />

## Securing the CAN Bus

### Intrusion Detection and Prevention Systems (IDPS)

#### 1. Signature-Based Intrusion Detection

Signature-based detection is a core approach used in Intrusion Detection and Prevention Systems (IDPS) to identify known cyber threats within automotive CAN networks. This technique relies on a repository of predefined attack signatures—distinct patterns associated with malicious behaviors such as unauthorized command injection, replay attacks, or spoofed ECU messages.

During operation, the system continuously monitors CAN traffic and compares each message against the stored signatures. Any match triggers an alert, allowing the system to detect and respond to suspicious activity in real-time. While effective against known threats, this method is limited when facing novel or evolving attacks.

#### 2. Deep Neural Network-Based Anomaly Detection

To address the inherent vulnerabilities of the CAN bus—such as the absence of authentication and encryption—advanced anomaly detection using Deep Neural Networks (DNNs) has emerged as a powerful solution. Recent research by Zhou et al. (2019) proposed a system leveraging DNNs to identify abnormal CAN messages in real time.

This approach treats anomaly detection as a cross-domain modeling problem. CAN bus messages are organized into three categories:

- **Anchor**: verified normal data
- **Positive**: regular operational data
- **Negative**: anomalous or malicious data

These groups are processed through a shared-weight DNN architecture, using an embedded triplet loss function (adapted from facial recognition systems) to optimize feature distances. The network is trained to minimize the distance between anchor and positive samples (normal behavior) while maximizing the distance between anchor and negative samples (anomalous behavior).

The DNN extracts detailed feature vectors from CAN messages, capturing critical behavioral patterns such as message frequency, sequence of message IDs, and payload content. In real-time deployment, this system continuously monitors incoming CAN traffic and detects deviations from the established baseline, enabling early identification of threats such as ransomware or command injection attacks. Once anomalies are detected, the IDPS can trigger automated mitigation measures, including network isolation or temporary ECU suspension.

#### 3. Network Segmentation for Enhanced Security

Complementing anomaly detection, network segmentation strengthens CAN bus resilience by isolating critical and non-critical vehicle systems. Safety-critical functions—such as braking, steering, and airbag control—are separated from non-critical systems like infotainment, telematics, and connected services. Gateway ECUs enforce strict communication rules between segments, limiting lateral movement of potential malware and preventing attacks from propagating across the vehicle’s network.

Together, these approaches—signature-based detection, deep learning-driven anomaly detection, and strategic network segmentation—provide a multi-layered defense framework, significantly enhancing the security and reliability of modern connected vehicles.

### Message Authentication and Encryption

Message authentication plays a crucial role in verifying that CAN bus messages originate from trusted sources and remain unmodified during transmission. Techniques such as Hash-Based Message Authentication Codes (HMAC) offer a dependable and resource-efficient method for validating message integrity.

> Research by Zhang et al. (2021) showed that pairing HMAC with the Tiny Encryption Algorithm (TEA) can deliver strong protection against unauthorized message alterations and replay attacks while keeping computational overhead low—an important requirement for real-time automotive systems.

Beyond software-based methods, hardware-assisted authentication significantly boosts security. Technologies like Physical Unclonable Functions (PUFs) generate cryptographic identifiers based on the unique physical properties of each ECU. These identifiers are extremely difficult to replicate, providing a robust defense against unauthorized device access and impersonation.

Encryption serves as a complementary layer to authentication by safeguarding message confidentiality. It ensures that only intended ECUs can interpret the transmitted data, preventing attackers from understanding or manipulating sensitive information. Due to the real-time constraints of in-vehicle networks, symmetric encryption techniques are preferred, as they offer fast processing and minimal resource consumption.

To meet automotive performance demands, specialized cryptographic frameworks have been developed. Lightweight Encryption and Authentication Protocols (LEAP), for example, are engineered specifically for embedded systems. LEAP employs streamlined, security-enhanced stream ciphers that deliver both encryption and authentication simultaneously, ensuring secure CAN communications without introducing excessive latency or burdening ECU resources.

### Secure Gateways and Network Segmentation

Network segmentation has become a vital defensive strategy for safeguarding automotive CAN systems from cyber intrusions. This technique divides the vehicle’s internal communication infrastructure into separate, isolated segments, each governed by its own security policies and access controls.

By limiting how these segments interact, segmentation reduces the attacker’s ability to move laterally through the vehicle’s network. Even if a non-critical segment is breached, this isolation ensures that essential systems—such as steering, braking, and other safety-critical components—remain protected and fully operational.

Modern implementations rely on specialized gateway ECUs that serve as security enforcement points. These gateways regulate all inter-segment communication, applying strict filtering rules and continuously monitoring traffic for suspicious patterns or unauthorized exchanges. When a potential threat is detected, the compromised segment can be immediately isolated, preventing the attack from spreading further.

This layered approach not only mitigates the impact of security breaches but also enables swift containment and recovery, preserving the vehicle’s safety, reliability, and overall operational stability.

### Secure Boot and Firmware Integrity Verification

Maintaining the authenticity and integrity of in-vehicle firmware is essential for safeguarding the CAN bus against malicious interference. Two foundational mechanisms—Secure Boot and Firmware Integrity Verification—serve as key defenses in preventing unauthorized or altered code from running on automotive ECUs.

**Secure Boot** establishes a trusted execution environment by verifying digital signatures during the startup sequence. Only firmware that has been cryptographically validated is allowed to load, ensuring attackers cannot introduce tampered or rogue software into the system.

Working alongside this, **Firmware Integrity Verification** continuously checks that ECU firmware remains unchanged throughout its lifecycle. By detecting unauthorized modifications or corruption, this mechanism preserves system reliability and prevents malicious code from influencing CAN communication.

Together, these safeguards form a critical layer of protection, ensuring that only trusted, authenticated firmware controls the vehicle’s essential functions and keeping CAN-based operations secure from cyber threats.

## Secure Boot: Establishing a Chain of Trust

Secure Boot is a foundational defense mechanism that ensures a vehicle’s embedded systems start only with software that is verified, trustworthy, and free from tampering. It creates a Chain of Trust (CoT) that begins with a hardware-anchored Root of Trust (RoT) and extends through every stage of the boot process.

At the core of this system is the Root of Trust, a permanent component stored in protected memory such as ROM. It contains the essential cryptographic keys and initial verification code used to authenticate the very first software module that loads during startup.

From this starting point, each stage validates the next using digital signatures:

1. **Bootloader Verification**: The RoT checks the integrity and authenticity of the bootloader using asymmetric cryptographic methods such as RSA or ECC. If the signature is correct, the bootloader runs; if not, the system halts to prevent malicious execution.

2. **Operating System and Application Validation**: Once the bootloader is verified, it takes on the role of the trusted entity. It then authenticates the operating system, followed by other software components and applications. Only those with valid digital signatures are permitted to load.

This layered, sequential verification ensures that every part of the startup process is authenticated before being allowed to execute. By preventing modified or unauthorized firmware from running, Secure Boot safeguards the integrity of the in-vehicle computing environment and plays a critical role in protecting the CAN network from cyber threats.

## Firmware Integrity Verification: Ensuring Continuous Trust

While Secure Boot protects the system during startup, Firmware Integrity Verification extends this protection throughout the vehicle’s operation. This mechanism continuously checks that firmware remains authentic and unaltered, ensuring persistent security across all ECU functions.

Key components of this process include:

- **Cryptographic Hashing**: A cryptographic hash value is generated from the ECU’s firmware and compared against a trusted reference hash. Any mismatch signals potential tampering, corruption, or unauthorized modification.

- **Digital Signature Validation**: Firmware updates are verified using digital signatures to confirm they come from legitimate sources and have not been modified during distribution or installation. Only properly signed updates are permitted to execute.

By detecting changes or anomalies in firmware at any point during the vehicle’s lifecycle, these techniques help maintain a trusted software environment and reinforce the protection of CAN-based communications.

## Implementation in Automotive Systems

Deploying Secure Boot and Firmware Integrity Verification within modern vehicles requires incorporating these protections directly into the Electronic Control Units (ECUs) responsible for managing critical and non-critical functions. Effective implementation depends on several key elements:

- **Hardware Capabilities**: ECUs must include dedicated hardware security modules (HSMs) or trusted execution environments capable of performing cryptographic operations. These components handle tasks such as key storage, signature verification, and hashing, which form the backbone of both Secure Boot and runtime integrity checks.

- **Supporting Software Framework**: The vehicle’s software architecture must be engineered to accommodate secure boot sequences and continuous integrity monitoring. These processes must operate efficiently so that security enhancements do not interfere with real-time automotive performance requirements.

Recent research involving the S32G274A vehicle network processor demonstrated a practical integration of post-quantum Secure Boot methods, underscoring the viability and growing importance of advanced firmware protection mechanisms in next-generation automotive platforms.
