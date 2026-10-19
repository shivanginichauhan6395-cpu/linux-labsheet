# Lab 9: Disk & Storage Management

**Safe practice setup (loop devices):** Real disk ke bajaye file se bana virtual disk use karo, taaki system disk safe rahe.
```bash
# Q11 ke liye: test file system image (partition ke bina)
dd if=/dev/zero of=$HOME/labfs.img bs=1M count=200
sudo mkfs.ext4 -F $HOME/labfs.img

# Q17 ke liye: test disk image (partition table ke saath)
dd if=/dev/zero of=$HOME/labdisk.img bs=1M count=300
LOOP=$(sudo losetup -fP --show $HOME/labdisk.img)    # jaise /dev/loop1
echo $LOOP
```

**Q1. Mounted file systems aur disk usage.**
```bash
df
```

**Q2. Human-readable df aur fields.**
```bash
df -h
# Filesystem  Size  Used  Avail  Use%  Mounted on
```
Filesystem = device, Size = total, Used = istemal, Avail = bacha, Use% = percentage, Mounted on = mount point.

**Q3. Current directory aur contents ka size.**
```bash
du
du -s .            # sirf total
```

**Q4. Human-readable sizes aur sabse zyada space lene wali directories.**
```bash
du -h --max-depth=1 ~ | sort -hr | head
```
Sabse upar wali entries sabse zyada space leti hain.

**Q5. Selected directory ka size subdirectories ke saath.**
```bash
du -h /var/log             # har subdirectory ka size
du -sh /var/log            # sirf total
du -h --max-depth=2 /var
```

**Q6. Saare block devices, disks aur partitions.**
```bash
lsblk
lsblk -a                   # empty devices bhi
```

**Q7. lsblk se disk/partition names, sizes, types, mount points.**
```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
```
TYPE: `disk` (poori disk), `part` (partition), `loop` (loop device), `lvm`, `rom`. Mount point khali ho to wo mounted nahi hai.

**Q8. Block devices ka file-system info.**
```bash
lsblk -f
lsblk -o NAME,FSTYPE,LABEL,UUID,FSAVAIL,FSUSE%,MOUNTPOINT
```

**Q9. Mounted file systems aur mount points.**
```bash
findmnt
findmnt -t ext4,xfs        # sirf kuch types
mount | column -t | head
```

**Q10. /mnt/labdisk banao aur permissions/existence verify karo.**
```bash
sudo mkdir /mnt/labdisk
ls -ld /mnt/labdisk        # drwxr-xr-x root root
```

**Q11. Test device /mnt/labdisk par attach karo aur verify karo.**
```bash
# Option A: spare partition (jaise /dev/sdb1) ho to:   sudo mount /dev/sdb1 /mnt/labdisk
# Option B: upar bana image:
sudo mount -o loop $HOME/labfs.img /mnt/labdisk
findmnt /mnt/labdisk
lsblk -f | grep labdisk
```

**Q12. Mounted /mnt/labdisk ka usage.**
```bash
df -h /mnt/labdisk
```

**Q13. Test file banao aur verify karo ki wo isi file system par hai.**
```bash
echo "lab data" | sudo tee /mnt/labdisk/test.txt
ls -l /mnt/labdisk
df /mnt/labdisk/test.txt            # Filesystem column /dev/loopX dikhayega
findmnt -T /mnt/labdisk/test.txt
```

**Q14. umount karo aur verify karo.**
```bash
sudo umount /mnt/labdisk
findmnt /mnt/labdisk                 # koi output nahi
df -h | grep labdisk                 # koi output nahi
ls /mnt/labdisk                      # khali directory (test.txt ab nahi dikhegi)
```

**Q15. Use mein file system, unmount kyun fail hota hai, process release karke umount.**
```bash
sudo mount -o loop $HOME/labfs.img /mnt/labdisk
cd /mnt/labdisk                      # shell ab is directory ke andar hai
# doosre terminal se:
sudo umount /mnt/labdisk             # umount: /mnt/labdisk: target is busy.
sudo lsof +f -- /mnt/labdisk         # ya: sudo fuser -vm /mnt/labdisk
```
Fail isliye hota hai kyunki koi process (shell ka cwd, khuli file) mount point use kar raha hai. Hal:
```bash
cd ~                                 # shell ko bahar nikalo (ya process band karo)
sudo umount /mnt/labdisk             # ab safe
```
Zabardasti ke liye `fuser -km` ya `umount -l` hain, par pehle data loss ka khatra samajho.

**Q16. Disk ka partition table dikhao.**
```bash
sudo fdisk -l $LOOP                  # real system par: sudo fdisk -l /dev/sda
sudo parted $LOOP print
lsblk $LOOP
```

**Q17. Dedicated test/virtual disk par practice partition banao aur lsblk se verify.**
```bash
sudo parted -s $LOOP mklabel msdos mkpart primary ext4 1MiB 150MiB
# ya interactive: sudo fdisk $LOOP  ->  n, p, 1, Enter, +150M, w
sudo partprobe $LOOP
lsblk $LOOP                          # loop1 (disk) aur loop1p1 (part)
```
**System disk par kabhi mat chalao.**

**Q18. Practice partition par file system banao, /mnt/labdisk par mount karo, verify.**
```bash
sudo mkfs.ext4 ${LOOP}p1
sudo mount ${LOOP}p1 /mnt/labdisk
df -hT /mnt/labdisk
lsblk -f $LOOP
findmnt /mnt/labdisk
```

**Q19. Unmount karke mount point, device aur partition info verify.**
```bash
sudo umount /mnt/labdisk
findmnt /mnt/labdisk                 # output nahi
lsblk -f $LOOP                       # partition maujood, MOUNTPOINT khali
ls -l /mnt/labdisk                   # khali
```
Unmount se partition ya data delete nahi hota; sirf directory tree se detach hota hai.

**Q20. Complete workflow.**
```bash
lsblk; df -h                                     # inspect
sudo mkdir -p /mnt/labdisk
sudo mount ${LOOP}p1 /mnt/labdisk                # mount
df -h /mnt/labdisk                               # usage
echo "final test" | sudo tee /mnt/labdisk/final.txt
cat /mnt/labdisk/final.txt                       # verify data
sudo umount /mnt/labdisk
findmnt /mnt/labdisk; lsblk -f $LOOP             # final state
# Cleanup
sudo losetup -d $LOOP
rm -f $HOME/labfs.img $HOME/labdisk.img
sudo rmdir /mnt/labdisk
```
