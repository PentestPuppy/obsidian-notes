# DragonDrain on OWE
Denial-of-service conditions can emerge from the high computational cost of modern authentication handshakes. OWE follows a similar cryptographic model and therefore inherits comparable resource exhaustion risks.

In OWE, these risks appear through DragonDrain, a denial-of-service attack that targets the protocol’s Diffie-Hellman key exchange. By repeatedly triggering expensive cryptographic operations during association, an attacker can degrade access point performance and disrupt client connectivity.
## How DragonDrain Works
DragonDrain exploits the imbalance between the minimal effort required to send association requests and the significant CPU resources the access point must spend processing them. Over time, this can lead to delayed associations, dropped clients, or a temporary denial of service, without compromising encryption or key material.

This behavior was first documented in the Dragonblood research and primarily affects older or unpatched OWE and WPA3 implementations that lack effective rate limiting or anti-clogging defenses.
### Tools
##### dragondrain-and-time
**NOTE**: This tool is specifically designed for SAE/Dragonfly. Use with OWE depends heavily on the AP implementation.

The github project: [vanhoefm/dragondrain-and-time](https://github.com/vanhoefm/dragondrain-and-time) is the main repository that provides proof-of-concept code for both DragonDrain and a related Dragontime test suite. It contains utilities to generate handshake frames that trigger the expensive parts of the Dragonfly process on an access point. Usage instructions show how to set up your wireless interface in monitor mode and run the tool against a target AP to test for vulnerability.

**Note**: This tool has only been tested with Atheros ath9k_htc–based wireless adapters. Other chipsets may fail to trigger the intended cryptographic workload on the access point or may not function reliably.  
  
The ath_masker kernel module is required when using ath9k_htc adapters. It enables controlled MAC address masking and proper handling of transmitted management frames, which is necessary for reliably eliciting handshake responses from the target access point during repeated Dragonfly or OWE association attempts.

A simple attack requires only a wireless interface, the target access point BSSID, and the operating channel:
```
cd /root/tools/dragondrain-and-time
./src/dragondrain -d wlan2 -a AA:BB:CC:DD:EE:FF -c 10
```
This minimal invocation performs a continuous stream of OWE association attempts.
- -d wlan2: Specifies the wireless interface used for transmission. The interface must support monitor mode and frame injection, as the tool crafts and injects raw 802.11 management frames.
- -a AA:BB:CC:DD:EE:FF: Defines the target access point BSSID. This is the MAC address of the AP that will be forced to process repeated Dragonfly handshakes.
- -c 10: Sets the Wi-Fi channel on which the attack is executed. The interface must be tuned to the same channel as the target AP, otherwise frames will not be processed.
###### Advanced Execution Option:
```
./src/dragondrain -d wlan1 -a AA:BB:CC:DD:EE:FF -b 54 -n 20 -c 10 -i 3000 -f
```
This configuration increases attack intensity and realism by enabling additional parameters.
- -d wlan1: Selects a different wireless interface, useful when testing with multiple radios or separating monitoring and injection roles.
- -a AA:BB:CC:DD:EE:FF: Target BSSID, identical in function to the basic example.
- -b 54: Controls the transmission rate used for injected frames. Higher rates allow the attacker to send more handshake requests per second, increasing pressure on the access point.
- -n 20: Sets the number of parallel handshake attempts. This simulates multiple clients attempting to associate simultaneously, amplifying the computational load on the AP.
- -c 10: Wi-Fi channel used for the attack.
- -i 3000: Defines the interval between handshake attempts in milliseconds. Tuning this value allows the attacker to balance sustained pressure versus burst-style exhaustion.
- -f: Enables continuous or forced execution mode, ensuring that handshake attempts persist even if responses are not received. This maximizes the likelihood of CPU exhaustion on vulnerable devices.