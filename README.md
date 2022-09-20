# wsl2-as-devbox
Personal note on setting up WSL2 as a development server

## Motivation
Some reasons why you might want to do this:

1. You have a beefy PC, but less-powerful laptop
   
    Say you already have a powerful PC, you can leverage your PC resources while working from your, say, Chromebook.

2. You like working from a Mac, but have some relatively heavy Docker workloads
   
    File system performance on Docker for Mac has always been behind Docker for Linux. There is a [long-running issue](https://github.com/docker/for-mac/issues/1592) here that eventually became [a roadmap item](https://github.com/docker/roadmap/issues/7) here.

For me, I have a pretty good gaming PC (i5-12400, 32GB RAM, EVO 970 Plus NVMe SSD) and a pretty good MacBook as well. Though I have heavy Docker use in my daily works and running Docker on my MacBook is just not pleasant. RAM usages are high and anytime something that involves I/O read/write happens in the containers (think hot reloads, builds/compilations), things will slow down. The fans in my MacBook would constantly be spinning. 

Offloading Docker to my Windows PC makes a lot of sense for my case, as I am not using my PC anyway during work hours. My MacBook fans now no longer spins in my daily work and everything simply runs much more smoothly now. Sure, I could've just worked directly on the PC, but I prefer MacBook's hardware and the OS and softwares available on the platform helps my productivity.

Even if you don't need Docker, this setup might still benefit you if you already have a beefy PC. You'd be able to leverage your powerful PC while still be able to work from anywhere using your less-powerful laptop as long as both devices are connected to the internet.

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

6. On the Windows machine, run the [`./ssh-setup.ps1`](./ssh-setup.ps1) script. This will allow the windows machine to forward incoming SSH connection to the WSL distro.

7. Verify that SSH is working by running the following commands:
   - From windows machine, try SSH-ing into WSL
     ```powershell
     ssh wsluser@windows.ip -p 2222
     ```
   - From another device in the same network, try SSH-ing into WSL
     ```sh
     ssh wsluser@windows.ip -p 2222
     ```
   Note: Replace `wsluser` with the username you created for the WSL distro. Replace `windows.ip` with the IP address assigned to your Windows PC in your local network. 

8. The `./setup-ssh.ps1` script would need to be re-run whenever the WSL is restarted, because it would be assigned a different IP. We can automate running this script whenever the windows machine boot by doing the following:
    - Press Win+R on Windows and enter shell:startup. This will open the Startup folder. Right click and create a new Shortcut.
    - Target: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command “C:\scripts\wsl-ports.ps1”
    - Try running the shortcut to make sure it works. Make sure to adjust the path to the script accordingly.
   
    Note that this only runs the script whenever Windows restarts, not when the WSL restarts. If you restart WSL manually, remember to re-run the script as well.
 
---

### Installing Docker
- [Install Docker on Windows with WSL2 backend](https://docs.docker.com/desktop/install/windows-install/)
- Once installed, go into `Settings -> Resources -> WSL Integration` and enable integration with the previously installed Ubuntu distro. This makes it so `docker` is available in the distro, even though we never installed docker into the distro ourselves.

### Optional: Move docker data to a different drive (useful to save space on `C:\`)
1. Make sure docker WSL instances are registered already.
    ```powershell
    wsl -l -v

    # There should be `docker-desktop` and `docker-desktop-data` WSL instances in the output
    # We only care about moving `docker-desktop-data`
    ```

2. Quit Docker (right click Docker icon on tray, quit Docker)

3. Shutdown the WSL instances
    ```powershell
    wsl --shutdown docker-desktop
    wsl --shutdown docker-desktop-data
    ```

4. Export `docker-desktop-data`. This is where containers and images data will reside in.
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

### Configuring WSL memory usage
By default, WSL will allocate up to 50% of your physical RAM as memory and 25% of it as swap. If you are running a lot of docker containers, this can fill up quickly. Also, WSL will keep the memory for itself eventhough the instances might not be using all that memory.

To limit the amount of memory used by WSL instances, we can configure it by creating a `.wslconfig` file in the **Windows machine** home folder. This is usually `C:\Users\UserName\.wslconfig`.

See the sample [`.wslconfig` file](./.wslconfig)

### What if I want to work outside of home?
Use [Tailscale](https://tailscale.com/) and set up the 2 devices in the same network. With this, you can simply update the IP address for SSH temporarily and still access your dev server from anywhere, as long as both devices are connected to the internet.

```shell
ssh wsluser@windows.ip.in.tailscale.network -p 2222 
```
