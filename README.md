# tinted-dircolors

Remote tinty template that provides a truecolor `dircolors` file per scheme
(base16, base24, tinted8), in the spirit of seebi/dircolors-solarized.

- `src/dircolors.in`: the one file to edit. `@ROLE@` placeholders expand to `R;G;B`.
- `gen-templates.sh`: expands roles into `templates/{base16,tinted8}.mustache`.
- `dircolors/`: built output, one file per scheme (committed; tinty reads it directly).

| Role    | base16/24 | tinted8                     | Used for                          |
|---------|-----------|-----------------------------|-----------------------------------|
| BG      | base00    | ui.global.background.normal | text on colored backgrounds       |
| BG_ALT  | base01    | palette.black.normal        | fifo/dev/orphan backgrounds       |
| COMMENT | base03    | palette.gray.normal         | backups, temp, logs, build output |
| RED     | base08    | palette.red.normal          | archives, packages, setuid        |
| ORANGE  | base09    | palette.orange.normal       | data/config                       |
| YELLOW  | base0A    | palette.yellow.normal       | documents, important files        |
| GREEN   | base0B    | palette.green.normal        | executables                       |
| CYAN    | base0C    | palette.cyan.normal         | symlinks, audio                   |
| BLUE    | base0D    | palette.blue.normal         | directories                       |
| MAGENTA | base0E    | palette.magenta.normal      | images, video, sockets            |
| BROWN   | base0F    | palette.brown.bright        | keys and crypto                   |

## Development

Edit `src/dircolors.in`, then preview locally:

```bash
./gen-templates.sh
tinty build .
```

Commit and push `src/`, `templates/`, and `dircolors/`. The
[update workflow](.github/workflows/update.yml) also regenerates the templates and
rebuilds `dircolors/` on every push touching `src/` and weekly against the latest
upstream schemes. Users pick up changes with `tinty update`.

The tinty hook copies the applied scheme to `~/.dircolors`,
which `_tinty_dircolors` in `~/.zshrc` loads.
When `$USERPROFILE` is set (WSL),
it also copies it to `$USERPROFILE/.dircolors` for Windows-side tools.

## Tinted Configuration

Append to `~/.config/tinted-theming/tinty/config.toml`:

```toml
[[items]]
name = "tinted-dircolors"
path = "https://github.com/jfishe/tinted-dircolors"
themes-dir = "dircolors"
hook = "cp -f %f \"$HOME/.dircolors\" && { [ -z \"$USERPROFILE\" ] || cp -f %f \"$USERPROFILE/.dircolors\"; }"
supported-systems = ["base16", "base24", "tinted8"]
```

On WSL, the hook reads `$USERPROFILE` from the shell that runs `tinty`.
To make it available, either:

- (Recommended) add `USERPROFILE/p` to `WSLENV` on the Windows side, so WSL
  passes it through as a Linux path. In PowerShell:

  ```powershell
  setx WSLENV "USERPROFILE/p:$env:WSLENV"
  ```

- or set it in `~/.zshrc` before `_tinty_dircolors` (see below).

Include in `~/.zshrc` prior to invoking Oh-My-Zsh:

```zsh
# WSL only, if USERPROFILE is not passed through WSLENV:
# export USERPROFILE=${USERPROFILE:-$(wslpath "$(cmd.exe /c 'echo %USERPROFILE%' 2>/dev/null | tr -d '\r')")}

# tinted-dircolors: LS_COLORS from ~/.dircolors (copied there by the tinty hook)
_tinty_dircolors() {
  local f=~/.dircolors
  [[ -r $f ]] || return
  eval "$(dircolors -b "$f")"
  zstyle ':completion:*' list-colors "${(s.:.)LS_COLORS}"
}
_tinty_dircolors
tinty() {
  command tinty "$@"
  local r=$?
  [[ $1 == (apply|cycle|init) ]] && _tinty_dircolors
  return $r
}

source $ZSH/oh-my-zsh.sh
```

## License and Credits

tinted-dircolors is released under the [MIT License](LICENSE.txt).

- File-type layout modelled on
  [seebi/dircolors-solarized](https://github.com/seebi/dircolors-solarized),
  Copyright (c) 2020 Sebastian Tramp, MIT License.
- Based on the GNU coreutils `dircolors` database (`dircolors -p`),
  Copyright (C) 1996-2025 Free Software Foundation, Inc.
- Color schemes from
  [tinted-theming/schemes](https://github.com/tinted-theming/schemes) (MIT);
  each generated file credits its scheme author.
