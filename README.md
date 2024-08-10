# ZY-M100-L
## Backup
```batchfile
JLink.exe -device GD32E230F8 -if SWD -speed 4000 -autoconnect 1 
```
```
savebin ZY-M100_.bin 0x08000000 0x10000
exit
```

## Flash

```batchfile
JLink.exe -device GD32E230F8 -if SWD -speed 4000 -autoconnect 1 
```

```
loadfile ZY-M100_TargetDistance_disable.bin 0x08000000 reset
VerifyBin ZY-M100_TargetDistance_disable.bin  0x08000000
exit
```
