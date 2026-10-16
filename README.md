# tinted-dircolors

Local tinty template that builds a truecolor `dircolors` file per scheme
(base16, base24, tinted8), in the spirit of seebi/dircolors-solarized.

- `src/dircolors.in`: the one file to edit. `@ROLE@` placeholders expand to `R;G;B`.
- `gen-templates.sh`: expands roles into `templates/{base16,tinted8}.mustache`.
- `dircolors/`: built output, one file per scheme.

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

## Rebuild (after editing src/ or `tinty update`)

```bash
./gen-templates.sh
tinty build .
tinty apply "$(tinty current)"
```

The shell picks up the artifact
`~/.local/share/tinted-theming/tinty/artifacts/tinted-dircolors-dircolors-file.dircolors`
through `_tinty_dircolors` in `~/.zshrc`.

## Tinted Configuration

Append to `~/.config/tinted-theming/tinty/config.toml`:

```toml
[[items]]
name = "tinted-dircolors"
path = "~/.config/tinted-theming/tinty/tinted-dircolors"
themes-dir = "dircolors"
supported-systems = ["base16", "base24", "tinted8"]
```

Include in `~/.zshrc` prior to invoking Oh-My-Zsh:

```zsh
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
