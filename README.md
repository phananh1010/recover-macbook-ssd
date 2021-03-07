# How to recover the data when your macbook is gone
I have been using the macbook pro for more than two years. Then one day it did not turn on. Bringing it to the store, the mechanic found out that the logic board was dead. According to his words, my laptop was too old and too expensive to repair, it was basically junk. But he could recover the data in the SSD for me for $200. I did not want to waste $200 and hand over my sensitive data to a stranger, so I took my dead laptop home and recovered the data by myself. I succeeded. My SSD is now turned into a flask disk which I can access via USB port, a 500GB SSD flash disk! For future reference, I detail the steps to turn the SSD in Macbook into a flask disk. Total cost to recover the data is less than $80 and it took me less than two hours to finish all steps. Totally worth it.

To retrieve my data, I bought an SSD enclosure for MacBook. Then I removed the SSD in my dead laptop and inserted it to the enclosure. The enclosure with SSD is now literally a USB flash disk. Now is the most challenging part, to make the desktop recognize the SSD flash disk.

Apple has been using APFS, a proprietary file system, to manage files and directories. Since this is a private standard, other OSs could not simply read the data in the flask disk, they don't know how to do it. There are two simple steps to solve it. First, install the [APFS driver](https://github.com/sgan81/apfs-fuse) for Linux. Second, mount the volume.



## Prepare the hardwares
Depending on the Macbook model, the SSD Enclosure may be different. In my case, I used the [ACASIS USB C 3.0 Enclosure](https://www.amazon.com/gp/product/B08B634C7C/) which works for laptops from Mid-2013 and later.

All Macbooks use special screws which can only be opened with special scewdrivers. However, it is very easy to obtain them from Amazon.

After everything is ready, the SSD can be extracted from the laptop. It is quite easy to remove the SSD, simply following the instructions from [ifixit](https://www.ifixit.com/Guide/MacBook+Pro+13-Inch+Retina+Display+Late+2013+SSD+Replacement/26811).

## How to mount the macOS APFS disk volumes in Linux
I used Linux to retrieve my data. As of the time I am writing this document, it is not clear if Windows can read an APFS volume or not.

### Install the APFS driver
Use the following commands to install the APFS driver.
sudo apt update
sudo apt install fuse libfuse-dev libicu-dev bzip2 cmake libz-dev libbz2-dev clang git libattr1-dev

After this command, there will be an error indicating that the `fuse` package is not found. This error persists for Ubuntu 18 and lower versions. We can work around this when compiling the program at the later step. For now, download the APFS Driver source code from the Github repository.
```
git clone https://github.com/sgan81/apfs-fuse.git
cd apfs-fuse
git submodule init
git submodule update
```
Then try to compile it.
```
mkdir build
cd build
cmake ..
make
```
And error happens after the make command is executed. Something like this: `fatal error: fuse3/fuse.h: No such file or directory`. To work around, install `ccmake` (if it is not installed) and change the compile configuration, so that `fuse 3.0` is not used.
```
sudo apt install cmake-curses-gui
ccmake .
Use arrow and change USE_FUSE3 to OFF, press Enter.
Press c to configure
Press g to generate the Makefile
Press q to exit ccmake
Execute make again, the previous error should disappear.
```


To make it convenient, the APFS program can be registered so that the full file path is not needed every time `apfs` is executed. To resiter the program, copy the executable binaries into the local bin directory.
```
sudo cp apfs-* /usr/local/bin
```

### Mount the device
If all previous steps are successful, the flask disk containing the SSD should be recognize by the OS. To verify, list all the disk volumes by typing
```
fdisk -l
```
There will be one line from the result showing a Device of unknown Type. Mark the file path to the dev directory for this device. Then mount the device to a directory.
```
sudo mkdir -p /media/$USERNAME/macssd
sudo ./apfs-fuse -o allow_other /dev/<device file name> /media/$USERNAME/macssd
```
Replace <device file name> with the name associated with device of unknown type.

If everything goes smoothly, a new drive icon will appear on the desktop. Data in the SSD can be accessed via the drive icon.

### Conclusion
The time and cost to recover data in the SSD of a dead Macbook is not high, much cheaper than asking an expert. I bought the SSD Enclosure from Amazon for $70 and a Screwdriver set to open the macbook for $6. The whole recovery procedure, from retrieving the SSD to installing the APFS driver, is quite simple for a normal Linux user.

## Reference
This small project refer to the following materials/instructions:
[1] [MacBook Pro 13 Inch Retina Display Late 2013 SSD Replacement](https://www.ifixit.com/Guide/MacBook+Pro+13-Inch+Retina+Display+Late+2013+SSD+Replacement/26811)

[2] [How to mount macos apfs disk volumes in Linux](https://linuxnewbieguide.org/how-to-mount-macos-apfs-disk-volumes-in-linux/)

[3] [apfs-fuse issues No. 87 - fatal error: fuse3/fuse.h No such file or directory
](https://github.com/sgan81/apfs-fuse/issues/87)
