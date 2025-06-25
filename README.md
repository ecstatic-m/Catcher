# Catcher
Catcher is a dual-purpose Python-based project aiming to fetch OS and various other information of local machine and target machine.
Catcher is a dual-purpose Python-based project developed as part of an Operating Systems course. It is designed to perform two types of scans:

Local Scan — Gathers system information about the machine where it's executed.

Remote (Global) Scan — Connects to a background server running on a target machine (once auto-installed) and retrieves system data from it, potentially useful for vulnerability assessments.

Disclaimer: This project mimics behavior similar to malware (auto-installation, persistence, remote control) for educational purposes only. Unauthorized deployment on machines without consent is unethical.

Features
LOCAL SCAN (Client-side GUI)
Fetches system information such as:

OS details

CPU info (cores, frequency, architecture)

Memory usage

Battery status

Peripheral devices (USBs, disks)

Distance Scan (Remote via IP)
Connects to a remote host where a stealth background server is running and retrieves:

CPU and OS info

Memory details

Battery status

Connected storage devices

USB/Peripheral devices

All data is displayed in a Tkinter-based GUI for the operator.

HOW IT WORKS :

1. C.py (Client/Controller)
Provides a graphical interface using tkinter.

Allows local or remote scan via buttons.

Sends commands to a server on a target machine via sockets.

Receives and parses JSON or binary responses.

2. S.py (Remote Server Agent)
Launches a background server that listens on port 12345.

Automatically adds itself to system startup:

Windows: Uses schtasks

Linux: Adds a @reboot cron job

macOS: Installs a launchd agent

Handles requests such as CPU_INFO, MEM_INFO, Devices, etc.

Returns data in compressed or JSON format to save bandwidth and reduce detection.

AS FOR Dependencies: 
  pip install psutil cpuinfo python-nmap pywin32

follow error messages in-case of forgotten installations

ON YOUR MACHINE (Operator/Client Side)
bash
Copy
Edit
python C.py
Run with administrator privileges.

Use the GUI to select local or global scan options.

ON TARGET MACHINE (Victim/Remote)
bash
Copy
Edit
python S.py
On first execution, it:

Sets itself to auto-launch on every boot.

Starts a background TCP server on port 12345.

This mimics the behavior of a remote agent/implant.

LIMITATIONS :

*  Only basic recon information, no exploitation modules.

*  Windows-only WMI code may fail on Linux/macOS if not checked.

*  No encryption/authentication in communication (security risk).

