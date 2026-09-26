# Incident 001 -- Private SSH Key Exposed in Public Repository

**Date:** 2026-09-26
**Severity:** High

## Situation
While setting up SSH key authentication, I placed a `.txt` file containing the SSH **private** key inside the project folder. The folder was already tracked by Git and pushed to a public GitHub repository, exposing the private key.

## Task
Contain the exposure, prevent unauthorized access using the compromised key, and stop it from happening again.

## Action
1. Generated a brand new SSH key pair to replace the compromised one
2. Removed the old public key from the VM's `authorized_keys` and added the new one
3. Verified the new key worked before removing the old one from the trust chain
4. Rewrote the local Git history (removed `.git`, reinitialized, force-pushed) to erase the private key from GitHub's history
5. Added a `.gitignore` file to block key files (`id_ed25519*`, `*.pem`, `*.key`, `*.txt`) from ever being tracked again

## Result
The compromised key is fully retired and no longer grants access. The current key is not accessible in any Git history. Future accidental commits of key material are now blocked by `.gitignore`.

## Lesson learned
Private keys must never live inside a project folder tracked by Git -- only in `~/.ssh`. A `.gitignore` should be set up **before** the first commit of any new project, not after.