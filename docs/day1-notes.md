# Day 1: SSH hardening

## Before
- maxauthtries 6
- permitrootlogin prohibit-password
- passwordauthentication no (AWS image default)

## After
- maxauthtries 3
- permitrootlogin no
- passwordauthentication no (pinned explicitly)

## Tests
- Key login after change: <worked>
- Login without a key: <failed>
- Root login with key: <failed>

## Security group
- SSH restricted to a single /32 address; HTTP and HTTPS open to all ipv4
