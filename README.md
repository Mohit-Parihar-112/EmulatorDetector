# EmulatorDetector
This PowerShell-based tool is designed to detect Android emulators running on a Windows PC, such as BlueStacks, MSI App Player, and Now.gg, even if they’ve been renamed or obfuscated.

🔍 Features:

Signature Verification: Detects applications by verifying digital signatures of running executables.

Process Name Matching: Identifies known emulator process names, including hidden background processes.

Executable Path Scanning: Detects emulators based on known installation paths, even if renamed.

PC Identification: Tags every detection with the local machine's name for remote monitoring.

Live Console Feedback: Shows real-time scanning details for each running process.

Logging: Writes all detections to a local log file at C:\ProgramData\EmulatorDetection\log.txt.

Discord Webhook Integration: Sends detailed alerts to a specified Discord channel for remote notifications.

📦 Use Cases:
- Anti-cheat systems

- Game protection environments

- Security auditing

- Software license enforcement

✅ Requirements:
- Windows 10 or 11

- PowerShell 5.1+

- Must be run as Administrator for full process visibility
