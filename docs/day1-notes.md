# Day 1: SSH and Security Group Hardening

**Goal:** only I can reach SSH, and SSH accepts only key-based logins with no root access.

For authorized, educational use only. This was done on my own AWS instance.

## SSH settings: before and after

### MaxAuthTries
- Before: 6
- After: 3
- What it does: how many wrong login attempts are allowed per connection before the server hangs up. Lower means less guessing.

### PermitRootLogin
- Before: prohibit-password
- After: no
- What it does: controls direct logins as the all-powerful root user. Before, root could log in with a key. Now root cannot log in at all.

### PasswordAuthentication
- Before: no (AWS image default)
- After: no (pinned explicitly)
- What it does: whether a password can be used to log in. It was already off. I set it explicitly so it stays off even if the image defaults change.

### KbdInteractiveAuthentication
- Before: no (AWS image default)
- After: no (pinned explicitly)
- What it does: a second route that can also ask for passwords. Kept off for the same reason.

### PubkeyAuthentication
- Before: yes
- After: yes (pinned explicitly)
- What it does: whether key-pair login is allowed. This is the only way in, so it stays on.

Only two values really changed (MaxAuthTries and PermitRootLogin). The other three were verified and pinned.

**How it was applied:** a drop-in file, `/etc/ssh/sshd_config.d/00-hardening.conf`. The `00-` prefix makes it load before AWS's `60-cloudimg-settings.conf`, and sshd uses the first value it reads for each setting. The main `sshd_config` file was not edited. I checked the result with `sshd -T`, which prints the settings actually in effect.

## Security group

- Port 22 (SSH): open to one address only (my IPv4, /32). Only my connection can reach the SSH service, and everyone else times out.
- Port 80 (HTTP): open to 0.0.0.0/0. It is a web server, so the public must reach it.
- Port 443 (HTTPS): open to 0.0.0.0/0. Same reason, for encrypted web traffic.
- All other inbound ports were removed.

## Tests

All tests were run from my Kali machine.

### Test 1: key login after the change
- Command: `ssh -4 -i key.pem ubuntu@server`
- Result: worked
- Why it matters: proves the hardening did not lock me out.

### Test 2: login without a key
- Command: `ssh -4 -o PubkeyAuthentication=no ubuntu@server`
- Result: refused (Permission denied (publickey))
- Why it is a pass: proves a password or no credentials cannot get in.

### Test 3: root login with a valid key
- Command: `ssh -4 -i key.pem root@server`
- Result: refused
- Why it is a pass: `PermitRootLogin no` blocks root even with a valid key.

For tests 2 and 3, a refusal is the result I wanted.

## Lessons learned

- `sshd -T` showed no change after my first attempt, which told me the setting was not being read. The file had a typo (ssh instead of sshd), and I fixed it.
- The AWS Ubuntu image already had password login off, so my real changes were smaller than I first assumed. I checked the "before" state instead of assuming it.
- I kept a second SSH session open while changing the config, so a mistake could not lock me out.

## Next steps

- Day 2: UFW firewall, fail2ban, Nginx security headers
- Day 3: Nmap scans before and after hardening
