# ssh-code

A hacky bash script to allow running the `code` command on a remote server, causing Visual Studio Code to run client-side.

## Requirements

- You must have an SSH server running on your client. This can be firewalled off from the LAN/WAN, but it must be running.
- VS Code must be installed on the client.
- [VS Code Remote Development](https://code.visualstudio.com/docs/remote/remote-overview) must be installed, and you should already be able to connect client --> server manually.

## Usage

Run `ssh-code`, passing any arguments as you normally would to ssh.

Note: A remote SSH command is not supported. The final argument is interpreted as the SSH destination (see _Limitations_).

Once logged in, run `code <file_or_directory>` as you would locally. VS Code will launch locally and connect to the remote server.

For example:
```
❯ ./ssh-code 1.2.3.4
```
which SSHs to the server. Later...
```
❯ code /server/path
```
which causes VS Code to be launched client-side, which then opens the specified server path.

## Limitations

- This script is for interactive usage and always launches a shell. Use `ssh` itself if you need to run other commands.
- The script does its best to clean up temporary keys after itself. If it fails you may find stray keys in your `authorized_keys` file - they are at least harmless as they're random and ephemeral.

### Not implemented
- The various parameters of the `code` script are not supported, just file and folder paths.

## How it works

On connection:
1. Generates a (temporary) SSH key pair.
2. Load this key as an authorized key.
3. Push configuration script to remote host.
4. Connect to remote host via standard SSH, forwarding local SSH port back.
5. Run config script, launch shell.
6. (On exit) Clean up server-side temporary files, remove client side ssh key.

On running `code` on remote.
1. SSH back to client, via forwarded port.
2. Run `code --folder-uri vscode-remote://ssh-remote%2B<host><path>` or `code --file-uri vscode-remote://ssh-remote%2B<host><path>`
