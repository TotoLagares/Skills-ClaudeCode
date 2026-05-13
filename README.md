6:49 PMClaude respondió: Ojo: los bloques de código internos los puse con un caracter invisible al lado de los backticks para que no rompan el bloque exterior.# My Claude Code Skills

Personal backup of my custom Claude Code skills so I can use them across multiple devices.

Claude Code skills live locally in `~/.claude/skills/` and don't sync through my Anthropic account, so this repo is the workaround: clone it on any machine and the skills come with me.

## Usage

Clone into your user skills directory:

​```bash
git clone https://github.com/<user>/<repo>.git ~/.claude/skills
​```

Or clone anywhere and symlink:

​```bash
ln -s /path/to/this/repo ~/.claude/skills
​```

## Structure

Each skill lives in its own folder with a `SKILL.md` at the root.
