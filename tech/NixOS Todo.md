## NixOS Todo
- [ ] Initial Setup
	- [x] Push changes to Git
	- [x] Get Pi set up
	- [x] Set up Flakes
	- [x] Add agents.md
	- [x] Git commit moving to Flakes
	- [x] Set up Home Manager
	- [x] Move Pi settings into Home Manager
	- [ ] Finish setting up Pi
	- [ ] Git tools
		- [ ] Tig
		- [ ] Delta
		- [ ] git-igitt
	- [ ] Set up Vimjoyers project structure: https://github.com/vimjoyer/flake-starter-config/tree/main
- [ ] TUI
	- [ ] zsh
	- [ ] Zoxide
	- [ ] Eza
	- [ ] Tmux
	- [ ] fd
	- [ ] Starship
	- [ ] builder.io ai cli tool
	- [ ] Neovim and Nixvim
	- [ ] Nerd Fonts
	- [ ] Just
- [ ] GUI
	- [ ] Set up Hyperland and associated packages.
	- [ ] Set up Stylix
	- [ ] Programs
		- [ ] Obsidian
			- [ ]  Git LFS
		- [ ] Helium
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
		- [ ] Plannotator CLI and Playwright/browser runtime setup.
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