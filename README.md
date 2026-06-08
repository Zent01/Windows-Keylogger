# Windows System Monitor Tool
⚠️ Educational Purpose Only
This code is a system monitoring and logging tool created for educational and research purposes. It demonstrates various Windows system programming concepts including:

Keyboard input monitoring

Screenshot capture

Microphone recording

Webcam image capture

Clipboard monitoring

Active window tracking

Email-based data exfiltration

Persistence mechanisms

Anti-analysis techniques

🔧 Technical Features
System Monitoring
Keylogging: Records keyboard input with window context

Screenshot Capture: Takes screenshots on mouse clicks (7-second cooldown)

Microphone Recording: 25-second audio capture triggered by Shift key

Webcam Capture: Takes photos on Tab key press with anti-flood protection

Clipboard Monitoring: Tracks copied content with 2-second throttling

Persistence
Copies itself to %APPDATA%\Microsoft\Windows\SysInternal\

Adds shortcut to Windows Startup folder

Creates Registry Run key (for .exe versions only)

Hides files using attrib +h

Anti-Analysis Evasion
Sandbox Detection: Checks for VM indicators, low RAM, low disk space, low process count

Process Monitoring: Looks for analysis tools (Task Manager, Process Hacker, Wireshark, antivirus)

Periodic Checks: Continuous scanning every 10-12 seconds

Data Exfiltration
Email Reports: Sends compressed logs (ZIP) via Gmail SMTP

⚙️ Trigger Mechanisms
Action	Trigger	Cooldown
Screenshot	Mouse left click	7 seconds
Webcam	Tab key	10 seconds
Microphone	Shift key	25 seconds
Email report	Enter key	7 seconds

Network Info: Captures ipconfig /all output

Storage Management: Auto-deletes oldest files when exceeding 1000MB
