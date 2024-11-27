#  Automated Backup Solution:

Write a script to automate the backup of a specified directory to a remote server or a cloud storage solution. The script should provide a report on the success or failure of the backup operation.

## Prerequisites
1. Install Required Python Modules:

```
pip install paramiko
```
2. Set up SSH Key Authentication:

Ensure the remote server allows passwordless SSH access using a private key for automation.

## Script Code

~~~
import os
import paramiko
import logging
from datetime import datetime

# Configuration
LOCAL_DIRECTORY = "/path/to/local/directory"  # Specify the directory to back up
REMOTE_DIRECTORY = "/path/to/remote/directory"  # Specify the destination directory on the remote server
REMOTE_SERVER = "remote.server.com"  # Remote server's address
REMOTE_PORT = 22  # SSH port
USERNAME = "your_username"  # Remote server username
PRIVATE_KEY_PATH = "/path/to/private/key"  # Path to your SSH private key

# Logging configuration
LOG_FILE = "backup_report.log"
logging.basicConfig(
    filename=LOG_FILE,
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
)

def backup_directory():
    """
Automates the backup of the local directory to a remote server.
    """
    try:
        # Check if the local directory exists
        if not os.path.exists(LOCAL_DIRECTORY):
            raise FileNotFoundError(f"Local directory '{LOCAL_DIRECTORY}' does not exist.")

        # Connect to the remote server using Paramiko
        private_key = paramiko.RSAKey.from_private_key_file(PRIVATE_KEY_PATH)
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh.connect(REMOTE_SERVER, port=REMOTE_PORT, username=USERNAME, pkey=private_key)

        # Create SFTP session
        sftp = ssh.open_sftp()

        # Ensure the remote directory exists
        try:
            sftp.stat(REMOTE_DIRECTORY)
        except FileNotFoundError:
            sftp.mkdir(REMOTE_DIRECTORY)

        # Backup files
        for root, dirs, files in os.walk(LOCAL_DIRECTORY):
            remote_path = os.path.join(REMOTE_DIRECTORY, os.path.relpath(root, LOCAL_DIRECTORY)).replace("\\", "/")
            try:
                sftp.stat(remote_path)
            except FileNotFoundError:
                sftp.mkdir(remote_path)

            for file in files:
                local_file = os.path.join(root, file)
                remote_file = os.path.join(remote_path, file).replace("\\", "/")
                sftp.put(local_file, remote_file)
                logging.info(f"Backed up: {local_file} -> {remote_file}")

        sftp.close()
        ssh.close()

        # Report success
        logging.info("Backup operation completed successfully.")
        print("Backup operation completed successfully.")

    except Exception as e:
        # Report failure
        error_message = f"Backup operation failed: {e}"
        logging.error(error_message)
        print(error_message)

if __name__ == "__main__":
    print(f"Backup started at {datetime.now()}")
    logging.info("Backup started")
    backup_directory()
    print(f"Backup completed at {datetime.now()}")

~~~

## How to use?

1. Update the script with the paths and credentials for your local and remote environments.

Run the script:
~~~
python backup_script.py
~~~


## What to expect?


## Enhancements (Optional):

1. Add support for cloud storage (e.g., AWS S3, Google Drive) by integrating appropriate APIs (e.g., boto3 for AWS).
2. Schedule regular backups using a cron job:

~~~
crontab -e
~~~

Add an entry like this for daily backups at midnight:
~~~
0 0 * * * python /path/to/backup_script.py
~~~