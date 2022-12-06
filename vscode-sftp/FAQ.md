- [Error: Failure](#error-failure)
	- [Error: Failure - Solution One](#error-failure---solution-one)
	- [Error: Failure - Solution Two](#error-failure---solution-two)
- [Error: Connection closed](#error-connection-closed)
- [Error: Clicking Upload Changed Files does not work](#error-clicking-upload-changed-files-does-not-work)
- [ENFILE: file table overflow ...](#enfile-file-table-overflow-)
	- [ENFILE: file table overflow ... - Solution for MacOS harsh limit](#enfile-file-table-overflow----solution-for-macos-harsh-limit)
- [How do I upload content inside a folder, but not the folder itself?](#how-do-i-upload-content-inside-a-folder-but-not-the-folder-itself)
- [How can I upload files as root?](#how-can-i-upload-files-as-root)
- [Automatically sync both ways without user interaction](#automatically-sync-both-ways-without-user-interaction)
- [Show dotfiles/hidden files in remote explorer](#show-dotfileshidden-files-in-remote-explorer)

## Error: Failure

The failure error message comes from the remote side and is more or less the default/generic error 
message that sftp server sends when a syscall fails or something similar happens.
To know what exactly is going wrong you could try to enable debug output for the sftp server 
and then execute your transfers again and see what (if anything) shows up in the logs there.

### Error: Failure - Solution One

Change `remotePath` to the actual path if it's a symlink.

### Error: Failure - Solution Two

The problem could be that your server runs out of file descriptors.
You should try to increase the file descriptors limit.
If you don't have the permission to do this, set [limitOpenFilesOnRemote](https://github.com/Natizyskunk/vscode-sftp/wiki/Configuration#limitopenfilesonremote) option in your config.

## Error: Connection closed

The problem could be that the SFTP extension keeps closing the connection for those who use more legacy/old systems.
You'll have to Explicitly override the default transport layer algorithms used for the connection to remove the new `"diffie-hellman-group-exchange-sha256"` algorithm that cause the problem from the `kex` section. Just add this in your `sftp.json` configuration file, which should make it work.
```json
{
	"algorithms": {
		"kex": [
			"ecdh-sha2-nistp256", 
			"ecdh-sha2-nistp384", 
			"ecdh-sha2-nistp521"
		],
		"cipher": [
			"aes128-gcm",
			"aes128-gcm@openssh.com",
			"aes256-gcm",
			"aes256-gcm@openssh.com",
			"aes128-cbc",
			"aes192-cbc",
			"aes256-cbc",
			"aes128-ctr",
			"aes192-ctr",
			"aes256-ctr"
		],
		"serverHostKey": [
			"ssh-rsa", 
			"ssh-dss",
			"ssh-ed25519",
			"ecdsa-sha2-nistp256", 
			"ecdsa-sha2-nistp384", 
			"ecdsa-sha2-nistp521",
			"rsa-sha2-256",
			"rsa-sha2-512"
		],
		"hmac": [
			"hmac-sha2-256", 
			"hmac-sha2-512"
		]
	}
}
```

## Error: Clicking Upload Changed Files does not work

See [vscode-sftp issue #854](https://github.com/liximomo/vscode-sftp/issues/854).

**@PaPa31** added a fix to make the 'Upload Changed Files' command visible and added a default keyboard shortcut to call it.
<!-- **danieleiobbi** has a workaround to create a keyboard shortcut. -->

![upload changed files keyboard shortcut](assets/faq/upload_changed_files_shortcut.png)

## ENFILE: file table overflow ...

MacOS have a harsh limit on number of open files.

### ENFILE: file table overflow ... - Solution for MacOS harsh limit

Run those command:
```sh
echo kern.maxfiles=65536 | sudo tee -a /etc/sysctl.conf
echo kern.maxfilesperproc=65536 | sudo tee -a /etc/sysctl.conf
sudo sysctl -w kern.maxfiles=65536
sudo sysctl -w kern.maxfilesperproc=65536
ulimit -n 65536
```

## How do I upload content inside a folder, but not the folder itself?

See [vscode-sftp issue #852](https://github.com/liximomo/vscode-sftp/issues/852).

As quoted from **raoul2000**, "as long as you set the `context` property to `./[path]` (e.g., `./build`), it
will work."

Example configuration (where all JS and HTML files in `./build` will be copied to `/folder1/folder2/folder3`):
```json
{
  "name": "My Server",
  "host": "<host_ip_address>",
  "protocol": "sftp",
  "port": 22,
  "username": "user1",
  "remotePath": "/folder1/folder2/folder3",
  "context": "./build",
  "uploadOnSave": false,
  "watcher": {
    "files": "*.{js,html}",
    "autoUpload": true,
    "autoDelete": false
  }
}
```

## How can I upload files as root?

See [vscode-sftp issue #559](https://github.com/liximomo/vscode-sftp/issues/559).

**Yevhen-development** has a workaround, but it may not work for everyone.  In `sftp.json`, set the
following:
```json
"sshCustomParams": "sudo su -;"
```

## Automatically sync both ways without user interaction

See [vscode-sftp issue #136](https://github.com/Natizyskunk/vscode-sftp/issues/136).

> *This can also be used with **GIT** this way when you're checking out a branch or reverting changes/commits, your server will also be updated.*

```json
{
  "name": "My Server",
  "host": "<host_ip_address>",
  "protocol": "sftp",
  "port": 22,
  "username": "user1",
  "remotePath": "/folder1/folder2/folder3",
  "uploadOnSave": false, // Set to false if watcher `autoUpload` is set to true & `files` is set to "**/*".
  "watcher": {
    "files": "**/*",
    "autoUpload": true,
    "autoDelete": true
  }
  "syncOption": {
    "delete": true // Delete extraneous files from destination directories.
  },
}
```

## Show dotfiles/hidden files in remote explorer

### If using proftpd

Please edit the config file `proftpd.conf`. Depending on your installation, the default location for this file can be one of those :
- `/etc/proftpd.conf`
- `/etc/proftpd/proftpd.conf`
- `/usr/local/etc/proftpd.conf`
- `/usr/local/etc/proftpd/proftpd.conf`

Search for the `ListOptions` parameter and change it from `"-l"` to `"-la"`.

It should look like this : 
```conf
#Global settings
<Global>
[...]
ListOptions 		"-la"
[...]
</Global>
```
