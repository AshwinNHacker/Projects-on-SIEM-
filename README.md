# Projects-on-SIEM-
Projects-on-SIEM/
│── README.md
│── src/
│   ├── log_collector.py  # Collect logs import os
import platform
import json
import time
import logging
from datetime import datetime
import subprocess

# Configure logging
logging.basicConfig(filename="log_collector.log", level=logging.INFO, 
                    format="%(asctime)s - %(levelname)s - %(message)s")

# Function to collect logs from Windows Event Viewer
def collect_windows_logs():
    logs = []
    try:
        command = "wevtutil qe System /c:10 /rd:true /f:json"
        result = subprocess.run(command, shell=True, capture_output=True, text=True)
        logs = json.loads(f'[{result.stdout.replace("}\n{", "},{")}]')  # Handling JSON format properly
        logging.info("Successfully collected Windows logs.")
    except Exception as e:
        logging.error(f"Error collecting Windows logs: {e}")
    return logs

# Function to collect logs from Linux syslog
def collect_linux_logs():
    logs = []
    try:
        log_path = "/var/log/syslog" if os.path.exists("/var/log/syslog") else "/var/log/messages"
        with open(log_path, "r") as file:
            lines = file.readlines()[-10:]  # Collect last 10 logs
            logs = [{"timestamp": str(datetime.now()), "log": line.strip()} for line in lines]
        logging.info("Successfully collected Linux logs.")
    except Exception as e:
        logging.error(f"Error collecting Linux logs: {e}")
    return logs

# Main function to collect logs based on OS
def collect_logs():
    os_type = platform.system()
    logging.info(f"Detecting OS: {os_type}")

    if os_type == "Windows":
        return collect_windows_logs()
    elif os_type == "Linux":
        return collect_linux_logs()
    else:
        logging.warning("Unsupported OS detected. No logs collected.")
        return []

if __name__ == "__main__":
    logs = collect_logs()
    log_file = "logs.json"

    with open(log_file, "w") as f:
        json.dump(logs, f, indent=4)

    print(f"Logs saved to {log_file}")
    logging.info(f"Logs saved to {log_file}")

│   ├── siem_dashboard.py  # Visualize logs
│   ├── alert_rules.json  # Detection rules
│   ├── threat_intel.py  # Integrate threat feeds
│── docs/
│   ├── installation.md  # How to install
│   ├── configuration.md  # SIEM setup guide
│── requirements.txt
│── LICENSE
