# Tutorial: Installation of Intel Quartus Prime Lite & ModelSim Starter Edition on Linux openSUSE Tumbleweed
I have used the following versions for this tutorial:
- Quartus Prime Lite Edition 19.1
- Operating System: OpenSUSE Tumbleweed (Kernel: 5.3.7-1-default) (2019-11)


## Download and install Quartus Prime
Download link: <http://fpgasoftware.intel.com/?edition=lite>

Install Quartus Prime by executing the installer (execute with `sudo`).
I choose the following installation path: `/opt/intelFPGA_lite/19.1/`

There are some Issues you have to solve when using Quartus Prime and ModelSim under Linux. If you want to apply the changes of this tutorial then just download this repository and use the files from there.

# Issues with Quartus Prime
Start Quartus by executing:
```
/opt/intelFPGA_lite/19.1/quartus/bin/quartus
```

## Error:
When you start Quartus Programmer it will give throw the following error:
```
Error (209053): Unexpected error in JTAG server -- error code 89
```
### Issue
You have to install the Programming Cable Drivers. Follow the Quartus Installation Manual (Section: *3.7. Step 3: Install Programming Cable Drivers*):
<https://www.intel.com/content/dam/www/programmable/us/en/pdfs/literature/manual/quartus_install.pdf>

If you don't want to read the manual, then follow this solution:
### Solution
1) Copyt the file `51-usbblaster.rules` (provided by this repository) to `/etc/udev/rules.d/51-usbblaster.rules`
1) change the owner of this file to root (`# chown root:root 51-usbblaster.rules`)
1) Reboot your system



# Issues with ModelSim
Start ModelSim by executing:
```
/opt/intelFPGA_lite/19.1/modelsim_ase/bin/vsim
```

## Error:
```
cant load lib...
```
### Issue
ModelSim needs certain 32-bit packages
### Solution
Install the following 32-bit packages (with YaST or zypper):
- libXft2-32bit
- libncurses5-32bit

## Error:
```
Error: cannot find "/opt/intelFPGA_lite/19.1/modelsim_ase/bin/../linux_rh60/vsim"
```
### Issue
ModelSim does not support the current kernel version
### Solution
Modify the file `/opt/intelFPGA_lite/19.1/modelsim_ase/vco` (with root rights)
At about line 200 at the `case $utype in` statement add a line that matches your kernel:
`5.[0-9]*)         vco="linux" ;;`

## Error:
```
"ncFyP12 -+"
    (file "/mtitcl/vsim/vsim" line 1)
** Fatal: Read failure in vlm process (0,0)
```
### Issue
The library *freetype2* is too new and will not work with vsim. Additionally the old *freetype2* library also requires an old fontconfig library.
### Solution
Add this library manually to Modelsim. Either download the source files and compile it your own or download the folder with already compiled binaries.

The instruction for compiling the libraries your own is located at the bottom of this page. If you want to download the already compiled binaries then follow these steps:

1) Extract the archives `fontconfig-2.12.4-32bit.tar.gz` and `freetype-2.4.7-32bit.tar.gz` provided by this repository.
Copy them either to your home directory or to the folder `/opt[...]/modelsim_ase/bin32` (you have to create this folder) - *I have stored them under `/home/simon/etc/packages`*

2) Add the new path to the variable `LD_LIBRARY_PATH` for ModelSim.
Modify the file `/opt/intelFPGA_lite/19.1/modelsim_ase/vco` (with root rights) and add the following lines to the beginning of the file :
```
export LD_LIBRARY_PATH=/home/simon/etc/packages/freetype-2.4.7-32bit/lib/:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/home/simon/etc/packages/fontconfig-2.12.4-32bit/lib/:$LD_LIBRARY_PATH
```

## Desktop Icons
If you want to have desktop icons to start Quartus and ModelSim you can copy the `.desktop` files provided by this repository to your Desktop folder.
- `Quartus Prime 19.1 Lite Edition.desktop`
- `ModelSim - Quartus 19.1.desktop`

## Environment Variables
Some programms require certain environment variables that point to the path of the binaries. The following environment variables worked for me.

Add these lines to your `~/.bashrc` file:
```
QUARTUS_PATH="/opt/intelFPGA_lite/19.1/quartus"
MODELSIM_PATH="/opt/intelFPGA_lite/19.1/modelsim_ase"

QUARTUS_BIN="$QUARTUS_PATH/bin"
MODELSIM_BIN="$MODELSIM_PATH/bin"
PATH="$PATH:$QUARTUS_BIN"
PATH="$PATH:$MODELSIM_BIN"
export PATH
export QSYS_ROOTDIR="$QUARTUS_PATH/sopc_builder/bin"
```

## Instruction for compiling the 32-bit packages required by ModelSim your own
1) Install the following packages on your system with YaST or zypper:
- gcc-32bit
- libexpat-devel-32bit
- gperf

2) Download the old Source Packages (freetype-2.4.7 & fontconfig-2.12.4)
- <https://download.savannah.gnu.org/releases/freetype/>
- <http://www.linuxfromscratch.org/blfs/view/8.1/general/fontconfig.html>

3) Compile them with make

Set the variable CFLAGS to `-m32` for compiling it to a 32-bit package. Additionally set the prefix to some directory in your home directory to install it there because you don't want to install it to your operating system.
### freetype-2.4.7
```
$ CFLAGS=-m32 ./configure --prefix=/home/simon/etc/packages/freetype-2.4.7-32bit
$ make install
```
Set the same variables as before and additionally add the path to the previousely compiled freetype package to the variable `PKG_CONFIG_PATH`
### fontconfig-2.12.4
```
$ PKG_CONFIG_PATH=/home/simon/etc/packages/freetype-2.4.7-32bit/lib/pkgconfig:PKG_CONFIG_PATH CFLAGS=-m32 ./configure --prefix=/home/simon/etc/packages/fontconfig-2.12.4-32bit
$ make install
```


# Resources
- <https://twoerner.blogspot.com/2017/10/running-modelsim-altera-from-quartus.html>
- <https://wiki.archlinux.org/index.php/Altera_Design_Software>
