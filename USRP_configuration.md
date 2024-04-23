**USRP Configuration Tips**

_Official Guide:_
```
https://kb.ettus.com/USRP_X_Series_Quick_Start_(Daughterboard_Installation)
```

_Useful commands to check USRP status:_
```
uhd_find_devices
uhd_usrp_probe
```

_How to flash new images on USRP:_
```
https://files.ettus.com/manual/page_usrp_x3x0.html#x3x0_flash
https://files.ettus.com/manual/page_images.html
```

_Useful commands to flash new USRP imges:_
```
uhd_images_downloader
uhd_image_loader --args="type=x300,addr=192.168.10.2"
```

_Run UHD example application to collect RX data:_
```
/usr/lib/uhd/examples/rx_samples_to_file --rate 100000000 --freq 1575420000 --file /tmp/bla.iq --duration 10
```


_**BUG FIXING:** Failed to find a valid XML path for RFNoC blocks:_
```
https://bugs.launchpad.net/ubuntu/+source/uhd/+bug/1780805
```
 - Get UHD 3.13.1 source files:
```
https://github.com/EttusResearch/uhd/releases/tag/v3.13.1.0
```
 - Extract downloaded sources and recreate the rfnoc folder:
```
sudo cp -Rv ~/Downloads/rfnoc/ /usr/lib/uhd/
export UHD_RFNOC_DIR=/usr/lib/uhd/rfnoc
```
- Add the following in your `${HOME}/.profile` file:
```
export UHD_RFNOC_DIR=/usr/lib/uhd/rfnoc
```

_Change USRP IPs:_
- 1Gbps IP:
  ```
  ./usrp_burn_mb_eeprom --values="ip-addr0=192.168.10.2"
  ```
- 10Gbps IP:
  ```
  ./usrp_burn_mb_eeprom --values="ip-addr3=192.168.50.2"
  ```
