# Connecting to External Signal Processing Equipment

## VISA via USB connection
This is the simplest way to communicate with devices like oscilloscopes, Signal Generators etc
Even older devices have a USB connection that uses VISA.

To connect to a device via VISA you need to install the VISA package for the programming language you prefer.

Despite VISA being used over the raw USB connection, you might have trouble with your operating system.
For Ubuntu Linux, you need to specifically allow the use of these kind of connection by the users.
Otherwise, you might not be able to list your device as VISA resource despite you being able to see it listed in `lsusb` output.

### Ubuntu:
- Modify the following file:
```shell
sudo vim /etc/udev/rules.d/99-com.rules
```

- Add the following lines, save and exit:
```shell
SUBSYSTEM=="usb", MODE="0666", GROUP="usbusers"
```

- reboot your Linux

(Adapted from https://stackoverflow.com/questions/66480203/pyvisa-not-listing-usb-instrument-on-linux/66480539#66480539)

### Using Python:
- Dependencies Installation:
  ```shell
  pip install pyvisa-py zeroconf
  ```
- Minimal Example:
  ```python
  import pyvisa
  
  def main():
   _rm = pyvisa.ResourceManager()
   devices = _rm.list_resources()
   print(devices)
  
  if __name__=='__main__':
  main()
  
  ```

  
