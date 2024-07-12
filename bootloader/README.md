Prevents unauthorized modifications to boot parameters and access to single-user mode.

Password-protect GRUB by generating a hashed password:

```bash
grub-mkpasswd-pbkdf2
```

Edit `/etc/grub.d/40_custom`

```bash
sudo nvim /etc/grub.d/40_custom
```

and add:

```bash
set superusers="admin"
password_pbkdf2 admin <hashed-password>
```

Update GRUB:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo update-grub # might work
```
