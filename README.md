## Guide to install the orbisdev toolchain and how to use it with orbislink.

## Requirements

To install the orbisdev toolchain, you must meet the following requirements:

- The following packages depending of your OS:

  - Arch: `sudo pacman -S git gcc dotnet-runtime dotnet-sdk llvm libnfs texinfo wget patch cmake clang m4 flex bison base-devel llvm-lib`

  - Debian/Ubuntu:

    ```bash
    wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb
    sudo apt update
    sudo apt install git gcc llvm libnfs-dev libnfs-utils texinfo wget patch cmake clang m4 flex bison build-essential dotnet-runtime-3.1 dotnet-sdk-3.1
    ```

    

  - Mac OS

- HEN according to your firmware version with the following patches:

  - Patches to load resources from /data/self
  - Patch for ssc_enable_data_mount_patch
  - Obviously, patches for fself and fpkg

  Since it can be somewhat difficult to get them, I leave you a list of .bin/.elf for each version in which the above patches are already:

  * For 6.72: modified version of mira in .elf format: [download](https://github.com/OpenPS4/guide-to-install-orbisdev-toolchain/resources/hen/Mira_Orbis_672.elf)
    Thanks to Al Azif for the `ssc_enable_data_mount_patch` 6.72 offset :stuck_out_tongue_winking_eye:
    ** (Side note: remember that to load the .elf you must use the "Bin Loader" of the OpenOrbis team)
  * For 5.05: xXxTheDarkprogramerxXx fork that already includes these patches: [download](https://github.com/OpenPS4/guide-to-install-orbisdev-toolchain/resources/hen/ps4-hen-vtx-505.bin)
  * For versions lower than 5.05: I didn't find any payload done, but you can backport [this patch](https://github.com/xXxTheDarkprogramerxXx/ps4-hen-vtx/commit/854e5cf0a17db0dbaf31b89bd5b93b6b557ff0fb#diff-bfece34b95e61897401e3e6451776315R383) and it should work :smile:
  
- An NFS server with the following permissions: `*(sync,rw,no_subtree_check,insecure,fsid=0)` (to save you time NFS servers don't work in WSL)

- Have orbislink.pkg installed: [download](https://github.com/orbisdev/orbisdev-orbislink/raw/master/pkg/IV0003-BIGB00004_00-ORBISLINK0000000.pkg) (no need to repackage the .pkg, it's universal)

* A modified version of `orbislink_config.ini` with the NFS address and IP for debugnet. You can get it from [here](https://github.com/orbisdev/orbisdev-orbislink/blob/master/pkg/orbislink_config.ini).

## Installation

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

4. Time to test it!