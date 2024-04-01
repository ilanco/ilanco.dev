+++
title = 'Sync Obsidian with Nextcloud on Android'
date = 2024-03-25T09:42:17+02:00
+++

### Backup

A simple solution to back up Obsidian notes to Nextcloud.

1. Install the Nextcloud App
1. In the Auto upload settings, add a custom folder and select the Obsidian Vault
1. Configure the remote folder (for example `/Backup/Obsidian)

### Sync

To sync two or more Obsidian installations, we can use the excellent plugin `remotely-save`. The plugin supports multiple backend cloud providers and WebDAV, which is supported by NextCloud.

1. Install the `remotely-sync` plugin from the official "community plugin list"
1. Enable the plugin and configure the Server Address, Username and Password for NextCloud
1. Check connectiviy and configure additional settings