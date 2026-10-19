# Lab 7: Process Management

**Q1. Current user ke processes dikhao.**
```bash
ps
ps -u $USER
```
Columns: PID (process ID), TTY (terminal), TIME (CPU time), CMD (command). `ps -u` mein STAT (state) bhi aata hai: R running, S sleeping, T stopped, Z zombie.

**Q2. Sabhi processes dikhao.**
```bash
ps -e        # ya ps -A  ya  ps -ef
ps -ef       # UID, PID, PPID, C, STIME, TTY, TIME, CMD
```

**Q3. ps aux se CPU aur memory usage dekho.**
```bash
ps aux
# USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

**Q4. Parent-child hierarchy dikhao.**
```bash
ps -ef --forest
ps axjf
pstree -p
```

**Q5. top chalao aur observe karo.**
```bash
top
```
Upar summary (uptime, load average, tasks, %Cpu(s), memory/swap). Neeche PID, USER, PR, NI, VIRT, RES, SHR, S, %CPU, %MEM, TIME+, COMMAND. Bahar nikalne ke liye `q`.

**Q6. top se high CPU/memory process dhundho aur PID note karo.**
```bash
yes > /dev/null &        # test load
top
```
top ke andar `P` = CPU se sort, `M` = memory se sort. Sabse upar wala process high usage wala hai. Test ke baad `kill %1`.

**Q7. htop chalao.**
```bash
htop                       # nahi mile to: sudo apt install htop  /  sudo dnf install htop (EPEL)
```
Keys: F3 search, F4 filter, F5 tree view, F6 sort, F9 kill, F10 quit. Upar CPU/memory bars dikhte hain.

**Q8. Foreground mein long command chalao aur Ctrl+C se roko.**
```bash
sleep 300
^C                         # Ctrl+C (SIGINT)
```

**Q9. & se background mein chalao aur job info dekho.**
```bash
sleep 300 &
# [1] 12345      <- [job number] PID
```

**Q10. jobs command.**
```bash
jobs
# [1]+  Running   sleep 300 &
```

**Q11. fg se job foreground mein lao aur safely terminate karo.**
```bash
fg %1                      # ya fg
^C                         # Ctrl+C se terminate
jobs                       # job gayab
```

**Q12. Ctrl+Z se foreground process suspend karo.**
```bash
sleep 300
^Z
# [1]+  Stopped   sleep 300
jobs                       # State: Stopped
```

**Q13. bg se stopped job ko background mein resume karo.**
```bash
bg %1
jobs                       # ab Running
```

**Q14. jobs -l se job number aur PID dikhao.**
```bash
jobs -l
# [1]+ 12345 Running   sleep 300 &
```

**Q15. kill se PID select karke process khatam karo.**
```bash
sleep 500 &
echo $!                    # PID
kill <PID>                 # default signal SIGTERM (15)
ps -p <PID>                # koi entry nahi = process khatam
jobs
```

**Q16. kill -TERM aur SIGTERM pehle kyun use hota hai.**
```bash
sleep 500 &
kill -TERM $!              # ya kill -15 $!
```
SIGTERM process ko request hai; process usse catch karke apni cleanup kar sakta hai (files close, buffers flush, temp files hataana, children ko band karna). SIGKILL ko process catch ya ignore nahi kar sakta, isliye data loss/corruption ho sakta hai. Isiliye pehle TERM, na maane to hi KILL.

**Q17. TERM kaam na kare to kill -KILL.**
```bash
( trap '' TERM; while true; do sleep 1; done ) &    # SIGTERM ignore karne wala test process
P=$!
kill -TERM $P ; sleep 1 ; ps -p $P                  # abhi bhi zinda
kill -KILL $P            # ya kill -9 $P
sleep 1 ; ps -p $P                                  # ab khatam
```

**Q18. pkill se naam se process khatam karo.**
```bash
sleep 600 &
pgrep -a sleep
pkill sleep                # ya pkill -x sleep (exact name)
pgrep sleep                # koi output nahi
```
Dhyan: `pkill` pattern match karta hai, isliye sirf test processes ka naam do.

**Q19. ps, jobs aur control commands se multiple test processes manage karo.**
```bash
sleep 700 & sleep 800 & sleep 900 &
jobs -l
ps -o pid,stat,cmd -C sleep
kill -STOP %2              # suspend
jobs
kill -CONT %2              # resume
kill %1                    # job 1 terminate
kill %3
jobs
```

**Q20. Complete workflow.**
```bash
ps aux | head
sleep 1000                 # foreground
^Z                         # suspend
bg %1                      # background mein chalao
jobs -l
kill -STOP %1 ; jobs       # suspend
kill -CONT %1 ; jobs       # resume
kill %1                    # terminate
jobs                       # Terminated
pgrep -a sleep             # koi test process nahi bacha (nahi to pkill sleep)
```
