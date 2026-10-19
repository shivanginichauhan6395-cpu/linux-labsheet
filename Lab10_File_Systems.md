# Lab 10: File Systems

**Setup:** Poore lab mein ek **dedicated test partition** use hoga. Yahan `DEV=/dev/loop1p1` (Lab 9 ka loop partition) maana gaya hai. Apne test partition (jaise USB `/dev/sdb1`) se badal lo.
```bash
dd if=/dev/zero of=$HOME/lab10.img bs=1M count=300
LOOP=$(sudo losetup -fP --show $HOME/lab10.img)
sudo parted -s $LOOP mklabel msdos mkpart primary 1MiB 250MiB
sudo partprobe $LOOP
DEV=${LOOP}p1
echo $DEV
```
**Warning:** `mkfs` device ka saara data mita deta hai. System partition par kabhi mat chalao.

**Q1. Block devices dikhao aur test partition pehchano.**
```bash
lsblk
lsblk -f
```
Wo partition chuno jo naya/khali ho aur jiska MOUNTPOINT khali ho (system partition `/`, `/boot` nahi).

**Q2. Test partition ka file-system type, UUID, mount-point info.**
```bash
lsblk -f $DEV
sudo blkid $DEV
findmnt -S $DEV            # mounted ho tabhi output aata hai
```
Abhi format nahi hua to FSTYPE/UUID khali honge.

**Q3. ext4 file system banao aur verify karo.**
```bash
sudo mkfs.ext4 $DEV
lsblk -f $DEV              # FSTYPE = ext4
sudo blkid $DEV
sudo file -s $DEV
```

**Q4. Naye file system ki detailed info aur UUID.**
```bash
sudo dumpe2fs -h $DEV | head -20
sudo tune2fs -l $DEV | grep -i -E 'UUID|name|block count'
sudo blkid -s UUID -o value $DEV
```

**Q5. /mnt/labfs mount point banao.**
```bash
sudo mkdir /mnt/labfs
ls -ld /mnt/labfs
```

**Q6. Manually mount karo aur verify karo.**
```bash
sudo mount $DEV /mnt/labfs
findmnt /mnt/labfs
mount | grep labfs
```

**Q7. df se total, used, available space.**
```bash
df -h /mnt/labfs
# Size = total,  Used = istemal,  Avail = bacha
```
Chhote ext4 par "Used" thoda dikhta hai kyunki `lost+found` aur metadata space leta hai.

**Q8. Test file banao aur verify karo ki mounted file system par hai.**
```bash
echo "hello labfs" | sudo tee /mnt/labfs/test.txt
ls -l /mnt/labfs
df /mnt/labfs/test.txt
findmnt -T /mnt/labfs/test.txt
```

**Q9. umount karo aur verify karo.**
```bash
sudo umount /mnt/labfs
findmnt /mnt/labfs                  # output nahi
mountpoint /mnt/labfs               # "/mnt/labfs is not a mountpoint"
ls /mnt/labfs                       # khali
```

**Q10. /etc/fstab ka content aur persistent mount ke fields.**
```bash
cat /etc/fstab
```
Har line ke 6 fields:
1. **Device** (`UUID=...`, `/dev/...`, `LABEL=...`)
2. **Mount point** (`/mnt/labfs`)
3. **File-system type** (`ext4`, `xfs`, ...)
4. **Mount options** (`defaults`, `noatime`, `nofail`, ...)
5. **Dump** (0 = backup nahi)
6. **fsck pass** (0 = check nahi, 1 = root, 2 = baaki)

**Q11. UUID se /etc/fstab mein entry add karo.**
```bash
sudo cp /etc/fstab /etc/fstab.bak                 # pehle backup
UUID=$(sudo blkid -s UUID -o value $DEV)
echo "UUID=$UUID  /mnt/labfs  ext4  defaults,nofail  0  2" | sudo tee -a /etc/fstab
tail -2 /etc/fstab
```
`nofail` se boot mein dikkat nahi aati agar device na mile (loop/removable devices ke liye zaroori).

**Q12. Bina reboot ke syntax verify aur test.**
```bash
sudo findmnt --verify              # fstab syntax/device check
sudo systemctl daemon-reload       # systemd ko naya fstab padhwao
sudo mount -a                      # fstab ke saare entries mount (error aaye to turant dikhega)
findmnt /mnt/labfs
```

**Q13. Reboot/remount karke automatic mount verify.**
```bash
sudo umount /mnt/labfs
sudo mount -a                      # fstab se auto mount
findmnt /mnt/labfs
# Real partition par: sudo reboot  ->  login ke baad findmnt /mnt/labfs
```
(Loop device reboot ke baad apne aap nahi banta, isliye loop test mein `mount -a` se simulate karo.)

**Q14. findmnt aur df se verify.**
```bash
findmnt /mnt/labfs                 # SOURCE, TARGET, FSTYPE, OPTIONS
df -hT /mnt/labfs                  # Type, Size, Used, Avail, Use%
```

**Q15. File-system UUID dikhao aur fstab wali UUID se compare karo.**
```bash
sudo blkid $DEV
lsblk -no UUID $DEV
grep labfs /etc/fstab
```
Dono UUID bilkul same honi chahiye. Mismatch ho to mount fail hoga.

**Q16. Safely unmount karke fsck chalao.**
```bash
sudo umount /mnt/labfs
sudo e2fsck -f $DEV                # -f = clean ho tab bhi force check
echo $?                            # 0 = no errors, 1 = errors corrected
```
Sirf dekhne ke liye (kuch fix na kare): `sudo e2fsck -fn $DEV`.

**Q17. Mounted vs unmounted file system check ka fark.**
```bash
# Mounted:
sudo mount $DEV /mnt/labfs
sudo e2fsck -f $DEV                # "Cannot continue, aborting" (mounted par refuse)
sudo e2fsck -fn $DEV               # sirf read-only (-n); "Warning! ... is mounted" aata hai, results bharose ke nahi
# Unmounted:
sudo umount /mnt/labfs
sudo e2fsck -f $DEV                # poora safe check
```
Mounted FS par check/repair karna khatarnak hai kyunki kernel us par likh raha hota hai, isse corruption ho sakta hai aur galat errors dikh sakte hain. Precautions: sirf test partition, pehle unmount, backup, aur mounted par repair (`-y`/`-p`) kabhi nahi. (XFS ke liye `xfs_repair`, wo bhi unmounted par.)

**Q18. Galat fstab entry ko theek karo.**
```bash
sudo umount /mnt/labfs 2>/dev/null
# galat UUID jaan-bujhkar:
sudo sed -i 's|^UUID=[^ ]*\(  */mnt/labfs\)|UUID=1111-WRONG\1|' /etc/fstab
sudo mount -a                      # error: can't find UUID=1111-WRONG
sudo findmnt --verify              # galti dikhata hai
# Theek karo:
UUID=$(sudo blkid -s UUID -o value $DEV)
sudo sed -i "s|^UUID=[^ ]*\(  */mnt/labfs\)|UUID=$UUID\1|" /etc/fstab
grep labfs /etc/fstab
sudo systemctl daemon-reload
sudo mount -a
findmnt /mnt/labfs                 # ab mount ho gaya
```
Isiliye reboot se pehle `mount -a` zaroor chalao. Galat fstab ke baad system emergency mode mein ja sakta hai.

**Q19. Unmount, fstab entry hatao, verify ki auto mount nahi hota.**
```bash
sudo umount /mnt/labfs
sudo sed -i '\|/mnt/labfs|d' /etc/fstab        # sirf lab wali line hatao (ya: sudo cp /etc/fstab.bak /etc/fstab)
grep labfs /etc/fstab                          # koi output nahi
sudo systemctl daemon-reload
sudo mount -a
findmnt /mnt/labfs                             # koi output nahi (mount nahi hua)
```

**Q20. Complete workflow aur cleanup.**
```bash
sudo mkfs.ext4 $DEV                                    # 1. create
sudo mkdir -p /mnt/labfs && sudo mount $DEV /mnt/labfs # 2. mount
df -hT /mnt/labfs && findmnt /mnt/labfs                # 3. verify
UUID=$(sudo blkid -s UUID -o value $DEV)
sudo cp /etc/fstab /etc/fstab.bak
echo "UUID=$UUID /mnt/labfs ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab   # 4. fstab
sudo findmnt --verify; sudo systemctl daemon-reload
sudo umount /mnt/labfs && sudo mount -a && findmnt /mnt/labfs
sudo umount /mnt/labfs
sudo e2fsck -f $DEV                                    # 5. safe check
# 6. cleanup
sudo sed -i '\|/mnt/labfs|d' /etc/fstab
sudo systemctl daemon-reload
sudo rmdir /mnt/labfs
sudo rm -f /etc/fstab.bak
sudo losetup -d $LOOP && rm -f $HOME/lab10.img          # loop test cleanup
grep labfs /etc/fstab; findmnt /mnt/labfs               # dono khali
```
