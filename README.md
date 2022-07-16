# wsl2-as-devbox
Personal note on setting up WSL 2 as a development server

## Steps
1. [Install WSL2 on Windows 10/11 machine](https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-10#1-overview)
1b. (Optional) Move WSL installation to a different drive. Useful if not much space is in `C:\`.
- Shut down the current running WSL
```powershell
wsl --shutdown
```
- Export the WSL mount image
```powershell
wsl --export Ubuntu E:\WSL\Ubuntu\ubuntu.tar 
# Replace the directory as needed. This will be the directory where the WSL distro disk will be mounted
```

- Unregister the previously installed distro
```powershell
wsl --unregister Ubuntu
```

- Import the distro from the `.tar` file we had before. This will `register` the distro again
```powershell
wsl --import Ubuntu E:\WSL\Ubuntu\Ubuntu  E:\WSL\Ubuntu\ubuntu.tar --version 2
```


- Ensure `docker-desktop-data` and `docker` WSL instances already exists
- Shut down the current running WSL
```powershell
wsl --shutdown docker-desktop-data
```

2. Allow incoming SSH connection from the local network into the WSL distro
> Source: https://medium.com/@gilad215/ssh-into-a-wsl2-host-remotely-and-reliabley-578a12c91a2

- Log into the WSL distro
```powershell
wsl
```

- Install `openssh-server` in the WSL distro
```sh
sudo apt install openssh-server
```

- Update SSH configuration in the distro
```sh
# edit /etc/ssh/sshd_config with the following three changes
Port 2222 # port 2222 is used here just so it's different than the default port 22, but still easy to remember
ListenAddress 0.0.0.0
PasswordAuthentication yes
```

- Allow starting `ssh` service without password
```sh
/etc/sudoers.d/
%sudo ALL=NOPASSWD: /usr/sbin/service ssh *
```

- Start SSH service
```sh
# start the service
service ssh start
```



- Export the WSL mount image
```powershell
wsl --export docker-desktop-data E:/Docker/wsl/data/docker-desktop-data.tar 
# Replace the directory as needed. This will be the directory where Docker's images and data are stored
```
