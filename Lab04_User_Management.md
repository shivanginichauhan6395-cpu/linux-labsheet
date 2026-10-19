# Lab Sheet 4: User Management

**Q1. Current username, UID, primary GID aur supplementary groups dikhao.**
```bash
whoami
id
groups
```
`id` se uid, gid aur groups teeno dikhte hain. `groups` sirf group names dikhata hai.

**Q2. Test user ki /etc/passwd entry dikhao aur fields batao.**
```bash
grep '^root' /etc/passwd       # ya: getent passwd labuser1 (user banane ke baad)
```
Format: `username:x:UID:GID:comment(GECOS):home_directory:login_shell`. `x` ka matlab password /etc/shadow mein hai.

**Q3. Password dikhaye bina account/password status dikhao.**
```bash
sudo passwd -S root
sudo chage -l root
```
`passwd -S` ke status codes: `P` = usable password, `L` = locked, `NP` = no password.

**Q4. useradd se labuser1 banao aur verify karo.**
```bash
sudo useradd labuser1
id labuser1
grep labuser1 /etc/passwd
```

**Q5. labuser2 ko home directory ke saath banao aur ownership verify karo.**
```bash
sudo useradd -m labuser2
ls -ld /home/labuser2
```
Owner aur group `labuser2` hona chahiye (`-m` = create home).

**Q6. labuser3 ko /bin/bash shell ke saath banao.**
```bash
sudo useradd -s /bin/bash labuser3
grep labuser3 /etc/passwd      # last field /bin/bash hona chahiye
```

**Q7. labuser1 ka password set karo aur status verify karo.**
```bash
sudo passwd labuser1
sudo passwd -S labuser1        # status P dikhna chahiye
```

**Q8. labuser1 ka password change karo aur status dobara dekho.**
```bash
sudo passwd labuser1
sudo passwd -S labuser1        # last-change date aaj ki ho jayegi
sudo chage -l labuser1
```

**Q9. usermod se labuser2 ka shell /bin/bash karo.**
```bash
sudo usermod -s /bin/bash labuser2
grep labuser2 /etc/passwd
```

**Q10. labuser2 ki home directory change karo: pehle bina contents move kiye, phir move ke saath.**
```bash
echo "data" | sudo tee /home/labuser2/file.txt

# a) Bina move (-m ke bina): sirf /etc/passwd entry badalti hai
sudo usermod -d /home/labuser2_new labuser2
grep labuser2 /etc/passwd
ls /home/labuser2_new           # exist nahi karegi
ls /home/labuser2               # purani directory aur contents wahin hain

# b) Move ke saath (-m -d)
sudo usermod -d /home/labuser2 labuser2           # pehle wapas purani par
sudo usermod -m -d /home/labuser2_moved labuser2
ls -l /home/labuser2_moved      # file.txt yahan aa gayi
ls /home/labuser2               # ab nahi hai
```

**Q11. userlabgroup banao, labuser1 ko usme add karo aur membership verify karo.**
```bash
sudo groupadd userlabgroup
sudo usermod -aG userlabgroup labuser1     # -a (append) ke bina purane groups hat jate hain
id labuser1
getent group userlabgroup
```

**Q12. labuser3 ko lock karo, verify karo, phir unlock karke verify karo.**
```bash
sudo passwd labuser3                # pehle password set (kuch systems par bina password unlock fail hota hai)
sudo usermod -L labuser3
sudo passwd -S labuser3             # status L
sudo usermod -U labuser3
sudo passwd -S labuser3             # status P
```
(`passwd -l` / `passwd -u` bhi use kar sakte ho.)

**Q13. chage se labuser3 ki detailed password-aging info dikhao.**
```bash
sudo chage -l labuser3
```
Fields: Last password change, Password expires, Password inactive, Account expires, Min/Max days, Warning days.

**Q14. labuser3 ki account expiration date set karo aur verify karo.**
```bash
sudo chage -E 2026-12-31 labuser3
sudo chage -l labuser3 | grep "Account expires"
```
(Expiry hatane ke liye: `sudo chage -E -1 labuser3`.)

**Q15. labuser1 ka primary group kisi existing group se badlo.**
```bash
sudo usermod -g userlabgroup labuser1      # -g = primary group (capital -G supplementary hai)
id labuser1                                # gid=userlabgroup dikhna chahiye
```

**Q16. id, groups aur getent se user/group info dikhao.**
```bash
id labuser1
groups labuser1
getent passwd labuser1
getent group userlabgroup
```

**Q17. labuser1 ko home directory hataye bina delete karo, verify karo.**
```bash
sudo mkdir -p /home/labuser1          # agar nahi bani thi (Ubuntu par useradd bina -m home nahi banata)
sudo userdel labuser1                 # -r nahi
getent passwd labuser1                # koi output nahi
ls -ld /home/labuser1                 # directory abhi bhi maujood
```

**Q18. labuser2 ko home directory ke saath delete karo.**
```bash
sudo userdel -r labuser2
getent passwd labuser2                # output nahi
ls -ld /home/labuser2_moved           # No such file or directory (agar Q10 ke baad ka path ho)
```

**Q19. Complete workflow: user create, password, shell, supplementary group, lock/unlock, verify.**
```bash
sudo useradd -m -s /bin/bash flowuser
echo 'flowuser:Test@123' | sudo chpasswd
sudo groupadd flowgroup
sudo usermod -aG flowgroup flowuser
sudo usermod -L flowuser && sudo passwd -S flowuser    # L
sudo usermod -U flowuser && sudo passwd -S flowuser    # P
getent passwd flowuser
id flowuser
ls -ld /home/flowuser
```

**Q20. Saare test users aur lab groups cleanup karo; system accounts safe hain verify karo.**
```bash
sudo userdel -r flowuser
sudo userdel -r labuser3
sudo rm -rf /home/labuser1 /home/labuser2_new          # bache hue folders
sudo groupdel flowgroup
sudo groupdel userlabgroup
getent passwd | grep -E 'labuser|flowuser'             # koi output nahi
getent group  | grep -E 'labuser|flowgroup|userlabgroup'
sudo pwck -r && sudo grpck -r                          # read-only integrity check
```
