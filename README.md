# kitty

Ansible role to install the [kitty](https://sw.kovidgoyal.net/kitty/) terminal, apply the Catppuccin Mocha theme, stow personal dotfiles config, and configure a GNOME keyboard shortcut on Debian/Ubuntu systems.

## Requirements

- Debian or Ubuntu host
- `become: true` privileges (sudo)
- Depends on the `dotfiles` role (clones `~/dotfiles`, which must contain a `kitty` stow package; provides `dotfiles_path` and `user_home_path`). `ansible_user` must be set
- GUI/GNOME session required at runtime for shortcut configuration (skipped gracefully if not present)

## Role Variables

| Variable | Default | Description |
|---|---|---|
| `kitty_package_name` | `kitty` | Package name to install via apt |
| `kitty_min_theme_version` | `0.26.0` | Minimum kitty version required to apply the theme via `kitten themes` |
| `kitty_theme_name` | `Catppuccin-Mocha` | Theme name passed to `kitty +kitten themes` |
| `kitty_shortcut_name` | `Kitty Terminal` | GNOME custom shortcut display name |
| `kitty_shortcut_command` | `kitty` | GNOME custom shortcut command |
| `kitty_shortcut_binding` | `<Control><Alt>t` | GNOME custom shortcut key binding |

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - role: dotfiles
    - role: kitty
```

Override variables if needed:

```yaml
- hosts: all
  become: true
  roles:
    - role: dotfiles
    - role: kitty
      vars:
        kitty_shortcut_binding: "<Control><Alt>k"
```

## Manual setup (without Ansible)

1. Install stow and kitty
   ```
   sudo apt update && sudo apt install stow kitty -y
   kitty --version
   # Minimal version supported: 0.26
   ```

2. Remove existing kitty stow
   ```
   cd ~/dotfiles
   stow -D kitty
   ```

3. Apply Catppuccin Mocha theme
   ```
   kitty +kitten themes --reload-in=all Catppuccin-Mocha
   ```

4. Apply kitty stow
   ```
   cd ~/dotfiles
   stow kitty
   ```

5. Append to `~/.config/kitty/kitty.conf`
   ```
   include kitty-private.conf
   ```

6. Set up kitty as a custom GNOME shortcut (Ubuntu)
   ```
   gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings \
   "['/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/']"

   gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ name 'Kitty Terminal'

   gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ command 'kitty'

   gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ binding '<Control><Alt>t'
   ```

   Expect:
   Settings -> Keyboard -> View and Customize Shortcuts -> Custom Shortcuts -> Add Shortcut
   Name: Kitty Terminal, Command: kitty, Shortcut: ctrl + alt + t

## Known issues

If your OS ships a kitty version < 0.26, install from source and set up manually:
https://sw.kovidgoyal.net/kitty/binary/

For Ubuntu, to set kitty as the default terminal:
https://linuxconfig.org/ubuntu-change-default-terminal-emulator
```
gsettings set org.gnome.desktop.default-applications.terminal exec '.local/bin/kitty'
```

## License

MIT

## Author

Varssos