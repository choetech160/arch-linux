# What is Lynis
[Lynis](https://github.com/CISOfy/Lynis) is a security auditing tool for systems based on UNIX. It performs an in-depth security scan and runs on the system itself. The primary goal is to test security defenses and provide tips for further system hardening. 

## Install
```bash
pacman -S lynis 
```

## System audit
```bash
lynis audit system
```

After the audit is complete, Lynis will provide detailed findings and recommendations for improving the security posture of your Arch Linux system. The results will be categorized into different sections such as file permissions, software packages, and system configuration
