## NixOS Todo
```
I would like to add the following features to my config. Using @~/ref-repos/omarchy as a guide. 
		- Screenshot
		- Screen Record
		- Color Picker
		- Transcoding menu
		- Text Extraction to Clipboard
		- Dictation

Also add the Capture Menu, along with these hotkeys to use them:

Capture #
Hotkey	Function
Super + Ctrl + C	Capture menu (for keyboards w/o PrintScr button)
Print Screen	Screenshot
Alt + Print Screen	Screenrecord
Super + Print Screen	Color picker
Super + Ctrl + Print Screen	Text extraction to clipboard
Alt + Shift + L	Copy current URL from webapp or Chromium
Super + Ctrl + X	Start/stop dictation (requires Install > AI > Dictation)
F9	Push-to-talk dictation (requires Install > AI > Dictation)
```

- [ ] Fix boot order
- [x] Commit LLM Prompt picker changes
- [x] remove S from password
	- [x] webres
	- [x] bishop
- [x] Change password to something simpler
- [x] Initial Setup
	- [x] Push changes to Git
	- [x] Get Pi set up
	- [x] Set up Flakes
	- [x] Add agents.md
	- [x] Git commit moving to Flakes
	- [x] Set up Home Manager
	- [x] Move Pi settings into Home Manager
	- [x] Finish setting up Pi
	- [x] Git tools
		- [x] Tig
		- [x] Delta
		- [x] git-igitt
	- [x] Set up Vimjoyers project structure: https://github.com/vimjoyer/flake-starter-config/tree/main
- [x] TUI
	- [x] zsh
	- [x] Zoxide
	- [x] Eza
	- [x] Tmux
	- [x] fd
	- [x] ripgrep
	- [x] Starship
	- [x] builder.io ai cli tool
	- [x] Neovim and Nixvim
	- [x] Nerd Fonts
	- [x] Just
- [ ] GUI
	- [x] Set up Hyperland and associated packages.
	- [x] Set up Stylix
	- [ ] Programs
		- [x] Obsidian
			- [ ]  Git LFS
		- [x] Helium
- [x] Development Environment
	- [x] Rust
	- [x] Docker
	- [x] Node
	- [x] Finish Setting up Pi
		- [x] Chrome companion installation and authorization.
		- [x] Context7/MCP configuration.
		- [x] Exa API-key setup.
		- [x] Beads and epic-worker setup, including model availability.
		- [x] Missing external skill dependencies and inconsistent skill references.
		- [x] Plannotator CLI.
		- [x] Set up Linear MCP.
		- [x] Migrate from bd to bd-rust
- [x] Theming
	- [x] Add shutdown menu
	- [x] Add more keyboard shortcuts
	- [x] Fix Clickable panels in Waybar
	- [x] Style Walker
	- [x] Style Waybar
	- [x] Style Hyperland
	- [x] Add LLM Prompts Walker Menu
	- [x] Style Staship
	- [x] Style Lock page
	- [x] Migrate apps
		- [x] Wofi -> Walker with Elephant
		- [x] NetworkManager applet -> Impala
		- [x] Blueman -> Bluetui
		- [x] Pavucontrol -> Wiremix
	- [x] Fix broken walker menus
		- [x] Fix the Super K.
		- [x] Migrate from Cliphist -> Walker / Elephant clipboard history
		- [x] Add options menu (Super + alt + space) - GRILLING SESSION
	- [x] Update layout of Waybar.
	- [x] Add Yazi File Manager, add the other file manager as the alt shortcut
	- [x] Add update icon.
	- [x] Confirm rollbacks are working.
	- [x] Add notification when screenshot taken.
- [ ] WSL
	- [x] Basic Config
	- [x] set NixOS as default wsl instance.
	- [x] Copy tig settings
	- [x] copy rg settings
	- [x] copy tmux settings
	- [x] Update ZSH
		- [x] General configs
			- [x] Add ZSH plugins
			- [x] Add Shell Integrations
		- [x] specific configs
			- [x] Add git status hooks from dotfiles/wsl-arch/.custom.zshrc
			- [x] Add bin scripts
				- [x] worktree sync
				- [x] git AI commit
	- [x] Add refreshing zsh and tmux to the switch recipe
	- [x] Clone and set up project
	- [x] Set up worktrees
	- [ ] Finish setting up git_ai_commit
		- [ ] Add alias to zshrc
		- [ ] Add openAI key
	- [ ] Fix AI Shell not working
	- [ ] Install Nix REPL
		- [ ] Figure out how to get the AI to use it.
- [ ] Neovim
	- [x] Port Nvim Kickstart Core
	- [x] Port Nvim Kickstart Optional Plugins
	- [x] Update theme.
		- [x] Switch to Catpuccin theme
		- [x] figure out if its possible to use the Stylix theming.
	- [ ] Add [Vim Coach](https://github.com/shahshlok/vim-coach.nvim)
- [ ] Pi
	- [x] Change layout of status bar on small windows.
	- [x] Add Catppuccin theme to Pi
	- [ ] Find better subagent package.
	- [ ] Add seperate field to statusbar for tracking total time.
	- [ ] Add notification when task completed.
- [ ] Neovim Round 2
	- [x] Update NVim theme to use Stylix instead of default theme.
	- [ ] Add Git diff/comparison.
	- [ ] Add breakpoints.
	- [ ] Add conditional breakpoints.
	- [ ] Add [Vim Be Good](https://github.com/ThePrimeagen/vim-be-good)
	- [ ] Add spell checker.
- [ ] Pi Round 2
	- [ ] Update Pi to use Stylix theme. 
	- [ ] Sort out config, it looks a bit overly complex.
- [ ] Theming - Missing Features
	- [x]  Grow Window shortcuts
	- [x] Toggle pseudo window style (natural v stretch)
	- [x] Go full width
	- [x] Go full screen inside window
	- [x] Scroll through workspaces
	- [x] Volume Popup
	- [x] Brightness Popup
	- [x] Notifications
		- [x] Dismiss latest notification
		- [x] Dismiss all notifications
		- [x] Toggle silencing notifications?
		- [x] Invoke most recent notification
	- [x] window grouping
	- [x] Scratchpad
	- [x] Toggle between dwindle and scrolling layout
	- [x] Power Profile
	- [x] Share menu (via LocalSend)
	- [x] Reminders
		- [x] Set a reminder
		- [x] See all reminders
		- [x] Clear all reminders
	- [x] Capture
		- [x] Screenshot 
		- [x] Screen Record (~/tmp/screen-record-guide.md)
		- [x] *Color Picker*
		- [x] Transcoding menu (~/tmp/transcoding-menu-implementation.md)
		- [x] Text Extraction to Clipboard (~/tmp/text-extraction-to-clipboard.md)
		- [x] Dictation (~/tmp/dictation-implementation.md)
	- [ ] Toggle Menu
		- [x] Night Light (~/tmp/night-light-instructions.md))
		- [x] Audio Output (~tmp/audio-output-toggle-guide.md)
		- [x] Idle/Sleep prevention (~/tmp/prevent-idle-implementation.md)
		- [x] Laptop Display (~tmp/laptop-display-toggle-guide.md)
		- [x] Toggle Top Bar (~/tmp/toggle-statusbar-instructions.md)
	- [x] Zoom in on screen (~/tmp/omarchy-screen-zoom-guide.md)
	- [x] Cycle forward through monitor scaling options (/tmp/monitor-scaling-guide.md)
	- [x] Minimum/Maximum screen brightness (~/tmp/omarchy-brightness-hotkeys-guide.md)
	- [x] Precise 1% volume and brightness changes (~/tmp/precise-media-hotkeys-guide.md)
	- [x] System Sleep (and sleep on battery)
	- [x] Hardware menu (/tmp/omarchy-hardware-menu-guide.md)
		- [x] Toggle Touchpad (~/tmp/toggle-touchpad-instructions.md)
		- [x] Toggle Screen Mirroring (~/tmp/screen-mirroring-guide.md)
	- [x] About Menu (~/tmp/fastfetch-about-page-guide.md)
	- [x] Change Btop to Super + Ctrl + T
	- [ ] Disable Notification Timeout (set default-timeout to 0 and ignore-timeout to 1).
	- [x] Check if any other features are missing
	- [x] Fix keybindings baseline test
- [ ] Theming Round 2
	- [x] Fix keyboard shortcut for moving windows.
	- [x] Fix dithering direction.
	- [x] Fix size and position of floating windows
	- [ ] Add a hotkey (or another way) to see the previous notifications.
	- [ ] Add Google Calendar shortcut
	- [ ] Set up Helium
	- [ ] Figure out why dictation is slow compared to Omarchy
	- [ ] Fix Alt + Brightness Down not working.
	- [ ] Fix styling the about menu to look like [this](https://forum.endeavouros.com/uploads/default/original/3X/9/1/916d87314f525a95ce5a0ad3278635d91af6336d.jpeg).
	- [ ] Style SwayOSD (volume/brightness) to look like Omarchy
	- [x] Add Screensaver (~/tmp/omarchy-screensaver-notes.md)
		- [x] [Omarchy Screensaver]([https://omarchy.org/manual/branding/](https://omarchy.org/manual/branding/#screensaver))
		- [x] [Pipes](https://github.com/pipeseroni/pipes.sh)
		- [x] [Birds](https://github.com/clainstone/cbirds)
		- [x] [Rain](https://github.com/nkleemann/ascii-rain)
	- [x] [About Screen](https://omarchy.org/manual/branding/#about-screen)
- [ ] Theming Round 3
	- [ ] Add VLC
	- [ ] Add an image viewer that supports moving between images.
	- [ ] Add a PDF reader that supports signing PDFs (like Xournal++)
	- [ ] Install WLR Which Key (and find an equivalent for Windows)
	- [ ] Quick Emojis (find a better example)
	- [ ] Style Login Page
	- [ ] Style Bootloader
	- [x] Figure out why login screen freezes when using mouse. 
	- [x] Get better background:
		- [x] https://github.com/zhichaoh/catppuccin-wallpapers/blob/main/os/nix-black-4k.png
- [ ] Theming Round 4
	- [ ] Install catppuccin-nix: https://github.com/catppuccin/nix
	- [ ] Add icons to the keyboard shortcuts menu.
	- [ ] Make update button interactive.
		- [ ] Pass the update process to an LLM.
	- [ ] Change Cursor
		- [ ] https://github.com/catppuccin/cursors
- [ ] Theming round 5
	- [ ] Install Dynamic Dark theme
	- [ ] check out ricing list:
		- [ ] https://github.com/fosslife/awesome-ricing
		- [ ] https://github.com/avtzis/awesome-linux-ricing
- [ ] Advanced Setup
	- [ ] Set up Flake Parts
	- [ ] Set up the Dendritic Pattern?
- [ ] Maybe Implement these Omarchy Features:
	- [ ] Hotkeys for moving between monitors?
	- [ ] Weather
	- [ ] Notification Toggle
		- [ ] Current Time
		- [ ] Current Battery
	- [ ] Windows VM
	- [ ] Hibernation
## Justfile
### Building
```bash
git add .

nix flake check

sudo nixos-rebuild build --flake .#bishop
sudo nixos-rebuild switch --flake .#bishop
```