# Fisher
fisher add
fisher self-update
fisher (upgrade dependencies)

# Z
z, zo (opens file manager)

# fzf
Legacy	New Keybindings	Remarks
Ctrl-t	Ctrl-f	Find a file.
Ctrl-r	Ctrl-r	Search through command history.
Alt-c	Alt-o	cd into sub-directories (recursively searched).
Alt-Shift-c	Alt-Shift-o	cd into sub-directories, including hidden ones.
Ctrl-o	Ctrl-o	Open a file/dir using default editor ($EDITOR)
Ctrl-g	Ctrl-g	Open a file/dir using xdg-open or open command

set -gx EDITOR en
set -U FZF_COMPLETE 1
set -U FZF_ENABLE_OPEN_PREVIEW 1 (not work for other keybindings, so removed)
