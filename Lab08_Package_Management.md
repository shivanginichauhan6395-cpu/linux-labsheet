# Lab 8: Package Management (rpm & dnf)

Environment: RHEL / Rocky Linux / AlmaLinux. Test package ke liye `tree` (aur `wget`) use kiye gaye hain; koi bhi chhota non-critical package chalega.

**Q1. RPM version dikhao aur rpm available verify karo.**
```bash
rpm --version
which rpm
```

**Q2. DNF version dikhao aur dnf available verify karo.**
```bash
dnf --version
which dnf
```

**Q3. RPM database mein package search karo, installed hai ya nahi.**
```bash
rpm -qa | grep -i bash
rpm -q bash                # installed: bash-5.x...  /  not installed: "package bash is not installed"
```

**Q4. Installed package ki detailed info.**
```bash
rpm -qi bash
```
(Name, Version, Release, Install Date, Size, License, Signature, URL, Summary, Description)

**Q5. Package ke saath install hui saari files.**
```bash
rpm -ql bash
rpm -qc bash               # sirf config files
rpm -qd bash               # sirf documentation
```

**Q6. Kaun sa package ek file ka owner hai.**
```bash
rpm -qf /usr/bin/ls        # coreutils-...
rpm -qf /etc/passwd
```

**Q7. Installation date aur baaki info.**
```bash
rpm -qi bash | grep -i "install date"
rpm -q --last bash
rpm -q --qf '%{NAME} %{VERSION}-%{RELEASE} %{INSTALLTIME:date}\n' bash
```

**Q8. Configured DNF repositories aur enabled/disabled status.**
```bash
dnf repolist               # sirf enabled
dnf repolist --all         # status column: enabled/disabled
```

**Q9. Kisi repository ki detailed info.**
```bash
dnf repoinfo baseos        # repo id aapke repolist se dekhein (jaise baseos, appstream)
```
Output mein Repo-id, Name, Status, Packages, Size, Base URL/Metalink, Last updated aati hai.

**Q10. dnf se package search.**
```bash
dnf search tree
dnf search "web server"
dnf search all httpd       # name + description dono mein
```

**Q11. dnf se package install karo aur verify karo.**
```bash
sudo dnf install -y tree
rpm -q tree
tree --version
```

**Q12. Repository se package install karo aur version rpm + dnf dono se verify karo.**
```bash
sudo dnf install -y wget
rpm -q wget
dnf list installed wget
dnf info wget
```

**Q13. Updates dikhao, apply kiye bina.**
```bash
dnf check-update           # exit code 100 = updates available, 0 = none
dnf list --upgrades
dnf updateinfo list        # security/bugfix advisories
```

**Q14. Ek selected package update karo aur new version verify karo.**
```bash
rpm -q tree                # pehle
sudo dnf upgrade -y tree   # ya dnf update tree
rpm -q tree                # baad mein
```
Agar update available nahi hai to "Nothing to do" aata hai.

**Q15. Saare packages update karo aur result verify karo.**
```bash
sudo dnf upgrade -y
dnf check-update           # ab koi update bacha nahi
dnf history | head
```
Zyada time lag sakta hai aur kernel update ho to reboot ki zaroorat padti hai.

**Q16. Test package remove karo aur verify karo.**
```bash
sudo dnf remove -y tree
rpm -q tree                # package tree is not installed
which tree
```

**Q17. Package kis repository se aata hai.**
```bash
dnf info wget              # Repository / From repo field
dnf list wget              # last column mein repo naam
dnf repoquery -i wget      # available repo wali info
dnf provides /usr/bin/wget # kaun sa package file deta hai
```

**Q18. DNF transaction history.**
```bash
dnf history
dnf history info <ID>      # kisi transaction ki detail (kya install/upgrade/remove hua)
```

**Q19. Complete workflow (rpm verify + dnf install/update/remove).**
```bash
rpm -q tree || echo "not installed"
sudo dnf install -y tree
rpm -q tree && rpm -V tree         # -V = files verify (koi output = sab theek)
sudo dnf upgrade -y tree
sudo dnf remove -y tree
rpm -q tree
dnf history | head -5
```

**Q20. Cleanup aur verify ki koi system package/repo galti se modify nahi hua.**
```bash
sudo dnf remove -y tree wget       # sirf wahi jo lab mein khud install kiye (wget pehle se tha to mat hatao)
sudo dnf clean all
ls /etc/yum.repos.d/               # koi extra .repo file nahi honi chahiye
sudo rm -f /etc/yum.repos.d/lab*.repo   # agar lab ke liye repo file banayi thi
dnf repolist --all                 # sirf original repos
dnf history | head                 # sirf lab ke transactions
```
`dnf autoremove` chalane se pehle list dhyan se padho. Wo aur packages bhi hata sakta hai.
