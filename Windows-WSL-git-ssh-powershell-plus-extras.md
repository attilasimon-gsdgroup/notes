# Windows + WSL dev setup cheatsheet (March 2026)
One-file reference for SSH, Git, PowerShell 7, Starship, VS Code terminal

## One-command installs via winget (run in PowerShell)
winget install Neovim.Neovim                  # Neovim (preferred over classic vim)
winget install Starship.Starship              # Starship prompt
winget install Microsoft.PowerShell           # PowerShell 7 (if not already from Store)
winget install GNU.MidnightCommander          # mc (Midnight Commander)
winget install vim.vim                        # classic Vim (optional fallback)

## 1. Windows side – SSH & Git
- Custom SSH key: ~/.ssh/id_ed25519_custom (ED25519, passphrase protected)
- Force Git to use system OpenSSH (not Git-for-Windows bundled):
  git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
- Load key into Windows agent (once per boot – solves most "Permission denied" issues):
  Start-Service ssh-agent
  ssh-add "C:\Users\YourUsername\.ssh\id_ed25519_custom"
  # Test: ssh -T git@bitbucket.org

## 2. WSL Ubuntu side – SSH & Git (Bitbucket)
- Custom key: ~/.ssh/id_ed25519_custom
- Use keychain for persistent agent (passphrase once per boot):
  sudo apt install keychain
  # In ~/.bashrc or ~/.zshrc (end of file):
  eval $(keychain --eval --quiet --agents ssh ~/.ssh/id_ed25519_custom)
- Manual fallback (when agent not loaded in a fresh terminal):
  eval "$(ssh-agent -s)"
  ssh-add ~/.ssh/id_ed25519_custom
  # Then test: ssh -T git@bitbucket.org
  # git push should now work

## 3. PowerShell 7 + Starship (in Windows Terminal)
- In $PROFILE (~Documents\PowerShell\Microsoft.PowerShell_profile.ps1):
  Invoke-Expression (&starship init powershell)
- Font: Cascadia Code NF / CaskaydiaCove NF (Nerd Font installed)
- Default terminal profile in Windows Terminal: PowerShell 7

## 4. VS Code integrated terminal
- Set to PowerShell 7:
  "terminal.integrated.defaultProfile.windows": "PowerShell 7"
  (or via Settings UI → Terminal → Default Profile: Windows → PowerShell 7)

## Quick tests when broken
- Windows: ssh -T git@bitbucket.org
- WSL: ssh -T git@bitbucket.org
- If agent empty in WSL: eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519_custom
- Git verbose push: GIT_SSH_COMMAND="ssh -vvv" git push
- Check key perms (WSL): ls -l ~/.ssh/id_ed25519_custom → must be 600

Last updated: March 2026
