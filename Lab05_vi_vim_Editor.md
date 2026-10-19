# Lab Sheet 5: vi/vim Editor

Vim ke 3 main modes: **Normal** (default, commands), **Insert** (text likhna), **Command-Line** (`:` se shuru).

**Q1. vi/vim se lab5.txt banao aur kam se kam 5 lines likho.**
```
vim lab5.txt
i                      # Insert mode
(5 lines type karo)
Esc  ->  :wq  Enter
```

**Q2. Existing lab5.txt kholo aur contents dikhao.**
```bash
vim lab5.txt           # file khulte hi contents dikhte hain
cat lab5.txt           # bahar se check
```

**Q3. Normal, Insert aur Command-Line mode ke beech switching dikhao.**
- Normal -> Insert: `i`, `a`, `o` (neeche status line par `-- INSERT --`)
- Insert -> Normal: `Esc`
- Normal -> Command-Line: `:` (jaise `:set number`)
- Command-Line -> Normal: `Esc` ya Enter

**Q4. Cursor se pehle aur baad mein text insert karo.**
- `i` = cursor se pehle insert, `a` = cursor ke baad (append)
- `I` = line ke shuru mein, `A` = line ke end mein

**Q5. Current line ke neeche aur upar nayi line jodo.**
- `o` = neeche nayi line (aur Insert mode)
- `O` = upar nayi line (aur Insert mode)

**Q6. File ko bina exit kiye save karo.**
```
Esc  ->  :w
```

**Q7. Save karke exit karo.**
```
:wq        ya      :x       ya      ZZ  (Normal mode mein)
```

**Q8. Bina save kiye exit karo.**
```
:q!
```
(Agar changes nahi kiye to `:q` kaafi hai.)

**Q9. Line ke shuru aur end par jao.**
- `0` = line ka pehla column, `^` = pehla non-blank character
- `$` = line ka end

**Q10. File ke shuru aur end par jao.**
- `gg` = file ki pehli line
- `G` = file ki aakhri line

**Q11. Character, word aur poori line delete karo.**
- `x` = cursor wala character
- `dw` = word
- `dd` = poori line (`3dd` = 3 lines)

**Q12. Current line copy karke neeche paste karo.**
```
yy      # copy (yank)
p       # neeche paste  (P = upar paste)
```

**Q13. Kai lines copy karke dusri jagah paste karo.**
```
3yy         # current se 3 lines copy
(cursor kisi aur line par le jao)
p           # paste
```
Ya Visual mode: `V` -> `j` se select -> `y` -> destination par `p`. Ya command-line: `:2,4y`.

**Q14. Word ko aage ki taraf search karo aur matches ke beech jao.**
```
/word  Enter
n       # agla match
N       # pichla match
```

**Q15. Word ko peeche ki taraf search karo.**
```
?word  Enter
n       # peeche ki disha mein agla match
N       # ulti disha (aage)
```

**Q16. Current line par word ka pehla occurrence replace karo.**
```
:s/old/new/
```

**Q17. Poori file mein sab occurrences replace karo.**
```
:%s/old/new/g
:%s/old/new/gc      # har replacement par confirm poochta hai
```

**Q18. Last change undo aur redo karo.**
- `u` = undo
- `Ctrl+r` = redo

**Q19. Line numbers dikhao aur kisi line par jao.**
```
:set number      # line numbers dikhao  (:set nonumber se band)
Ctrl+g           # current line number aur file info
:10              # line 10 par jao   (ya 10G Normal mode mein)
```

**Q20. Complete workflow: create, edit, search, replace, copy, paste, save, exit.**
```
vim lab5.txt
i  ...kuch lines likho...  Esc
yy  p                     # line copy-paste
/linux  Enter  n          # search
:%s/linux/Linux/g         # replace
dd  u  Ctrl+r             # delete, undo, redo
:set number
:wq
```
Aakhir mein `cat lab5.txt` se result verify karo.
