# <p align="center"><b>Coreboot Configuration for Lenovo x220</b></p>


## <b>Reference</b>
  * <a href="https://www.coreboot.org/Board:lenovo/x220">Coreboot Wiki x220</a>
  * <a href="https://github.com/michaelmob/x220-coreboot-guide">Michael Mob's Guide</a>
  * <a href="https://tylercipriani.com/blog/2016/11/13/coreboot-on-the-thinkpad-x220-with-a-raspberry-pi/">Tyler Cipriani's Guide</a>
  * <a href="https://blogs.coreboot.org/blog/category/grub2/">Coreboot wiki GRUB2</a>
  * <a href="https://web.archive.org/web/20210420062101/https://notabug.org/libreboot/libreboot/src/master/resources/grub/config/menuentries/common.cfg">Libreboot's GRUB2 config</a>


## <b>Requirements</b>
  * Raspberry Pi
  * Void Linux PC


## <b>Preparation</b>

### Raspberry Pi
  * Enable SPI and install flashrom

### Void Linux
```
xbps-install -S ncurses-devel autoconf automake m4 bison flex xz curl openssl-devel zlib-devel gcc gcc-ada pciutils usbutils pciutils-devel gettext gettext-devel-tools pkg-config freetype-devel font-unifont-bdf
```


## <b>Reading</b>

### Raspberry Pi
```
flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=1024
flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=1024 -r flash1.bin
flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=1024 -r flash2.bin
md5sum flash1.bin flash2.bin
```

### Raspberry Pi
```
flashrom -p internal -r flash1.bin
flashrom -p internal -r flash1.bin
md5sum flash1.bin flash2.bin
```


## <b>Building</b>

### Void Linux
```
git clone --recurse-submodules https://review.coreboot.org/coreboot.git
cd coreboot/3rdparty
git clone https://review.coreboot.org/p/blobs.git
mkdir -p blobs/mainboard/lenovo/x220/
git clone https://github.com/corna/me_cleaner
cd ../../
```
```
cd coreboot/3rdparty/me_cleaner
python me_cleaner.py -c flash1.bin
python me_cleaner.py -S -r -t -d -O cleaned_flash.bin -D descriptor.bin -M me.bin flash1.bin
cp me.bin descriptor.bin ../blobs/mainboard/lenovo/x220/
cp cleaned_flash.bin ../../util/ifdtool/
cd ../../../
```
```
cd coreboot/util/ifdtool
make
./ifdtool -x cleaned_flash.bin
cp flashregion_3_gbe.bin ../../3rdparty/blobs/mainboard/lenovo/x220/gbe.bin
cd ../../../
```
```
cp config coreboot/.config
cp cmos.default coreboot/src/mainboard/lenovo/x220/
cp grub.cfg coreboot/grub.cfg
cp iosevka.pf2 coreboot/iosevka.pf2
cp iosevka_minimal.pf2 coreboot/iosevka_minimal.pf2
```
```
cd coreboot
make nconfig
For musl versions, add -static flag to TOOLLDFLAGS in cbfstool
make crossgcc-i386
make
cd ../
```
```
cd coreboot/util/cbfstool
make
cd ../../../
```
```
cd coreboot
util/cbfstool/cbfstool build/coreboot.rom add -n etc/iosevka.pf2 -t raw -f iosevka.pf2
util/cbfstool/cbfstool build/coreboot.rom add -n etc/iosevka_minimal.pf2 -t raw -f iosevka_minimal.pf2
cp build/coreboot.rom ../
cd ../
```


## <b>Flashing</b>

### Raspberry Pi
```
flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=1024 -w coreboot.rom
```

### Void Linux
```
flashrom -p internal -w coreboot.rom --ifd --image bios
```
