# wsl2-as-devbox
Personal note on setting up WSL 2 as a development server

## Steps

Before doing anything, first [install WSL2 on Windows 10/11 machine](https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-10#1-overview)

### (Optional) Move WSL installation to a different drive. Useful if not much space is in `C:\`.
1. Shut down the current running WSL
    ```powershell
    wsl --shutdown
    ```

2. Export the WSL mount image
    ```powershell
    wsl --export Ubuntu E:\WSL\Ubuntu\ubuntu.tar 
    # Replace the directory as needed. This will be the directory where the WSL distro disk will be mounted
    ```

3. Unregister the previously installed distro
    ```powershell
    wsl --unregister Ubuntu
    ```

4. Import the distro from the `.tar` file we had before. This will `register` the distro again
    ```powershell
    wsl --import Ubuntu E:\WSL\Ubuntu\Ubuntu  E:\WSL\Ubuntu\ubuntu.tar --version 2
    ```

5. Ensure `docker-desktop-data` and `docker` WSL instances already exists

6. Shut down the current running WSL
    ```powershell
    wsl --shutdown docker-desktop-data
    ```

---

### Allow incoming SSH connection from the local network into the WSL distro
> Source: https://medium.com/@gilad215/ssh-into-a-wsl2-host-remotely-and-reliabley-578a12c91a2

1. Log into the WSL distro
    ```powershell
    wsl
    ```

2. Install `openssh-server` in the WSL distro
    ```sh
    sudo apt install openssh-server
    ```

3. Update SSH configuration in the distro
    ```sh
    # edit /etc/ssh/sshd_config with the following three changes
    Port 2222 # port 2222 is used here just so it's different than the default port 22, but still easy to remember
    ListenAddress 0.0.0.0
    PasswordAuthentication yes
    ```

4. Allow starting `ssh` service without password
    ```sh
    /etc/sudoers.d/
    %sudo ALL=NOPASSWD: /usr/sbin/service ssh *
    ```

5. Start SSH service
    ```sh
    # start the service
    service ssh start
    ```

6. On the Windows machine, run the [`./setup-ssh.ps1`](./setup-ssh.ps1) script. This will allow the windows machine to forward incoming SSH connection to the WSL distro.

7. Verify that SSH is working by running the following commands:
   - From windows machine, try SSH-ing into WSL
     ```powershell
     ssh wsluser@windows.ip -p 2222
     ```
   - From another device in the same network, try SSH-ing into WSL
     ```sh
     ssh wsluser@windows.ip -p 2222
     ```

8. The `./setup-ssh.ps1` script would need to be re-run whenever the WSL is restarted, because it would be assigned a different IP. We can automate running this script whenever the windows machine boot by doing the following:
    - Press Win+R on Windows and enter shell:startup. This will open the Startup folder. Right click and create a new Shortcut.
    - Target: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command “C:\scripts\wsl-ports.ps1”
    - Try running the shortcut to make sure it works. Make sure to adjust the path to the script accordingly.

---

### Installing Docker
- [Install Docker on Windows with WSL2 backend](https://docs.docker.com/desktop/install/windows-install/)
- Once installed, go into `Settings -> Resources -> WSL Integration` and enable integration with the previously installed Ubuntu distro. This makes it so `docker` is available in the distro, even though we never installed docker into the distro ourselves.

### Optional: Move docker data to a different drive (useful to save space on `C:\`)
1. Make sure docker WSL instances are registered already.
    ```powershell
    wsl -l -v

    # There should be `docker` and `docker-desktop-data` WSL instances in the output
    ```

2. Quit Docker (right click Docker icon on tray, quit Docker)

3. Shutdown the WSL instances
    ```powershell
    wsl --shutdown docker
    wsl --shutdown docker-desktop-data
    ```

4. Export `docker-desktop-data`
    ```powershell
    wsl --export docker-desktop-data E:\Docker\wsl\data\docker-desktop-data.tar 
    # Replace the directory as needed. This will be the directory where Docker's images and data are stored
    ```

5. Unregister `docker-desktop-data`
    ```powershell
    wsl --unregister docker-desktop-data
    ```

6. Import
    ```powershell
    wsl --import docker-desktop-data E:\Docker\wsl\data E:\Docker\wsl\data\docker-desktop-data.tar  --version 2
    ```

7. Start Docker desktop again, everything should work.


### What if I want to work outside of home?
Use [Tailscale](https://tailscale.com/) and set up the 2 devices in the same network. With this, you can simply update the IP address for SSH temporarily and still access your dev server from anywhere, as long as both devices are connected to the internet.
