## NixOS Todo
- [ ] Initial Setup
	- [x] Push changes to Git
	- [x] Get Pi set up
	- [x] Set up Flakes
	- [x] Add agents.md
	- [x] Git commit moving to Flakes
	- [x] Set up Home Manager
	- [ ] Move Pi settings into Home Manager
	- [ ] Git tools
		- [ ] Tig
		- [ ] Delta
		- [ ] git-igitt
	- [ ] Just
	- [ ] Finish setting up Pi
	- [ ] Set up Vimjoyers project structure: https://github.com/vimjoyer/flake-starter-config/tree/main
- [ ] TUI
	- [ ] zsh
	- [ ] Zoxide
	- [ ] Eza
	- [ ] Tmux
	- [ ] fd
	- [ ] set up builder.io ai cli tool
	- [ ] Set up Neovim and Nixvim
	- [ ] Nerd Fonts
	- [ ] Docker
	- [ ] Rust
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
- [ ] Advanced Setup
	- [ ] Set up Flake Parts
	- [ ] Set up the Dendritic Pattern?


## Justfile
### Building
```bash
git add .

sudo nixos-rebuild build --flake .#nixos
sudo nixos-rebuild switch --flake .#nixos
```