# Lab Sheet 6: File Permissions & ACL

**Q1. lab6 directory aur uske andar permissions.txt banao.**
```bash
mkdir lab6
touch lab6/permissions.txt
cd lab6
```

**Q2. Long listing dikhao aur owner, group, rwx pehchano.**
```bash
ls -l permissions.txt
# -rw-rw-r-- 1 user user 0 Jan 1 10:00 permissions.txt
```
Pehla character file type (`-` = file, `d` = directory). Phir 9 bits: owner(rw-) group(rw-) others(r--). Phir owner aur group ke naam.

**Q3. Symbolic mode: owner rwx, group rw, others kuch nahi.**
```bash
chmod u=rwx,g=rw,o= permissions.txt
ls -l permissions.txt        # -rwxrw----
```

**Q4. Numeric 640 set karo.**
```bash
chmod 640 permissions.txt
ls -l permissions.txt        # -rw-r-----
```
(r=4, w=2, x=1: 6=rw-, 4=r--, 0=---)

**Q5. Numeric 755 set karo.**
```bash
chmod 755 permissions.txt
ls -l permissions.txt        # -rwxr-xr-x
```

**Q6. Owner se execute permission hatao.**
```bash
chmod u-x permissions.txt
ls -l permissions.txt        # -rw-r-xr-x
```

**Q7. Group ko read aur execute do (owner ko chhue bina).**
```bash
chmod g+rx permissions.txt
ls -l permissions.txt
```

**Q8. Others se saari permissions hatao.**
```bash
chmod o-rwx permissions.txt      # ya chmod o= permissions.txt
ls -l permissions.txt            # last teen bits ---
```

**Q9. securedata directory banao aur 750 karo.**
```bash
mkdir securedata
chmod 750 securedata
ls -ld securedata                # drwxr-x---
```

**Q10. File aur directory ki r/w/x permissions ka fark samjhao aur demo do.**

| Permission | File par | Directory par |
|-----------|----------|---------------|
| r | contents padhna | names ki list dekhna (`ls`) |
| w | contents badalna | files banana/delete/rename karna (x bhi chahiye) |
| x | program ki tarah chalana | directory ke andar jana (`cd`) aur files access karna |

Demo (normal user se; root permissions bypass karta hai):
```bash
mkdir practice && echo hi > practice/a.txt
chmod 644 practice/a.txt

chmod u-x practice
cd practice            # Permission denied (x nahi)
ls practice            # naam dikhte hain (r hai) par ls -l error deta hai
chmod u+x practice

chmod u-r practice
ls practice            # Permission denied (r nahi)
cat practice/a.txt     # chalta hai (x hai, file ka naam pata hai)
chmod u+r practice

chmod u-w practice
touch practice/b.txt   # Permission denied (directory par w nahi)
chmod u+w practice
```

**Q11. Current umask dikhao aur note karo.**
```bash
umask          # jaise 0022 ya 0002
umask -S       # symbolic form
```

**Q12. Temporarily umask 027 karo, file/directory banao, permissions dekho.**
```bash
old=$(umask)
umask 027
touch file27
mkdir dir27
ls -ld file27 dir27
# file: 666 & ~027 = 640  -> -rw-r-----
# dir : 777 & ~027 = 750  -> drwxr-x---
```

**Q13. umask purani value par restore karo.**
```bash
umask $old
umask
```

**Q14. ACL practice ke liye naya user, group aur aclfile.txt banao.**
```bash
sudo useradd -m acluser
sudo groupadd aclgroup
touch aclfile.txt
```

**Q15. getfacl se ACL dekho aur entries pehchano.**
```bash
getfacl aclfile.txt
# file: aclfile.txt
# owner: yourname
# group: yourgroup
# user::rw-        <- owner entry
# group::rw-       <- owning group entry
# other::r--
```
Named `user:<name>:` aur `group:<name>:` entries tab aati hain jab ACL set hoti hai. `mask::` entry tab dikhti hai jab pehli extended ACL entry add hoti hai.

**Q16. Kisi user ko aclfile.txt par read+write do.**
```bash
setfacl -m u:acluser:rw aclfile.txt
getfacl aclfile.txt          # user:acluser:rw-  aur  mask::rw-
ls -l aclfile.txt            # permissions ke baad '+' lagta hai
```
Owner aur owning-group wali entries (`user::`, `group::`) nahi badalti. Dhyan rahe: `ls -l` ka group field ab mask dikhata hai.

**Q17. Kisi group ko read do.**
```bash
setfacl -m g:aclgroup:r aclfile.txt
getfacl aclfile.txt          # group:aclgroup:r--
```

**Q18. Ek ACL entry modify karo aur doosri remove karo.**
```bash
setfacl -m u:acluser:r aclfile.txt      # modify: rw -> r
getfacl aclfile.txt
setfacl -x u:acluser aclfile.txt        # remove user entry
setfacl -x g:aclgroup aclfile.txt       # remove group entry
getfacl aclfile.txt
```

**Q19. Directory par default ACL set karo; nayi file mein inherit verify karo.**
```bash
mkdir acldir
setfacl -d -m u:acluser:rw acldir        # -d = default ACL
getfacl acldir                            # default:user:acluser:rw-
touch acldir/newfile.txt
getfacl acldir/newfile.txt                # user:acluser:rw-  (inherit hua)
```
Agar `#effective:r--` dikhe to mask limit kar raha hai: `setfacl -m m::rw acldir/newfile.txt`, ya default mask ke liye `setfacl -d -m m::rw acldir`.

**Q20. Complete workflow aur cleanup.**
```bash
mkdir wf && cd wf
touch f.txt
chmod 640 f.txt && ls -l f.txt
umask 027 && touch g.txt && ls -l g.txt && umask 022
setfacl -m u:acluser:rw f.txt && getfacl f.txt
mkdir d && setfacl -d -m u:acluser:rw d && touch d/h.txt && getfacl d/h.txt
setfacl -b f.txt            # saari ACL entries hatao
setfacl -k d                # default ACL hatao
getfacl f.txt d
cd .. && rm -rf wf lab6 practice acldir securedata aclfile.txt file27 dir27
sudo userdel -r acluser
sudo groupdel aclgroup
```
