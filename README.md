# Overview

See https://github.com/NickWaterton/samsung-tv-ws-api for original README

See https://notes.schaberg.xyz/Homelab for context around this project

But, to summarize:

- I want to sync an immich album to a frame tv for use in art mode
- The `example/async_art_update_from_directory.py` script was promising
- My idea was to make a separate proxmox-style LXC container to run this script to sync a mounted/shared folder of photos to the frame tv
- But, immich can't natively put photos uploaded via the web UI somewhere that can be mounted/shared to a separate container
- So, I'm going to install this package and script into the immich LXC container directly and add a systemd unit service thing to start the script on boot and restart it if it fails
- This means the sync script needs to be able to sync across many folders (immich stores photos in /opt/immich/upload/library/<user>/<album> given my current storage template settings), so uploads from different users will end up in different folders

# Setup steps

Assumptions: Debian 13.3 (or similar) with Python 3.13.5 already installed.

**Prerequisites**

1. Install pip (if not present) and a virtualenv (recommended):
   ```bash
   sudo apt update
   sudo apt install -y python3-pip python3-venv
   ```

2. Clone this repo and install the `samsungtvws` package (and its dependencies) used by the script. From the repo root:
   ```bash
   cd /opt
   sudo git clone <this-repo-url> immich-frametv-art-sync
   cd immich-frametv-art-sync
   sudo pip3 install -e .
   ```
   Replace `<this-repo-url>` with your clone URL (e.g. the upstream `https://github.com/NickWaterton/samsung-tv-ws-api.git` or your fork).
   (Or use a venv and install with `pip install -e .` there; then point the systemd unit at that Python/interpreter if you prefer.)

3. Install Pillow so the script can sync art and match uploaded files (recommended):
   ```bash
   sudo pip3 install Pillow
   ```
   If Pillow is not installed, the script still runs but will not auto-sync artwork with the TV’s “My Photos” list.

**Install and enable the systemd service**

1. Copy the unit file and adjust paths if your repo is not in `/opt/immich-frametv-art-sync`:
   ```bash
   sudo cp /opt/immich-frametv-art-sync/immich-frametv-art-sync.service /etc/systemd/system/
   ```
   Edit `/etc/systemd/system/immich-frametv-art-sync.service` if needed (e.g. change `192.168.0.117` or the `--folder` glob).

2. Reload systemd, enable the service to start on boot, and start it:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable immich-frametv-art-sync.service
   sudo systemctl start immich-frametv-art-sync.service
   ```

3. Check status and logs:
   ```bash
   sudo systemctl status immich-frametv-art-sync.service
   journalctl -u immich-frametv-art-sync.service -f
   ```

The service runs `example/async_art_update_from_directory.py` with `--folder /opt/immich/upload/library/*/FrameTV` and TV IP `192.168.0.117`, and restarts on failure.


# Future TODOs

1. Auto-convert from HEIC to jpeg?