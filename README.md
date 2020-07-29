# Guide to install the orbisdev toolchain and how to use it with orbislink

## Requirements

To install the orbisdev toolchain, you must meet the following requirements:

- The following packages depending of your OS:

  Arch: 

  ```bash
  sudo pacman -Sy git gcc dotnet-runtime dotnet-sdk llvm libnfs texinfo wget patch cmake clang m4 flex bison base-devel llvm-lib
  ```

  Debian/Ubuntu:

  ```bash
  wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
  sudo dpkg -i packages-microsoft-prod.deb
  sudo apt update
  sudo apt install git gcc llvm libnfs-dev libnfs-utils texinfo wget patch cmake clang m4 flex bison build-essential dotnet-runtime-3.1 dotnet-sdk-3.1
  ```

  

  - Mac OS: WIP

- HEN according to your firmware version with the following patches:

  - Patches to load resources from /data/self
  - Patch for ssc_enable_data_mount_patch
  - Obviously, patches for fself and fpkg

  Since it can be somewhat difficult to get them, I leave you a list of .bin/.elf for each version in which the above patches are already:

  * For 6.72: modified version of mira in .elf format: [download](https://github.com/OpenPS4/guide-to-install-orbisdev/raw/master/resources/hen/Mira_Orbis_672.elf)
    Thanks to Al Azif for the `ssc_enable_data_mount_patch` 6.72 offset :stuck_out_tongue_winking_eye:
    ** (Side note: remember that to load the .elf you must use the "Bin Loader" of the OpenOrbis team)
  * For 5.05: xXxTheDarkprogramerxXx fork that already includes these patches: [download](https://github.com/OpenPS4/guide-to-install-orbisdev/raw/master/resources/hen/ps4-hen-vtx-505.bin)
  * For versions lower than 5.05: I didn't find any payload done, but you can backport [this patch](https://github.com/xXxTheDarkprogramerxXx/ps4-hen-vtx/commit/854e5cf0a17db0dbaf31b89bd5b93b6b557ff0fb#diff-bfece34b95e61897401e3e6451776315R383) and it should work :smile:
  
- You will need `libScePigletv2VSH.sprx` and `libSceShaccVSH.sprx` libraries. You can get them by loading RetroArch R4 and download the libraries from the `sce_module` folder mounted in the sandbox through FTP.

- An NFS server with the following permissions: `*(sync,rw,no_subtree_check,insecure,fsid=0)` (to save you time NFS servers don't work in WSL)

- Have orbislink.pkg installed: [download](https://github.com/orbisdev/orbisdev-orbislink/raw/master/pkg/IV0003-BIGB00004_00-ORBISLINK0000000.pkg) (no need to repackage the .pkg, it's universal)

* A modified version of `orbislink_config.ini` with the NFS address and IP for debugnet. You can get it from [here](https://github.com/orbisdev/orbisdev-orbislink/blob/master/pkg/orbislink_config.ini).

## Installation of the SDK

1. Add the following variable and the bin folder to your login script (i.e ~/.bashrc)

   ```bash
   export ORBISDEV=~/orbisdev
   export PATH=$ORBISDEV/bin:$PATH
   ```

   To do this automatically you can run:

   ```bash	
   echo "export ORBISDEV=~/orbisdev" | tee -a ~/.bashrc && echo 'export PATH=$ORBISDEV/bin:$PATH' | tee -a ~/.bashrc && source ~/.bashrc
   ```

2. Clone the repository:

   ```bash
   git clone https://github.com/orbisdev/orbisdev ~/orbisdev
   ```

3. Start the installation:

   ```bash
   cd ~/orbisdev && chmod +x build-all.sh && bash build-all.sh
   ```

   In my case, I ran it on Ubuntu 20.04 with an Intel i9 9900k and it took me about 5 minutes, it's a quick install :stuck_out_tongue:

## Set up the NFS server in Ubuntu

I can't do a tutorial on how to mount an NFS server on all Linux distributions so I'll do it with Ubuntu which I think is what most of ya'll use.

1. Install the necessary packages and create the shared dir:

   ```bash
   sudo apt update && sudo apt install nfs-kernel-server
   # create the shared dir
   sudo mkdir -p ~/NFS/hostapp
   sudo chown nobody:nogroup ~/NFS/hostapp
   sudo chmod -R 777 ~/NFS/hostapp
   ```

2. Configure the NFS exports:

   ```bash
   sudo vim /etc/exports
   # we'll add a line at the bottom of the file following this stucture:
   # [NFS_SHARED_DIR_PATH] *(sync,rw,no_subtree_check,insecure,fsid=0)
   # instead of * you can just allow your PS4 IP to get more security
   # full example: /home/alfa/NFS/hostapp *(sync,rw,no_subtree_check,insecure,fsid=0)
   # remember to don't use the ~ for the path as the service is executed with root permissions!
   ```

3. Refresh the configuration and start the NFS server:

   ```bash
   sudo exportfs -arvf
   # if everything was okay, you should get an output like this: exporting *:/home/alfa/NFS/hostapp
   # and we restart the server
   sudo systemctl restart nfs-kernel-server
   
   ```

4. Remember to configure your firewalls to allow NFS server traffic. I'll not do a tutorial on it because it depends a lot on your setup.

## Compiling a example

1. Clone any example. In my case, I'll use masterzorag's example which you can download from [here](https://github.com/masterzorag/orbisdev-samples)

   1. Compile it following these steps:

      ```bash
      git clone https://github.com/masterzorag/orbisdev-samples
      cd orbis-dev-samples
      make
      make oelf
      make eboot
      # optional: just if u wanna make it pkg
      make pkg_build
      ```

      Known issues: if you get an error during the `make eboot`step check, if you're using Python 2.7. In case you aren't, follow this extra step:

      ```bash
      sudo apt install python 2.7
      sed -i 's/python/python2.7/g' Makefile
      ```

   2. Done. You compiled your first example.
      In case u wanna use it with orbislink, remember to move the `homebrew.self` into the root of your NFS shared dir!

## Installation of orbislink and loading a test app

1. Download orbislink.pkg from orbisdev [repository](https://github.com/orbisdev/orbisdev-orbislink/blob/master/pkg/IV0003-BIGB00004_00-ORBISLINK0000000.pkg).

2. Modify `orbislink_config.ini` for your setup. Leave it in an accessible place since we will use it later. You can download it from [here](https://github.com/orbisdev/orbisdev-orbislink/blob/master/pkg/orbislink_config.ini). 

   Following the upper NFS server installation, an example for it'd be like this:

   ```ini
   [debugnet]
   serverIp=192.168.2.61
   debugPort=18194
   level=3
   [orbisNfs]
   enabled=1
   nfsurl=nfs://192.168.2.61/home/alfa/NFS/hostapp
   ```

3. Check if you have following files to your shared directory:

   ```bash
   alfa@alfa:~$ tree /home/alfa/NFS/hostapp
   /home/alfa/NFS/hostapp
   ├── homebrew.self
   ├── libScePigletv2VSH.sprx
   └── libSceShaccVSH.sprx
   ```

   - In case you're missing `libScePigletv2VSH.sprx` or `libSceShaccVSH.sprx`, check [this requirement](https://github.com/OpenPS4/guide-to-install-orbisdev#requirements)
   - In case you're missing the `homebrew.self`, check [this step](https://github.com/OpenPS4/guide-to-install-orbisdev#compiling-a-example)

4. Load HEN with the patches mentioned in the requirements and install the .pkg.

5. Have a socat or netcat session ready to listen the debugnet. Examples:

   ```bash
   socat udp-recv:18194 stdout
   ```

   ```bash
   nc -ul -p 18194
   ```

4. Open orbislink app and send the config to it using this command: 

   ```bash
   # we send the config to orbislink. Remember to be in the same folder as the config is!!
   socat -u FILE:orbislink_config.ini TCP:YOURPLAYSTATIONIP:18193
   ```

5. If it was correct, after sending the config you should have had some return in the debugnet and the application should have started.

   In case you used the example we compiled above, you should get a beep. If after the first beep you get three beeps, that means something went wrong.
   By pressing the different buttons you will see in the debugnet that you get a return that they have been pressed. If you hit the triangle, you will exit the application.

   ** Remember that you must rerun the debugnet session after it loads since the socket is closed and another is opened **
   
## Contributors

This guide could not be possible without the work of:
- psxdev aka bigboss
- hitodama
- flatz
- maxton
- masterzorag
- fjtrujy
- frangar
- The rest of OrbisDev team
- Many more I'm missing!
