# ZY-M100-L (_TZE204_ztc6ggyl) solving network spamming problem by disabling Target Distance reporting
This solution for **5.8G sensors only** with [JYSJ_5807_A01](./5807_A01.pdf) module.

![JLink ZY-M100-L wall mount](./pix/wall_1.jpg)
![JLink ZY-M100-L ceiling mount](./pix/ceiling_1.jpg)
![JLink ZY-M100-L ceiling mount](./pix/ceiling_2.jpg)

## Connect to [JLink](http://www.segger.com)

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

## tech info
- Microwave motion sensor module [JYSJ_5807_A01](./5807_A01.pdf)
- TuyaMCU [GD32E230F8P6TR](./GD32E230F8P6.pdf)
- Zigbee module [ZS3L](https://developer.tuya.com/en/docs/iot/zs3l?id=K97r37j19f496)


