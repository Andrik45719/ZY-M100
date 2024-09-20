# TuyaMCU OTA research

- [Zigbee Generic Interfaces](https://developer.tuya.com/en/docs/iot/tuya-zigbee-universal-docking-access-standard?id=K9ik6zvofpzql)
- [Serial Communication Protocol](https://developer.tuya.com/en/docs/iot/tuya-zigbee-module-uart-communication-protocol?id=K9ear5khsqoty)

## TUYA_MCU_VERSION_REQ
## TUYA_MCU_VERSION_RSP
```c
int tuya_rep_fw_ver()
{
  char ver; // r0
  int l; // r0

  ver = get_current_mcu_fw_ver();  //66
  l = set_zigbee_uart_byte(0, ver);
  return zigbee_uart_write_frame(11, l);
}
```
## TUYA_MCU_OTA_NOTIFY
```c
int __fastcall response_mcu_ota_notify_event(int a1)
{
  char *zb_rx_hdr; // r5
  char *zb_rx_data; // r6
  int i; // r0
  int v4; // r0
  unsigned int v6; // r5
  int v7; // r6

  zb_rx_hdr = &zigbee_uart_rx_buf[a1];
  zb_rx_data = &zigbee_uart_rx_buf[a1 + 8];
  init_mcu_fw_pid();
  for ( i = 0; i != 8; ++i )
    mcu_ota_pid[i] = zb_rx_data[i];             // PID  'ztc6ggyl'
  upd_fw_ver = zb_rx_hdr[16];
  mcu_ota_fw_size = (zb_rx_hdr[18] << 16) + (zb_rx_hdr[17] << 24) + (zb_rx_hdr[19] << 8) + zb_rx_hdr[20];
  upd_fw_chksum32 = (zb_rx_hdr[22] << 16) + (zb_rx_hdr[21] << 24) + (zb_rx_hdr[23] << 8) + zb_rx_hdr[24];
  if ( !memcmp(mcu_ota_pid, prod_model_sign, 8)  //'ztc6ggyl'
    && (v6 = upd_fw_ver, v6 > get_current_mcu_fw_ver())
    && (mcu_ota_fw_size - 1) >> 0xD <= 2 )
  {
    v7 = set_zigbee_uart_byte(0, 0);            // OK
    init_RCU(0x71Cu);
    set_BKPWEN();
    set_FMC_KEY();
    RTC_BKP1 = 0x45670123;
    RTC_BKP2 = 0xCDEF89AB;
    mcu_current_offset = 0;
    zigbee_uart_write_frame(0xC, v7);           // MCU_OTA_NOTIFY_CMD
    return mcu_ota_fw_request();
  }
  else
  {
    v4 = set_zigbee_uart_byte(0, 1);            // error
    mcu_current_offset = 0;
    return zigbee_uart_write_frame(0xC, v4);    // MCU_OTA_NOTIFY_CMD
  }
}
```
## TUYA_OTA_BLOCK_DATA_REQ
```c
void __fastcall mcu_ota_fw_request()
{
  char len; // r1

  if ( mcu_ota_fw_size > mcu_current_offset )
  {
    byte_200006D6 = mcu_current_offset;
    word_200006CE = *&mcu_ota_pid[4];
    tuya_pkt_data = mcu_ota_pid[0];
    byte_200006D4 = BYTE2(mcu_current_offset);
    byte_200006D5 = BYTE1(mcu_current_offset);
    byte_200006D3 = HIBYTE(mcu_current_offset);
    unk_200006D2 = upd_fw_ver;
    unk_200006D1 = mcu_ota_pid[7];
    unk_200006D0 = mcu_ota_pid[6];
    unk_200006CD = mcu_ota_pid[3];
    unk_200006CC = mcu_ota_pid[2];
    unk_200006CB = mcu_ota_pid[1];
    len = mcu_ota_fw_size - mcu_current_offset;
    if ( (mcu_ota_fw_size - mcu_current_offset) >= 48 )
      len = 48;
    byte_200006D7 = len;
    zigbee_uart_write_frame(0xD, 14);
  }
}
```
## TUYA_OTA_BLOCK_DATA_RSP
```c
void __fastcall mcu_ota_fw_request_event(int a1)
{
  char *zb_rx_hdr; // r1
  int idx; // r2
  unsigned __int8 *offs_ptr; // r2
  int offs; // r0
  int v6; // r3
  int len; // r2
  char *data_ptr; // r1
  char *v9; // r3
  int v10; // r5
  int fw_size; // r0
  int curr_offs; // r1
  int chksum32; // r1
  char *ptr; // r2
  char buf[48]; // [sp+0h] [bp-40h] BYREF

  memzero(&buf[1], 47);
  buf[0] = -1;
  zb_rx_hdr = &zigbee_uart_rx_buf[a1];
  if ( zigbee_uart_rx_buf[a1 + 8] != 1 )
  {
    idx = 0;
    while ( prod_model_sign[idx] == zb_rx_hdr[idx + 9] )
    {
      if ( ++idx == 8 )
      {
        if ( upd_fw_ver == zb_rx_hdr[17] )
        {
          offs_ptr = (zb_rx_hdr + 18);
          offs = 0;
          v6 = 32;
          do
          {
            v6 -= 8;
            offs |= *offs_ptr++ << v6;
          }
          while ( v6 );
          if ( mcu_current_offset == offs )
          {
            len = mcu_ota_fw_size - offs;
            if ( (mcu_ota_fw_size - offs) >= 48 )
              len = 48;
            if ( len )
            {
              data_ptr = zb_rx_hdr + 22;
              v9 = buf;
              v10 = len;
              do
              {
                *v9++ = *data_ptr++;
                --v10;
              }
              while ( v10 );
            }
            mcu_current_offset = len + offs;
            flash_write_48(offs, buf, len);
            fw_size = mcu_ota_fw_size;
            curr_offs = mcu_current_offset;
            if ( mcu_current_offset < mcu_ota_fw_size )
            {
              mcu_ota_fw_request();
              fw_size = mcu_ota_fw_size;
              curr_offs = mcu_current_offset;
            }
            if ( curr_offs == fw_size )
            {
              chksum32 = 0;
              if ( fw_size )
              {
                ptr = flash_pgm_addr;
                do
                {
                  chksum32 += *ptr++;
                  --fw_size;
                }
                while ( fw_size );
              }
              if ( upd_fw_chksum32 == chksum32 )
                mcu_ota_result_report(0);
              else
                clear_RTC_BKP1();
            }
          }
        }
        return;
      }
    }
  }
}
```
## TUYA_MCU_OTA_RESULT
```c
void __fastcall mcu_ota_result_report(char status)
{
  tuya_pkt_data = status;
  unk_200006CB = mcu_ota_pid[0];
  byte_200006D3 = upd_fw_ver;
  unk_200006D2 = mcu_ota_pid[7];
  unk_200006D1 = mcu_ota_pid[6];
  unk_200006D0 = mcu_ota_pid[5];
  word_200006CE = *&mcu_ota_pid[3];
  unk_200006CD = mcu_ota_pid[2];
  unk_200006CC = mcu_ota_pid[1];
  zigbee_uart_write_frame(0xE, 10);
}
```
