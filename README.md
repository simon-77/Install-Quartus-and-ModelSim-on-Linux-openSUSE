# Install-Quartus-and-ModelSim-on-Linux-openSUSE
Tutorial: Installation of Intel Quartus Prime Lite &amp; ModelSim Starter Edition on Linux openSUSE Tumbleweed

I have used the following versions for this tutorial:
- Quartus Prime Lite Edition 19.1
- Operating System: OpenSUSE Tumbleweed (Kernel: 5.3.7-1-default) (2019-11)

## Download and install Quartus Prime
Download link: <http://fpgasoftware.intel.com/?edition=lite>
Install Quartus Prime by executing the downloaded installer (execute with `sudo`).
I choose the following installation path: `/opt/intelFPGA_lite/19.1/`

# Issues with Quartus Prime
Start Quartus by executing:
`/opt/intelFPGA_lite/19.1/quartus/bin/quartus`

--TODO:
USB ...

# Issues with ModelSim
Start ModelSim by executing:
`/opt/intelFPGA_lite/19.1/modelsim_ase/bin/vsim`

## Error:
`cant load lib...`
### Issue
ModelSim needs certain 32-bit packages
### Solution
Install the following 32-bit packages (with YaST or zypper):
- libXft2-32bit
- libncurses5-32bit

## Error:
`Error: cannot find "/opt/intelFPGA_lite/19.1/modelsim_ase/bin/../linux_rh60/vsim"`
### Issue
ModelSim does not support the current kernel version
### Solution
Modify the file `/opt/intelFPGA_lite/19.1/modelsim_ase/vco` (with root rights)
At about line 200 at the `case $utype in` statement add a line that matches your kernel:
`5.[0-9]*)         vco="linux" ;;`

## Error:
`"ncFyP12 -+"
    (file "/mtitcl/vsim/vsim" line 1)
** Fatal: Read failure in vlm process (0,0)`
### Issue
The library freetype2 is too new and will not work with vsim. Additionally the old freetype2 library also requires an old fontconfig library.
### Solution
Add this library manually to Modelsim. Either download the source files and compile it your own or download the folder with already compiled binaries.

The instruction for compiling the binaries your own is located at the bottom of this page. Otherwise, follow these steps:

1) Download `fontconfig-2.12.4-32bit` and `freetype-2.4.7-32bit`
Copy them either to your home directory or create a new folder and copy them to: `/opt[...]/modelsim_ase/bin32`
I have stored them under `/home/simon/etc/packages`

2) Add the new path to the variable `LD_LIBRARY_PATH` for ModelSim.
Modify the file `/opt/intelFPGA_lite/19.1/modelsim_ase/vco` (with root rights) and add the following lines to the beginning of the file :
`
export LD_LIBRARY_PATH=/home/simon/etc/packages/freetype-2.4.7-32bit/lib/:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/home/simon/etc/packages/fontconfig-2.12.4-32bit/lib/:$LD_LIBRARY_PATH
`

## Desktop Icons
If you want to have desktop icons to start Quartus and ModelSim you can download them and add them to your Desktop folder.

## Environment Variables
Some programms need certain environment variables that point to the binaries they can execute. The following environment variables worked for me.

Add these lines to your `~/.bashrc` file:
`
QUARTUS_PATH="/opt/intelFPGA_lite/19.1/quartus"
MODELSIM_PATH="/opt/intelFPGA_lite/19.1/modelsim_ase"

QUARTUS_BIN="$QUARTUS_PATH/bin"
MODELSIM_BIN="$MODELSIM_PATH/bin"
PATH="$PATH:$QUARTUS_BIN"
PATH="$PATH:$MODELSIM_BIN"
export PATH
export QSYS_ROOTDIR="$QUARTUS_PATH/sopc_builder/bin"
`

## Instruction for compiling the 32-bit packages required by ModelSim your own
1) Install the following packages on your system with YaST or zypper:
- gcc-32bit
- libexpat-devel-32bit
- gperf

2) Download the old Source Packages (freetype-2.4.7 & fontconfig-2.12.4)
<https://download.savannah.gnu.org/releases/freetype/>
<http://www.linuxfromscratch.org/blfs/view/8.1/general/fontconfig.html>

3) Compile them with make
Set the variable CFLAGS to `-m32` for compiling it to a 32-bit package. Additionally set the prefix to some directory in your home directory to install it there because you don't want to install it to your operating system.
`$ CFLAGS=-m32 ./configure --prefix=/home/simon/etc/packages/freetype-2.4.7-32bit`
`$ make install`

Set the same variables as before and additionally add the path to the previousely compiled freetype package to the variable `PKG_CONFIG_PATH`
`$ PKG_CONFIG_PATH=/home/simon/etc/packages/freetype-2.4.7-32bit/lib/pkgconfig:PKG_CONFIG_PATH CFLAGS=-m32 ./configure --prefix=/home/simon/etc/packages/fontconfig-2.12.4-32bit`
`$ make install`


# Resources
<https://wiki.archlinux.org/index.php/Altera_Design_Software>
<https://twoerner.blogspot.com/2017/10/running-modelsim-altera-from-quartus.html>
