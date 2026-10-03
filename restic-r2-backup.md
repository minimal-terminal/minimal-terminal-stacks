# Automated Offsite Encrypted Backups with Restic & Cloudflare R2

This blueprint guides you through setting up a robust, automated offsite backup solution for your critical homelab data using Restic for encrypted backups and Cloudflare R2 for S3-compatible object storage. This ensures your data is secure, versioned, and resilient against local failures.

## Prerequisites

*   A Linux server (e.g., Raspberry Pi, NAS, or any Debian/Ubuntu-based system) with `sudo` access.
*   A Cloudflare account with a Workers & Pages subscription (required for R2).
*   Basic familiarity with the Linux command line.

## Step 1: Set Up Cloudflare R2 Bucket

1.  **Log in to Cloudflare:** Go to your Cloudflare dashboard.
2.  **Navigate to R2:** Select 'R2' from the left-hand menu.
3.  **Create Bucket:** Click 'Create bucket', give it a meaningful name (e.g., `homelab-backups`), and choose your preferred region.
4.  **Create API Token:** Go to 'Manage R2 API Tokens' under the 'Overview' tab of your R2 dashboard. Click 'Create API Token'.
    *   Give it a descriptive name (e.g., `restic-backup-token`).
    *   Select 'Edit' permissions for the specific bucket you just created.
    *   Note down the `Access Key ID` and `Secret Access Key`. These are crucial and will only be shown once.

## Step 2: Install Restic on Your Server

1.  **Download Restic:** Choose the appropriate binary for your system from the [Restic releases page](https://github.com/restic/restic/releases).
    For most Linux systems (e.g., Raspberry Pi, Debian/Ubuntu x64):
    ```bash
    sudo apt update
    sudo apt install -y rsync # Restic dependency for some operations
    wget https://github.com/restic/restic/releases/download/v0.16.4/restic_0.16.4_linux_amd64.bz2 # Adjust version/arch as needed
    bzip2 -d restic_0.16.4_linux_amd64.bz2
    sudo mv restic_0.16.4_linux_amd64 /usr/local/bin/restic
    sudo chmod +x /usr/local/bin/restic
    ```
2.  **Verify Installation:**
    ```bash
    restic version
    ```

## Step 3: Configure Restic Repository

1.  **Set Environment Variables:** For security, it's best to use environment variables for R2 credentials and your Restic password. Create a file, e.g., `/etc/restic/env.sh` (ensure proper permissions to prevent unauthorized access).
    ```bash
    sudo mkdir -p /etc/restic
    sudo nano /etc/restic/env.sh
    ```
    Add the following content, replacing placeholders:
    ```bash
    #!/bin/bash
    export RESTIC_REPOSITORY="s3:https://<ACCOUNT_ID>.r2.cloudflarestorage.com/<BUCKET_NAME>"
    export AWS_ACCESS_KEY_ID="<YOUR_R2_ACCESS_KEY_ID>"
    export AWS_SECRET_ACCESS_KEY="<YOUR_R2_SECRET_ACCESS_KEY>"
    export RESTIC_PASSWORD="<YOUR_STRONG_RESTIC_PASSWORD>"
    # Optional: For logging verbosity
    export RESTIC_PROGRESS_FPS=0
    ```
    *   `<ACCOUNT_ID>`: Found in your Cloudflare R2 dashboard URL or API tokens page.
    *   `<BUCKET_NAME>`: The name of your R2 bucket (e.g., `homelab-backups`).
    *   `<YOUR_R2_ACCESS_KEY_ID>`: Your R2 Access Key ID.
    *   `<YOUR_R2_SECRET_ACCESS_KEY>`: Your R2 Secret Access Key.
    *   `<YOUR_STRONG_RESTIC_PASSWORD>`: **CRITICAL!** A very strong, unique password for Restic's encryption. DO NOT LOSE THIS.

    Set appropriate permissions:
    ```bash
    sudo chmod 600 /etc/restic/env.sh
    ```
2.  **Source Environment Variables:**
    ```bash
    source /etc/restic/env.sh
    ```
3.  **Initialize Restic Repository:**
    ```bash
    restic init
    ```
    You will be prompted to enter your `RESTIC_PASSWORD` (which should already be set via `env.sh`). This creates the encrypted repository in your R2 bucket.

## Step 4: Perform Your First Backup

1.  **Source Environment Variables (if not already):**
    ```bash
    source /etc/restic/env.sh
    ```
2.  **Run Backup Command:** Replace `/path/to/data` with the actual path(s) you want to back up.
    ```bash
    restic backup /path/to/data --verbose --tag homelab-daily
    ```
    *   `--verbose`: Provides detailed output.
    *   `--tag homelab-daily`: Adds a tag to the snapshot for easier management.

3.  **Verify Backup:**
    ```bash
    restic snapshots
    ```
    This will list all snapshots stored in your R2 repository.

## Step 5: Automate Backups with Systemd Timer

Using `systemd` timers is generally preferred over `cron` for more robust scheduling and logging.

1.  **Create Restic Service File:**
    ```bash
    sudo nano /etc/systemd/system/restic-backup.service
    ```
    ```ini
    [Unit]
    Description=Restic Backup Service
    Requires=restic-backup.timer

    [Service]
    Type=oneshot
    User=root # Or a dedicated backup user
    EnvironmentFile=/etc/restic/env.sh
    ExecStart=/usr/local/bin/restic backup /path/to/data --verbose --tag homelab-daily
    # Add more ExecStart lines for additional paths, or combine them
    # ExecStart=/usr/local/bin/restic backup /path/to/another/data --verbose --tag homelab-daily
    ExecStartPost=/usr/local/bin/restic forget --tag homelab-daily --prune --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --keep-yearly 1
    ```
    *   Replace `/path/to/data` with your actual data path(s).
    *   The `forget --prune` command ensures old snapshots are removed based on your retention policy (e.g., keep 7 daily, 4 weekly, 6 monthly, 1 yearly).

2.  **Create Restic Timer File:**
    ```bash
    sudo nano /etc/systemd/system/restic-backup.timer
    ```
    ```ini
    [Unit]
    Description=Daily Restic Backup Timer

    [Timer]
    OnCalendar=*-*-* 03:00:00 # Run daily at 3:00 AM
    Persistent=true

    [Install]
    WantedBy=timers.target
    ```

3.  **Enable and Start the Timer:**
    ```bash
    sudo systemctl enable restic-backup.timer
    sudo systemctl start restic-backup.timer
    ```
4.  **Check Timer Status:**
    ```bash
    systemctl list-timers | grep restic
    ```
    This will show when the next backup is scheduled.

## Step 6: Restoring Data (Important for Testing)

It's crucial to test your backup system by performing a restore.

1.  **Source Environment Variables:**
    ```bash
    source /etc/restic/env.sh
    ```
2.  **List Snapshots:** Identify the snapshot you want to restore.
    ```bash
    restic snapshots
    ```
3.  **Restore to a Temporary Location:**
    ```bash
    restic restore <SNAPSHOT_ID> --target /tmp/restic_restore_test
    ```
    Replace `<SNAPSHOT_ID>` with the ID from `restic snapshots`. The `--target` flag specifies where to restore the data.

## Conclusion

You've successfully set up an automated, encrypted offsite backup solution using Restic and Cloudflare R2. Regularly check your `systemd` journal (`journalctl -u restic-backup.service`) for backup logs and ensure your retention policies are meeting your needs. Your data is now significantly more secure!