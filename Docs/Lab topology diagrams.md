
=======================================================================
                     🕸️ THE CYBER DOJO TOPOLOGY v2.0 🕸️                
=======================================================================

                     [ 🥷 YOUR COMMAND CENTER ]
                    (Kali / Parrot OS / Ubuntu)
                      IP: 192.168.56.10 (Local)
                                  │
           ┌──────────────────────┴──────────────────────┐
           │                                             │
           ▼                                             ▼
[ 🛡️ THE INVISIBLE MOAT ]                    [ 🐳 THE DOCKER BASEMENT ]
(Host-Only Virtual Switch)                     (Localhost Environments)
(Isolated from the internet)                   (Runs directly on your OS)
           │                                             │
           ├──► [ 🤖 TARGET 1 ]                          ├──► [ 🏦 APP 1 ]
           │     Mr. Robot VM                            │     DVBank
           │                                             │
           ├──► [ 📦 TARGET 2 ]                          ├──► [ 💸 APP 2 ]
           │     Prime VM                                │     DVBLab
           │                                             │
           ├──► [ 🤪 TARGET 3 ]                          └──► [ 📱 APP 3 ]
           │     DerpNStink VM                                 DVB-Mobile
           │                                              (Bridged to emulator)
           └──► [ 🪟 TARGET 4 ]
                 PwnOS VM                         

=======================================================================


⚙️ Recommended Network Settings
To keep your environment safe and ensure your tools work correctly, you should configure your virtualization software (VirtualBox or VMware) using one of the following methods:

1. Host-Only Adapter (Recommended & Safest)

How it works: Creates a private, invisible network that only exists between your physical computer, your Kali VM, and your target VMs.

Why use it: The vulnerable machines cannot access the actual internet, and the rest of your home network (like your smart TV or family members' devices) cannot see the vulnerable machines.

Setup: Assign both your Attacker VM and your Target VM to the same vboxnet0 (VirtualBox) or VMnet1 (VMware) interface.

2. NAT Network (Alternative)

How it works: Puts your VMs on a shared private subnet that uses your physical computer's internet connection to reach the outside world.

Why use it: Sometimes useful if a specific vulnerable VM explicitly requires internet access to pull down a script or resource, but this is rare for eWPT prep.

Setup: Assign both VMs to a custom NAT Network (not just "NAT").

🐳 Docker Targets (like DVBLab)
For web applications running in Docker rather than a full VM, the topology is even simpler. You install Docker directly onto your Attacker Machine (e.g., Kali Linux). The target application binds to your local ports, meaning your "network" path is simply navigating to http://localhost:80 or http://127.0.0.1 in your web browser.
