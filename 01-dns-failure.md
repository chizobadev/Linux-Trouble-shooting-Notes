# Case 01: apt update could not reach Ubuntu's servers

**Setup:** Ubuntu 22.04 on WSL1, Windows 10

## What went wrong
I ran `sudo apt update`. Instead of downloading package lists, it printed
"Temporary failure resolving 'archive.ubuntu.com'".

## How I worked it out
1. `ping -c 3 8.8.8.8` worked. A ping to a raw IP address succeeded,
   so the internet connection itself was fine.
2. `ping -c 3 google.com` failed. A ping by name did not work.
3. Since the connection worked but names did not, the problem was DNS,
   the lookup that turns names into IP addresses.

## Cause
Ubuntu's DNS server setting (in /etc/resolv.conf) was not working.
On WSL this file is usually generated automatically from Windows, which is
the likely source of the bad setting. I did not check the original contents.

## Fix
1. Created /etc/wsl.conf with `generateResolvConf = false`, so WSL stops
   overwriting the DNS file.
2. Replaced /etc/resolv.conf with `nameserver 8.8.8.8` (Google's DNS server).
3. Ran `ping -c 3 google.com` again, and it worked.
4. Ran `sudo apt update` again, and it downloaded the package lists.
5. Restarted WSL (`wsl --shutdown`) and reopened Ubuntu. `ping -c 3 google.com`
   still worked, so the fix is permanent.
## What I learned
Test by IP address first, then by name. If the IP works and the name does
not, the problem is DNS, not the connection.
