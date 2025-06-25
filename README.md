# Catcher
Catcher is a dual-purpose Python-based project aiming to fetch OS and various other information of local machine and target machine.
Catcher is a dual-purpose Python-based project developed as part of an Operating Systems course. It is designed to perform two types of scans:

Local Scan — Gathers system information about the machine where it's executed.

Remote (Global) Scan — Connects to a background server running on a target machine (once auto-installed) and retrieves system data from it, potentially useful for vulnerability assessments.

⚠️ Disclaimer: This project mimics behavior similar to malware (auto-installation, persistence, remote control) for educational purposes only. Unauthorized deployment on machines without consent is illegal and unethical.

Features
🖥️ Local Scan (Client-side GUI)
Fetches system information such as:

OS details

CPU info (cores, frequency, architecture)

Memory usage

Battery status

Peripheral devices (USBs, disks)

🌐 Global Scan (Remote via IP)
Connects to a remote host where a stealth background server is running and retrieves:

CPU and OS info

Memory details

Battery status

Connected storage devices

USB/Peripheral devices

All data is displayed in a Tkinter-based GUI for the operator.

How It Works
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
