# ZY-M100-L/S_1/S_2/24G/24GV2 solving network spamming problem by disabling Target Distance reporting
Mods:
- Target distance (DP: 9) reporting disabled.
- Settings report interval (DPs: 1, 4, 2, 3, 6, 101, 102, 103, 104) increased to 60s, stock FW 10s.
- Illuminance Lux report interval (DP 104) increased to 10s, stock FW 500ms

Suppoted models:
- ZY-M100_L [(_TZE204_ztc6ggyl)](./ZY-M100_L_TZE204_ztc6ggyl-TargetDistance_disable.bin)
- ZY-M100-S_1 [(_TZE204_sxm7l9xa, _TZE204_e5m9c5hl)](./ZY-M100-S_1_TZE204_sxm7l9xa-TargetDistance_disable.bin)
- ZY-M100-S_2 [(_TZE204_qasjif9e)](./ZY-M100-S_2_TZE204_qasjif9e-TargetDistance_disable.bin)
- ZY-M100-24G [(_TZE204_ijxvkhd0)](./ZY-M100-24G_TZE204_ijxvkhd0-TargetDistance_disable.bin)
- ZY-M100-24GV2 [(_TZE204_7gclukjs)](./ZY-M100-24GV2_TZE204_7gclukjs-TargetDistance_disable.bin)
##
![JLink ZY-M100-L wall mount](./pix/wall_1.jpg)
![JLink ZY-M100-L ceiling mount](./pix/ceiling_1.jpg)
![JLink ZY-M100-L ceiling mount](./pix/ceiling_2.jpg)
![JLink ZY-M100-L ceiling mount](./pix/ceiling_3.jpg)

![ZY-M100-S_2 ceiling mount](./pix/ZY-M100-S_2.jpg)

# Connect to [JLink](http://www.segger.com)

|Signal|ZY-M100-L     |J-Link  |GD32|
|:-----|-------------:|-------:|---:|
|VCC   |(square pad) 1|(Vref) 1|    |
|GND   |2             |       4|    |
|SWDIO |3             |       7|19  |
|SWCLK |4             |       9|20  |

![JLink ZY-M100-L wall mount](./pix/wall_jlink.jpg)
![JLink ZY-M100-L ceiling mount](./pix/ceiling_jlink.jpg)

## Backup original firmware
```batchfile
JLink.exe -device GD32E230F8 -if SWD -speed 4000 -autoconnect 1 
```
```
savebin ZY-M100_L.bin 0x08000000 0x10000
exit
```

## Flash modified firmware

```batchfile
JLink.exe -device GD32E230F8 -if SWD -speed 4000 -autoconnect 1 
```
```
loadfile ZY-M100_L-TargetDistance_disable.bin 0x08000000 reset
VerifyBin ZY-M100_L-TargetDistance_disable.bin  0x08000000
exit
```
## Restore backuped firmware

```batchfile
JLink.exe -device GD32E230F8 -if SWD -speed 4000 -autoconnect 1 
```
```
loadfile ZY-M100_L.bin 0x08000000 reset
VerifyBin ZY-M100_L.bin  0x08000000
exit
```
# Connect to [ST-Link V2 clone](http://www.aliexpress.com)

|Signal|ZY-M100-L     |ST-Link V2|GD32|
|:-----|-------------:|---------:|---:|
|VCC   |(square pad) 1|         2|    |
|GND   |2             |         7|    |
|SWDIO |3             |         6|19  |
|SWCLK |4             |         4|20  |

**Be careful check adapter pinout! Some clones have different one.**


![JLink ZY-M100-L ceiling mount](./pix/ST-Link_pinout.png)
![JLink ZY-M100-L wall mount](./pix/wall_st-link.jpg)


## Backup original firmware using [OpenOCD](https://github.com/openocd-org/openocd/releases/tag/latest)
```batchfile
openocd -f interface/stlink-v2.cfg -f target/gd32e23x.cfg -c init -c "reset halt" -c "flash read_bank 0 ZY-M100_L.bin" -c "reset" -c shutdown
```

## Flash modified firmware
```batchfile
openocd -f interface/stlink-v2.cfg -f target/gd32e23x.cfg -c init -c "reset halt" -c "flash erase_sector 0 0 last" -c "flash write_bank 0 ZY-M100_L-TargetDistance_disable.bin" -c "flash verify_bank 0 ZY-M100_L-TargetDistance_disable.bin" -c "reset" -c shutdown
```
## Restore backuped firmware
```batchfile
openocd -f interface/stlink-v2.cfg -f target/gd32e23x.cfg -c init -c "reset halt" -c "flash erase_sector 0 0 last" -c "flash write_bank 0 ZY-M100_L.bin" -c "flash verify_bank 0 ZY-M100_L.bin" -c "reset" -c shutdown
```

# tech info
- Microwave motion sensor module [JYSJ_5807_A01](./5807_A01.pdf)
- TuyaMCU [GD32E230F8P6TR](./GD32E230F8P6.pdf)
- Zigbee module [ZS3L](https://developer.tuya.com/en/docs/iot/zs3l?id=K97r37j19f496)
- [Адаптеры JTAG с поддержкой SWD](https://microsin.net/programming/arm/swd-jtag-adapters.html)
- [SWJ adapters](https://wiki.cuvoodoo.info/doku.php?id=jtag)
