# macserve

Turn a Mac into a hardened SSH server automatically.

`macserve` enables Remote Login, locks SSH down to key-only authentication (no root login, no passwords), keeps the Mac awake on power, and can set it to restart after a power failure. It records every change it makes so `undo` can put things back. A one-shot `status` command shows whether the Mac is ready to accept connections, including its LAN address, Tailscale address if present, free disk space, sleep settings, and authorized key count.

The setup tool lives on the `feat/setup-tool` branch until that pull request is merged. After you clone the repo:

```bash
git checkout feat/setup-tool
./macserve --dry-run setup
```

When you are ready to apply changes on the Mac itself:

```bash
./macserve setup
```

By default the tool asks before each change that needs `sudo`. Pass `--yes` to skip the prompts, or `--dry-run` to print the commands without running them.

## Commands

| Command | What it does |
| --- | --- |
| `setup` | Enable Remote Login, harden `sshd` with a drop-in under `/etc/ssh/sshd_config.d/`, keep the Mac awake on AC power, and set auto-restart after a power failure. Prints Tailscale steps for reach-from-anywhere, and offers to install Tailscale with Homebrew if `brew` is available. |
| `add-key <pubkey or file>` | Append an SSH public key to `~/.ssh/authorized_keys` with the right permissions. Duplicate keys are skipped. |
| `status` | Show Remote Login state, LAN IP, Tailscale IP if any, free disk space (warns under 5 GB), sleep settings, and authorized key count. |
| `undo` | Revert every change `macserve` recorded in its state file (`~/.macserve/state`). |

Options can appear before or after the command: `--dry-run`, `-y` / `--yes`, `-h` / `--help`.

## Connect from chroot / Termux

On the phone (Termux, or the chroot fork), after `setup` has run on the Mac. Use the LAN IP from `./macserve status` while you are on the same Wi-Fi, or the Tailscale IP when you are away.

```bash
pkg install openssh
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
ssh-copy-id user@ip
ssh user@ip
```

`ssh-copy-id` asks for the Mac password once. After the key is installed, later logins do not need it. If you already have a public key, you can also run `./macserve add-key ~/.ssh/id_ed25519.pub` on the Mac, or paste the key: `./macserve add-key "ssh-ed25519 AAAA… comment"`.

To reach the Mac from outside your home network, install [Tailscale](https://tailscale.com/download/mac) on the Mac and the [Tailscale Android app](https://tailscale.com/download/android) (or Tailscale in Termux) and sign both into the same tailnet. Then `ssh user@<tailscale-ip>`.

## Requirements

- macOS with `systemsetup`, `pmset`, and OpenSSH (the tools macOS already ships)
- An administrator account, for `setup` and `undo` of system settings
- No extra packages. Homebrew is optional and only used if you ask `setup` to install Tailscale.

## License

MIT. Copyright (c) 2026 Joshua Almeida.
