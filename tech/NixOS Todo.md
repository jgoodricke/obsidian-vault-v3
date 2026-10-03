## NixOS Todo
- [ ] Initial Setup
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
	- [ ] Set up Vimjoyers project structure: https://github.com/vimjoyer/flake-starter-config/tree/main
- [x] TUI
	- [x] zsh
	- [x] Zoxide
	- [x] Eza
	- [x] Tmux
	- [x] fd
	- [x] ripgrep
	- [x] Starship
	- [ ] builder.io ai cli tool
	- [ ] Neovim and Nixvim
	- [ ] Nerd Fonts
	- [x] Just
- [ ] GUI
	- [x] Set up Hyperland and associated packages.
	- [x] Set up Stylix
	- [ ] Programs
		- [x] Obsidian
			- [ ]  Git LFS
		- [x] Helium
- [ ] Development Environment
	- [ ] Rust
	- [ ] Docker
	- [ ] Node
	- [ ] Finish Setting up Pi
		- [ ] Chrome companion installation and authorization.
		- [ ] Context7/MCP configuration and verification of the unchanged web-researcher tool names.
		- [ ] Exa API-key setup.
		- [ ] Beads and epic-worker setup, including model availability.
		- [ ] Missing external skill dependencies and inconsistent skill references.
		- [ ] Plannotator CLI and Playwright/browser runtime setup.Add 
		- [ ] Sort out config, it looks a bit overly complex.
- [ ] Theming
	- [ ] Add shutdown menu
	- [ ] Add alt options menu
	- [ ] Add more keyboard shortcuts
	- [ ] Fix Clickable panels in Waybar
	- [ ] Style Waybar
	- [ ] Style Hyperland
	- [ ] Style menus
- [ ] Advanced Setup
	- [ ] Set up Flake Parts
	- [ ] Set up the Dendritic Pattern?


## Justfile
### Building
```bash
git add .

nix flake check

sudo nixos-rebuild build --flake .#nixos
sudo nixos-rebuild switch --flake .#nixos
```