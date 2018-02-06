# Cloudshell2 setup for Odroid

# Setup Cloudshell
Follow [instructions](https://wiki.odroid.com/accessory/add-on_boards/xu4_cloudshell2/xu4_cloudshell2) to control LCD, fan, remote (steps 3, 4 ,6).

## Test lirc
* `irw`
* Push buttons on remote

## Configure lircrc
[Example config file](http://marklodato.github.io/2013/10/24/how-to-use-lirc.html)

### Contents of `/etc/lirc/lircrc`
```
begin
    button = KEY_POWER
    prog = irexec
    config = /root/lcd_toggle
    repeat = 1
end

begin
    button = KEY_ENTER
    prog = irexec
    config = /root/fan_toggle
    repeat = 1
end
```
