# Installation instructions

1. Download `zip` archive of the `main` branch
2. `sudo apt install unzip -y`
3. `unzip amdgpufan-main.zip`
4. `cd amdgpufan-main`
5. `cp amdgpufan /usr/bin/`
6. `cp amdgpufan.service /etc/systemd/system/`
7. `sudo systemctl daemon-reload`
8. `sudo systemctl start amdgpufan`
9. `sudo systemctl enable amdgpufan`
10. `sudo systemctl status amdgpufan`

