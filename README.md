# Beam CLI Application <!-- omit in toc -->

Beam is a secure file sharing application that enables private networks between organizations for exchanging large volumes of data—such as for AI training, media assets, or research datasets. It leverages the BitTorrent protocol for fast, scalable transfers and provides end-to-end encryption to keep your data safe and confidential.

## Table of Contents <!-- omit in toc -->

- [1. Installation](#1-installation)
- [2. User configuration](#2-user-configuration)
  - [Prerequisites](#prerequisites)
  - [Configuration Process](#configuration-process)
- [3. Configuring the daemon service](#3-configuring-the-daemon-service)
  - [Running in a terminal window](#running-in-a-terminal-window)
  - [Running as a Service](#running-as-a-service)
    - [Windows (using NSSM - Non-Sucking Service Manager)](#windows-using-nssm---non-sucking-service-manager)
    - [Linux (systemd)](#linux-systemd)
    - [macOS (launchd)](#macos-launchd)
- [4. File sharing](#4-file-sharing)
  - [4.1 Torrent generation](#41-torrent-generation)
  - [4.2 Torrent sharing and management](#42-torrent-sharing-and-management)
    - [Revoke a torrent](#revoke-a-torrent)
    - [Info](#info)
    - [Stop a torrent](#stop-a-torrent)
    - [Add a torrent](#add-a-torrent)
  - [4.3 Obtaining the shared files](#43-obtaining-the-shared-files)
- [Troubleshooting](#troubleshooting)
  - [Common Issues](#common-issues)
  - [Logs](#logs)
- [Security Considerations](#security-considerations)
- [Commands](#commands)
  - [Installation Command](#installation-command)
  - [Status](#status)
  - [Torrent Management](#torrent-management)
    - [Start](#start)
    - [Create](#create)
    - [Share](#share)
    - [Fetch](#fetch)
    - [List](#list)
    - [Add](#add)
    - [Revoke](#revoke)
    - [Stop](#stop)
    - [Info](#info-1)
    - [Diagnose](#diagnose)
- [Roadmap](#roadmap)
- [References](#references)

## 1. Installation

1. Download the Beam binary for your platform from the [releases](./releases) page and rename is to "beam" (UNIX) or "beam.exe" (Windows).
   1. UNIX: make the binary executable with
      ```bash
      chmod +x beam
      ```   
2. Place the binary in a location included in your system's PATH.
   1. On Windows add the location containing the binary to the PATH manually.
   2. On Unix navigate to the location with the binary and use
        ```bash
        sudo mv beam /usr/local/bin/
        ```
3. Verify installation by running:

```bash
beam --version
```

## 2. User configuration

Before using Beam, you need to complete the installation process to set up your encryption keys and user configuration.

### Prerequisites

- A GitHub account (required for key exchange)
- Available network ports
- Write permissions to your download directory

### Configuration Process

Run the installation command to begin setup:

```bash
beam install
```

The installation wizard will guide you through these steps:

1. **Port Availability Check**: Confirms required network ports are available.
2. **Download Location**: Specify where downloaded files will be stored.
    - Windows example: `C:\\Users\\JohnDoe\\Downloads\\beam-files`
    - Linux/Mac example: `./beam-files`
3. **Key Generation (automatic)**: Generates your public/private key pair for secure communication.
4. **GitHub Configuration**:
    - Create a GitHub account if you don't have one.
    - Create a public repository named `gid`.
    - Create a file named `gid.pem` in the main branch.
    - Copy your public key from the installer and paste it into this file.
    - Enter your GitHub username to verify the configuration.
5. **Configuration Saving (automatic)**: Your settings are saved for future use.

After completing all steps, verify your setup with:

```bash
beam status
```

## 3. Configuring the daemon service

Before using any file-sharing commands, you need to start the Beam daemon in the terminal or set it up as a background service.

### Running in a terminal window

In a new terminal window, navigate to the beam directory and start the torrent daemon:

```bash
beam torrent start
```

You can also specify a custom port with:

```bash
beam torrent start --port 60001
```

This daemon must be running for most commands to work.

Alternatively, you can use screen or tmux to keep it running in the background:

```bash
tmux new -s beam "beam torrent start"
```

You can verify the daemon is running with:

```bash
beam status
```

**You can now proceed to Section 4.**


### Running as a Service

#### Windows (using NSSM - Non-Sucking Service Manager)

1. Download [NSSM](https://nssm.cc/)
2. Open Command Prompt as an Administrator
3. Install the service:

```sh
nssm.exe install BeamDaemon "C:\path\to\beam.exe" "torrent start"
nssm.exe start BeamDaemon
```

#### Linux (systemd)

```bash
# Create a systemd service file
sudo nano /etc/systemd/system/beam-daemon.service

# Add the following content:
[Unit]
Description=Beam Torrent Daemon
After=network.target

[Service]
Type=simple
User=YOUR_USERNAME
ExecStart=/path/to/beam torrent start
Restart=on-failure

[Install]
WantedBy=multi-user.target

# Enable and start the service
sudo systemctl enable beam-daemon
sudo systemctl start beam-daemon
```

#### macOS (launchd)

Create a file at `~/Library/LaunchAgents/com.beam.daemon.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.beam.daemon</string>
    <key>ProgramArguments</key>
    <array>
        <string>/path/to/beam</string>
        <string>torrent</string>
        <string>start</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
```

Then load the service:

```bash
launchctl load ~/Library/LaunchAgents/com.beam.daemon.plist
```

You can verify the daemon is running with:

```bash
beam status
```

You can now proceed to section 4.


## 4. File sharing
With the daemon running, you’re ready to exchange files using encrypted torrents.
To view torrents shared with you, run:

```bash
beam torrent list
```

To view torrents you’ve shared with others, run:

```bash
beam torrent list --sent
```

At this point, both torrent lists are likely empty. Let’s generate and share a .torrent file as a test (you can even share it with yourself).

### 4.1 Torrent generation

To generate a .torrent file from a local file or folder, run:

```bash
beam torrent create --file filepath
```

Replace `filepath` with the full path to your file or folder (`folderpath`):

```bash
# Windows
filepath = "C:\Users\JohnDoe\Documents\example.pdf"

# Unix
filepath = "/Users/johndoe/Documents/example.pdf"
```

The resulting .torrent file will be saved by default to:

```bash
# Windows
" C:\Users\JohnDoe\.beam\torrents\ "

# Unix
" /Users/johndoe/.beam/torrents/ "
```

You can also include a comment:

```bash
# Windows
beam torrent create --file filepath --comment "My comment"

# Unix
beam torrent create --file filepath --comment "My comment"
```

### 4.2 Torrent sharing and management

To share a torrent with a GitHub user run

```bash
beam torrent share --torrent torrent-path --users bill,bob --label "my-label"
```

Replace torrent-path with the full path to your .torrent file:

```bash
# Windows
"C:\Users\JohnDoe\.beam\torrents\example.pdf.torrent"

# Unix
"/Users/johndoe/.beam/torrents/example.pdf.torrent" 
```

Note that by default, the --seed flag is enabled—so once the torrent is shared with the user(s), the file begins sharing automatically.

The `--users` flag specifies the GitHub usernames of the recipients (in this case, bill and bob). The file is encrypted so that only these users—identified by the public keys stored in their `gid.pem` files within their GitHub gid repositories—can decrypt and access it.

The `--label` flag lets you add a short description of the shared content. This label appears in your torrent list, making it easier to identify each item later.

You can now view the list of torrents you’ve shared using `beam torrent list --sent`. If you shared the torrent with yourself, it should also appear under `beam torrent list`.

#### Revoke a torrent

To revoke access to content from a shared torrent—making it unavailable to the recipient—run:

```bash
beam torrent revoke
```

This command opens an interactive list of torrents you’ve shared, allowing you to select which one to revoke.

Alternatively, if you already know the torrent ID, you can revoke it directly:

```bash
beam torrent revoke --id abc123def456
```

#### Info

To check if the daemon is running and view active torrents along with their status, use:

```bash
beam torrent info
```

This command provides information such as download/upload progress, seeding status, and torrent IDs.

#### Stop a torrent 

To stop any ongoing download or seeding activity for a torrent, run:

```bash
beam torrent stop
```

You’ll see a list of active torrents and can choose which one to stop. This removes the torrent from the daemon.

If you know the torrent ID (get it from `beam torrent info`), you can stop it directly:

```bash
beam torrent stop --id abc123def456
```

#### Add a torrent 

To add a .torrent file (back) to the daemon for downloading or seeding, run:

```bash
beam torrent add --torent filepath
```

Replace filepath with the full path to your .torrent file:

```bash
# Windows
filepath = "C:\Users\JohnDoe\Downloads\received.torrent"

# Unix
filepath = "/Users/johndoe/Downloads/received.torrent"
```


### 4.3 Obtaining the shared files

If a torrent has been shared with you and appears in your list of torrents, you can access it with 

```bash
beam torrent fetch
```

This command lists torrents shared with you and lets you select which ones to download. The chosen .torrent file will be saved to your configured download folder (see Step 2 in User configuration), and automatically added to the daemon to begin the file transfer.
The content will start downloading right away to your designated folder.

For torrent management, refer to the earlier sections on checking torrent status, stopping downloads, and re-adding torrent files to the daemon.

## Troubleshooting

### Common Issues

1. **Port unavailable**: Make sure no other applications are using the required BitTorrent ports.
2. **GitHub key verification failed**: Ensure your public key was correctly uploaded to your GitHub 'gid' repository.
3. **Daemon not running**: Many commands require the daemon to be running first. Start it with `beam torrent start`.
4. **Windows path issues**: Remember to use double backslashes in paths (e.g., "C:\\Users\\JohnDoe\\Downloads") or use forward slashes instead (e.g., "C:/Users/JohnDoe/Downloads").

### Logs

Log files are stored in the following locations and can be helpful for diagnosing issues:

- Windows: `C:\Users\YourUsername\.beam\logs\`
- Linux/Mac: `~/.beam/logs/`

## Security Considerations

Beam uses RSA encryption to secure your content. Your private key never leaves your machine, and only users with the
correct private key can decrypt content shared with them.

If your private key is compromised, your GitHub ID can be misused. In the case of compromise, replace your GitHub ID key.

## Commands
Below is an overview of all beam commands. 
### Installation Command

```bash
beam install
```

Runs the setup wizard to configure Beam. This is the first command you should run after downloading the application.

### Status

```bash
beam status
```

Checks the status of your Beam installation, including:

- BitTorrent port availability
- Configuration validation
- Encryption key verification

This helps troubleshoot issues with your installation.

### Torrent Management

#### Start

```bash
beam torrent start
```

Starts the Beam torrent client as a daemon process to manage downloads and seeding operations.

You can also specify a custom port:

```bash
beam torrent start --port 60001
```

#### Create

```bash
# Windows
beam torrent create --file "C:\Users\JohnDoe\Documents\example.pdf"

# Unix
beam torrent create --file "/Users/johndoe/Documents/example.pdf"
```

Creates a .torrent file from local content that you want to share. By default, the `--seed` flag is enabled, which automatically starts sharing this content.

You can add an optional comment to the torrent:

```bash
# Windows
beam torrent create --file "C:\Users\JohnDoe\Documents\example.pdf" --comment "Project documentation"

# Unix
beam torrent create --file "/Users/johndoe/Documents/example.pdf" --comment "Project documentation"
```

#### Share

```bash
# Windows
beam torrent share --torrent "C:\Users\JohnDoe\.beam\torrents\example.pdf.torrent" --users bill,bob --label "documentation"

# Unix
beam torrent share --torrent "/Users/johndoe/.beam/torrents/example.pdf.torrent" --users bill,bob --label "documentation"
```

Encrypts a .torrent file for a specific GitHub user and shares it through the Beam platform. Only the target user with
the matching private key can decrypt and access this content.

Parameters:

- `--torrent`: Path to the .torrent file
- `--users`: GitHub usernames of the recipient separated with ,
- `--label`: Category or description of the content being shared

#### Fetch

```bash
beam torrent fetch
```

Lists torrents that have been shared with you and allows you to select which ones to download.

#### List

```bash
beam torrent list
```

Shows torrents shared with you.

To see torrents you have shared with others:

```bash
beam torrent list --sent
```

#### Add

```bash
# Windows
beam torrent add --torrent "C:\Users\JohnDoe\Downloads\received.torrent"

# Unix
beam torrent add --torrent "/Users/johndoe/Downloads/received.torrent"
```

Adds a .torrent file to the daemon for downloading or seeding. The daemon must be running first.

#### Revoke

```bash
beam torrent revoke
```

Revokes a shared torrent file from the server, making it unavailable to the recipient. This command shows an interactive list of torrents you've shared.

If you know the torrent ID, you can specify it directly:

```bash
beam torrent revoke --id abc123def456
```

#### Stop

```bash
beam torrent stop
```

Stops a torrent from seeding or downloading. This command shows an interactive list of active torrents to choose which one to stop.

If you know the torrent ID (get it from `beam torrent info`), you can specify it directly:

```bash
beam torrent stop --id abc123def456
```

#### Info

```bash
beam torrent info
```

Verifies if the daemon is running and lists active torrents, providing status information.

#### Diagnose

```bash
beam torrent diagnose --id abc123def456
```

Provides detailed diagnostic information about a specific torrent, including tracker status, peer connections, and
torrent configuration.

You can get the torrent ID from `beam torrent info`.


## Roadmap

- Add support for end-to-end data encryption using GID and AGE
- Add rsync capabilities
- Add P2P (relayed) exchanged

## References

- [GitHub ID (GID)](https://github.com/MyNextID/git-id)
- [AGE file encryption](https://filippo.io/age)
