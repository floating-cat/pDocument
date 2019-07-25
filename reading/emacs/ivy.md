https://oremacs.com/swiper/#getting-started

(global-set-key (kbd "C-c c") 'counsel-compile)

# Action
C-m or RET (ivy-done)
M-o (ivy-dispatching-done)
C-j (ivy-alt-done) == TAB TAB
On a directory, restarts completion from that directory.
On a file or ./, exit completion with the selected candidate.
C-' (ivy-avy)

# Multiple selections and actions, keep minibuffer open
Adding an extra meta key to the normal key chord invokes the special version of the regular commands that enables applying multiple actions.
C-M-m (ivy-call)
C-M-o (ivy-dispatching-call)
C-M-n (ivy-next-line-and-call)
C-M-p (ivy-previous-line-and-call)
ivy-resume
Recalls the state of the completion session just before its last exit.
Useful after an accidental C-m (ivy-done).

# Key bindings that alter the minibuffer input
M-n (ivy-next-history-element)
M-p (ivy-previous-history-element)
M-i (ivy-insert-current)
M-j (ivy-yank-word)
S-SPC (ivy-restrict-to-matches
C-r (ivy-reverse-i-search
Starts a recursive completion session through the command's histor

M-w (ivy-kill-ring-save)

C-o (hydra-ivy/body)

# Commands
// (self-insert-command)
~ (self-insert-command)
M-r (ivy-toggle-regexp-quote)

# My usage:
C-j , C-g (done), M-o, M-n, M-p, M-i, C-r, M-r
C-? (hydra-ivy/body)
s-a (swiper-avy)
