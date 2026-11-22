# ubuntu-to-rocky

deb2rpm is a Python script that intercepts Ubuntu-style commands and translates them into their Rocky Linux equivalents, executing them immediately.

Installation (on a Rocky server)
bash
# Rocky 8 may not have python3 installed (it is included by default in Rocky 9)
sudo dnf install -y python3

chmod +x deb2rpm
sudo ./deb2rpm --install

Once installed, symbolic links are created in /usr/bin—mapping names like `apt`, `dpkg`, and `ufw` to `deb2rpm`. From that point on, you can simply enter the same commands you would use on Ubuntu. These links are placed in /usr/bin because the `sudo` configuration in RHEL-based systems does not include /usr/local/bin in its search path.

Usage Example
bash
sudo apt update && sudo apt install -y build-essential libssl-dev apache2
# → sudo dnf makecache && { sudo dnf group install -y 'Development Tools' && sudo dnf install -y openssl-devel httpd; }

sudo ufw allow 'Nginx Full'
# → sudo firewall-cmd --permanent --add-service=http && ... --add-service=https && ... --reload

sudo systemctl restart apache2    # → httpd
sudo update-grub                  # → grub2-mkconfig -o /boot/grub2/grub.cfg
Usage Method	Description
apt install nginx	Enter the command as is after installation
deb2rpm "apt update && apt upgrade"	Direct invocation (enclose multiple commands in quotes)
deb2rpm -n apt install nginx	Show the conversion result only (do not execute)
deb2rpm -c ...	Confirm before execution
deb2rpm	Interactive mode (enter Ubuntu command → convert → execute)
sudo deb2rpm --uninstall	Uninstall
What gets converted
Package Management: apt, apt-get, apt-cache, aptitude, apt-mark, dpkg, add-apt-repository, and apt-key are converted to dnf/rpm.
Package Names: apache2 becomes httpd, -dev becomes -devel, build-essential becomes the "Development Tools" group, and php8.1-xxx becomes php-xxx.
EPEL: For EPEL packages like htop, fail2ban, and certbot, epel-release is installed automatically.
Firewall: ufw is converted to firewalld (firewall-cmd). It supports port, service, and IP-based rules, as well as rule deletion.
System Administration: Supports adduser, deluser, addgroup, update-alternatives, lsb_release, update-grub, update-rc.d, invoke-rc.d, and netplan.
Service Names and Paths: apache2 becomes httpd, ssh becomes sshd, /var/log/syslog becomes /var/log/messages, /etc/apache2 becomes /etc/httpd, etc. Shell syntax: Elements such as `sudo`, `&&`, `||`, `;`, `|`, and redirections like `2>/dev/null` are preserved as-is.
Limitations
`.deb` installation: `dpkg -i x.deb` and PPA-based operations are not supported on RPM-based systems. In such cases, an informational message is displayed, and the program exits with an exit code of 1.
Package names: Package names not found in the mapping table are either passed directly to `dnf` or converted using the `-dev` to `-devel` rule. Consequently, some packages may need to be located manually using `dnf search`.
`-devel` packages: If a `-devel` package cannot be found, please run `sudo dnf config-manager --set-enabled crb` (or `powertools` if using Rocky 8).
`adduser`: Since RHEL includes an `adduser` command that links to `useradd`, no additional symbolic link is created. To replicate Ubuntu-style `adduser` behavior, please use the command `deb2rpm adduser [name]`.

Please let me know if any conversions are incorrect or if any commands are missing.
