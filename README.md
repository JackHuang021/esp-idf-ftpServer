# esp-idf-ftpServer
FTP Server for esp-idf using FAT file system.   
I found [this](https://www.esp32.com/viewtopic.php?f=13&t=5013#p21738) information.   
So, I ported from [here](https://github.com/loboris/MicroPython_ESP32_psRAM_LoBo/blob/master/MicroPython_BUILD/components/micropython/esp32/libs/ftp.c).   
Since it uses the FAT file system instead of SPIFFS, directory operations are possible.   

# Software requirements
esp-idf v4.4 or later.   
This is because this version supports ESP32-C3.   

# Installation
```
git clone https://github.com/nopnop2002/esp-idf-ftpServer
cd esp-idf-ftpServer/
idf.py set-target {esp32/esp32s2/esp32s3/esp32c3}
idf.py menuconfig
idf.py flash monitor
```

__If you need more storage space on FLASH, you need to modify partitions_example.csv.__   

# Partition table
```
# Name,   Type, SubType, Offset,  Size, Flags
# Note: if you have increased the bootloader size, make sure to update the offsets to avoid overlap
nvs,      data, nvs,     0x9000,  0x6000,
phy_init, data, phy,     0xf000,  0x1000,
factory,  app,  factory, 0x10000, 1M,
storage,  data, spiffs,         , 0xF0000,  ---> This is for FAT file system
```

# Configuration

![config-main](https://user-images.githubusercontent.com/6020549/107847756-a4aa2900-6e31-11eb-9525-6fd82bead5a3.jpg)
![config-app](https://user-images.githubusercontent.com/6020549/127939325-1b565ef7-9045-4800-95ad-0153342b5fc1.jpg)

## File System Selection
ESP32 supports the following file systems.   
You can select any one using menuconfig.   
- FAT file system on FLASH   
- FAT file system on SPI peripheral SDCARD   
- FAT file system on SDMMC peripheral SDCARD(Valid only for ESP32/ESP32S3)   
- FAT file system on SPI Flash Memory like Winbond W25Q64(Not supported in this project)   
- FAT file system on USB Memory Stick(Not supported in this project)   

Besides this, the ESP32 supports the SPIFFS filesystem, but I don't use it because it can't handle directories.   

![config-filesystem-1](https://user-images.githubusercontent.com/6020549/165684095-d1fd8f77-afb7-466e-9eda-061776e8b9f9.jpg)
![config-filesystem-2](https://user-images.githubusercontent.com/6020549/165684099-6d1a9563-17a3-49ba-a995-2ce400633cde.jpg)

Note:   
The connection when using SDSPI, SDMMC will be described later.   

## WiFi Setting
![config-wifi-1](https://user-images.githubusercontent.com/6020549/127940375-f19519f0-88ce-407a-aa83-a00de776689e.jpg)

You can connect using mDNS.   
![config-wifi-2](https://user-images.githubusercontent.com/6020549/127940382-d431c962-746e-45d7-9693-3f844c0b01d3.jpg)

You can use static IP.   
![config-wifi-3](https://user-images.githubusercontent.com/6020549/127940390-3edfb3ea-6545-4709-9786-3e8a944e5ac7.jpg)


## FTP Server Setting
![config-server](https://user-images.githubusercontent.com/6020549/127940653-0d54f2ca-5dee-4c97-a7e7-276299237a41.jpg)



# Using FAT file system on SPI peripheral SDCARD

|ESP32|ESP32S2/S3|ESP32C3|SPI card pin|Notes|
|:-:|:-:|:-:|:-:|:--|
|GPIO23|GPIO35|GPIO04|MOSI|10k pullup if can't mount|
|GPIO19|GPIO37|GPIO06|MISO||
|GPIO18|GPIO36|GPIO05|SCK||
|GPIO14|GPIO34|GPIO01|CS|||
|3.3V|3.3V|3.3V|VCC|Don't use 5V supply|
|GND|GND|GND|GND||

![config-filesystem-SDSPI](https://user-images.githubusercontent.com/6020549/165686735-21461822-f19e-47d5-aedc-b91401670098.jpg)

__You can change it to any pin using menuconfig.__   

Note:   
This project doesn't utilize card detect (CD) and write protect (WP) signals from SD card slot.   


# Using FAT file system on SDMMC peripheral SDCARD

On ESP32, SDMMC peripheral is connected to specific GPIO pins using the IO MUX.   
__GPIO pins cannot be customized.__   
Please see the table below for the pin connections.

|ESP32 pin|SD card pin|Notes|
|:-:|:-:|:--|
|GPIO14|CLK|10k pullup|
|GPIO15|CMD|10k pullup|
|GPIO2|D0|10k pullup or connect to GPIO0|
|GPIO4|D1|not used in 1-line SD mode; 10k pullup in 4-line SD mode|
|GPIO12|D2|not used in 1-line SD mode; 10k pullup in 4-line SD mode|
|GPIO13|D3|not used in 1-line SD mode, but card's D3 pin must have a 10k pullup
|N/C|CD|not used in this project|
|N/C|WP|not used in this project|
|3.3V|VCC|Don't use 5V supply|
|GND|GND||

![config-filesystem-SDMMC-ESP32](https://user-images.githubusercontent.com/6020549/165684959-8370ae90-8f5c-40e6-961d-7a21b48a9ae5.jpg)


On ESP32-S3, SDMMC peripheral is connected to GPIO pins using GPIO matrix.   
__This allows arbitrary GPIOs to be used to connect an SD card.__   

|ESP32-S3 pin|SD card pin|Notes|
|:-:|:-:|:--|
|GPIO36|CLK|10k pullup|
|GPIO35|CMD|10k pullup|
|GPIO37|D0|10k pullup|
|GPIO38|D1|not used in 1-line SD mode; 10k pullup in 4-line SD mode|
|GPIO33|D2|not used in 1-line SD mode; 10k pullup in 4-line SD mode|
|GPIO34|D3|not used in 1-line SD mode, but card's D3 pin must have a 10k pullup
|N/C|CD|not used in this project|
|N/C|WP|not used in this project|
|3.3V|VCC|Don't use 5V supply|
|GND|GND||

![config-filesystem-SDMMC-ESP32S3](https://user-images.githubusercontent.com/6020549/165685227-6d78e26f-e7e3-4dc4-8aba-6d47d6c15b62.jpg)


## Note about GPIO2 (ESP32 only)   
GPIO2 pin is used as a bootstrapping pin, and should be low to enter UART download mode. One way to do this is to connect GPIO0 and GPIO2 using a jumper, and then the auto-reset circuit on most development boards will pull GPIO2 low along with GPIO0, when entering download mode.

- Some boards have pulldown and/or LED on GPIO2. LED is usually ok, but pulldown will interfere with D0 signals and must be removed. Check the schematic of your development board for anything connected to GPIO2.

## Note about GPIO12 (ESP32 only)   
GPIO12 is used as a bootstrapping pin to select output voltage of an internal regulator which powers the flash chip (VDD_SDIO). This pin has an internal pulldown so if left unconnected it will read low at reset (selecting default 3.3V operation). When adding a pullup to this pin for SD card operation, consider the following:

- For boards which don't use the internal regulator (VDD_SDIO) to power the flash, GPIO12 can be pulled high.
- For boards which use 1.8V flash chip, GPIO12 needs to be pulled high at reset. This is fully compatible with SD card operation.
- On boards which use the internal regulator and a 3.3V flash chip, GPIO12 must be low at reset. This is incompatible with SD card operation.
    * In most cases, external pullup can be omitted and an internal pullup can be enabled using a `gpio_pullup_en(GPIO_NUM_12);` call. Most SD cards work fine when an internal pullup on GPIO12 line is enabled. Note that if ESP32 experiences a power-on reset while the SD card is sending data, high level on GPIO12 can be latched into the bootstrapping register, and ESP32 will enter a boot loop until external reset with correct GPIO12 level is applied.
    * Another option is to burn the flash voltage selection efuses. This will permanently select 3.3V output voltage for the internal regulator, and GPIO12 will not be used as a bootstrapping pin. Then it is safe to connect a pullup resistor to GPIO12. This option is suggested for production use.

# Using long file name support   
By default, FATFS file names can be up to 8 characters long.   
If you use filenames longer than 8 characters, you need to change the values below.   
![config_long_file_name_support-1](https://user-images.githubusercontent.com/6020549/193048975-318a5a78-24de-4656-ab7c-161b55962ce6.jpg)
![config_long_file_name_support-2](https://user-images.githubusercontent.com/6020549/193048987-7ffe34f1-1c3a-46a2-a614-6382d2d4c2c0.jpg)
![config_long_file_name_support-3](https://user-images.githubusercontent.com/6020549/193059963-b883c577-f6ed-4450-b01b-fc00fa9c9afd.jpg)

# Limitations   
- The server does not support multiple connections.   
- The server does not support active connection.    
 __Only passive connections are supported.__   
 Unfortunately, Windows standard ftp.exe does not allow passive mode (PASV) connections.   
 If you have to make a passive mode connection On Windows, you need to use another software such as FFFTP / WinSCP / FileZilla to connect in passive mode.   
- The server can only process these commands.
   * SYST
   * CDUP
   * CWD
   * PWD
   * XPWD(Same as PWD)
   * SIZE
   * MDTM(Always GMT)
   * TYPE
   * USER
   * PASS
   * PASV
   * LIST
   * RETR
   * STOR
   * DELE
   * RMD
   * MKD
   * RNFR(Rename From)
   * RNTO(Rename To)
   * NOOP
   * QUIT
   * APPE
   * NLST

# Using LilyGo ESP32-S2
The LilyGo ESP32-S2 development board has a micro SD card slot on the board.   
It is connected to the ESP32 by SPI, and the peripheral power is supplied from GPIO14.   
With this, you can easily build an FTP server.   
__No equipment other than the development board is required.__   
It works very stably.   

![LilyGo-esp32-s2-1](https://user-images.githubusercontent.com/6020549/107864770-00f96100-6ea3-11eb-8549-6885ae398111.JPG)
![LilyGo-esp32-s2-2](https://user-images.githubusercontent.com/6020549/127941181-63e48570-88ab-4d5b-8be7-343168079c4a.jpg)

# Screen Shot   
![ftp-srver-1](https://user-images.githubusercontent.com/6020549/107865100-9d713280-6ea6-11eb-94d2-4a249bd36a7b.jpg)
![ftp-srver-2](https://user-images.githubusercontent.com/6020549/107848057-fce22a80-6e33-11eb-8c67-471deb16e190.jpg)

# Windows ftp client
I tested these client.   
__You need to set the connection type to Passive Mode.__   
- WinSCP   
![WinSCP](https://user-images.githubusercontent.com/6020549/110587152-9d6b0680-81b6-11eb-857d-31d87379a3b1.jpg)

- FileZilla   
You need to make this setting when using FileZilla.
![FileZilla-2](https://user-images.githubusercontent.com/6020549/131819541-0ad0aacb-ed79-45c3-a536-bd239463ef02.jpg)
![FileZilla](https://user-images.githubusercontent.com/6020549/110587823-795bf500-81b7-11eb-8dcb-eeb6e096e9f6.jpg)

- FFFTP   
![FFFTP](https://user-images.githubusercontent.com/6020549/110587181-ab208c00-81b6-11eb-9c41-95e3e1d67949.jpg)

# Troubleshooting   
I sometimes get this error when using external SPI SD card readers.   
Requires a PullUp resistor.   
![sd-card-1](https://user-images.githubusercontent.com/6020549/107848058-fe135780-6e33-11eb-9eac-7ce160571276.jpg)


You can see all the logging on the server side by commenting it out here.   
```
void ftp_task (void *pvParameters)
{
  ESP_LOGI(FTP_TAG, "ftp_task start");
  //esp_log_level_set(FTP_TAG, ESP_LOG_WARN); ------------> Comment out
  strcpy(ftp_user, CONFIG_FTP_USER);
  strcpy(ftp_pass, CONFIG_FTP_PASSWORD);
  ESP_LOGI(FTP_TAG, "ftp_user:[%s] ftp_pass:[%s]", ftp_user, ftp_pass);
```


# Reference
https://github.com/nopnop2002/esp-idf-ftpClient
