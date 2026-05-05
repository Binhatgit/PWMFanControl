# PWMFanControl
Simple script to control a PWM fan on a Raspberry Pi based on CPU temperature.
https://the-diy-life.com/connecting-a-pwm-fan-to-a-raspberry-pi/

Debian GNU/Linux 11 (Linux 6.1.21) (bullseye) on aarch64 
- Machine: Raspberry Pi 4 Model B Rev 1.2

#Bullseye version. Fix unstable by rolling back the firmware update [rpi-update] to the stable version.
sudo apt-get install --reinstall raspberrypi-bootloader raspberrypi-kernel
#Check temperature
cat /sys/class/thermal/thermal_zone0/temp 
FanProportional is less noise. The FanStepped script ramps up the fan speed in steps. 

#INSTALL

#Install FanStepped #########################################
sudo cp FanStepped.py /usr/local/sbin
sudo cp FanStepped.service /etc/systemd/system/

systemctl daemon-reload
systemctl enable --no-pager FanStepped.service
systemctl restart --no-pager FanStepped.service
systemctl status --no-pager FanStepped.service

#Edit and check
sudo nano /usr/local/sbin/FanStepped.py
sudo systemctl stop --no-pager FanStepped.service
sudo systemctl restart --no-pager FanStepped.service && sudo systemctl status --no-pager FanStepped.service

sudo systemctl restart --no-pager FanStepped.service
sudo systemctl status --no-pager FanStepped.service

#Remove
systemctl stop --no-pager FanStepped.service
systemctl disable --no-pager FanStepped.service


#Install FanProportional #########################################
sudo cp FanProportional.py /usr/local/sbin
sudo cp FanProportional.service /etc/systemd/system/

systemctl start --no-pager FanProportional.service
systemctl enable --no-pager FanProportional.service
systemctl restart --no-pager FanProportional.service
systemctl start --no-pager FanProportional.service
systemctl status --no-pager FanProportional.service


sudo systemctl start FanProportional.service
sudo systemctl status FanProportional.service

#Edit and check
nano /etc/systemd/system/FanProportional.service
nano /usr/local/sbin/FanProportional.py

#Board layout 
```
Fan pin
|.........................|                        
| Raspi chip    1 | 2 +5v |
| |....|          | 4 +5v |-----> +5V
| |....|          | 6 GND |-----> GND
|                 |       |
|                 |       |
|                 |       |
|                 | 33    |-----> Tachometer - fan speed (BCM 13)
|                 | 35    |<----- PWM for Fan drive speed (BCM 19)
|                 | 37    |
|                 | 39    |
|.........................|

```
#gpio readall
```
 +-----+-----+---------+------+---+---Pi 4B--+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | PHYSICAL | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 |     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
 |   2 |   8 |   SDA.1 |   IN | 1 |  3 || 4  |   |      | 5v      |     |     |
 |   3 |   9 |   SCL.1 |   IN | 1 |  5 || 6  |   |      | 0v      |     |     |
 |   4 |   7 | GPIO. 7 |   IN | 1 |  7 || 8  | 1 | IN   | TxD     | 15  | 14  |
 |     |     |      0v |      |   |  9 || 10 | 1 | IN   | RxD     | 16  | 15  |
 |  17 |   0 | GPIO. 0 |   IN | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
 |  27 |   2 | GPIO. 2 |   IN | 0 | 13 || 14 |   |      | 0v      |     |     |
 |  22 |   3 | GPIO. 3 |   IN | 0 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
 |     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
 |  10 |  12 |    MOSI |   IN | 0 | 19 || 20 |   |      | 0v      |     |     |
 |   9 |  13 |    MISO |   IN | 0 | 21 || 22 | 0 | IN   | GPIO. 6 | 6   | 25  |
 |  11 |  14 |    SCLK |   IN | 0 | 23 || 24 | 1 | IN   | CE0     | 10  | 8   |
 |     |     |      0v |      |   | 25 || 26 | 1 | IN   | CE1     | 11  | 7   |
 |   0 |  30 |   SDA.0 |   IN | 1 | 27 || 28 | 1 | IN   | SCL.0   | 31  | 1   |
 |   5 |  21 | GPIO.21 |   IN | 1 | 29 || 30 |   |      | 0v      |     |     |
 |   6 |  22 | GPIO.22 |   IN | 1 | 31 || 32 | 0 | IN   | GPIO.26 | 26  | 12  |
 |  13 |  23 | GPIO.23 |   IN | 0 | 33 || 34 |   |      | 0v      |     |     |
 |  19 |  24 | GPIO.24 |  OUT | 0 | 35 || 36 | 0 | IN   | GPIO.27 | 27  | 16  |
 |  26 |  25 | GPIO.25 |   IN | 0 | 37 || 38 | 0 | IN   | GPIO.28 | 28  | 20  |
 |     |     |      0v |      |   | 39 || 40 | 0 | IN   | GPIO.29 | 29  | 21  |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+---Pi 4B--+---+------+---------+-----+-----+
```
<img width="941" height="540" alt="image" src="https://github.com/user-attachments/assets/b19dcc25-534f-440b-80e0-67409003603c" />
