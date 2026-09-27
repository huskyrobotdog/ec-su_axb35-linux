# ec-su_axb35-linux
Linux driver for the embedded controller on the Sixunited AXB35-02 board.

Vendors using that board:
  - GMKtec EVO-X2
  - Bosgame M5
  - FEVM FA-EX9
  - Peladn YO1
  - NIMO AI MiniPC
  - Corsair AI Workstation 300

An update to date list can be found [here](https://strixhalo-homelab.d7.wtf/Hardware/Boards/Sixunited-AXB35)

For more details, please have a look at the desevens wiki: https://strixhalo-homelab.d7.wtf/Guides/Power-Mode-and-Fan-Control

# Build instructions
```
$ make
$ sudo make install
$ sudo insmod ec_su_axb35
```

# Devices
```
# Fan devices
/sys/class/ec_su_axb35/fan1/                    - CPU fan 1
/sys/class/ec_su_axb35/fan2/                    - CPU fan 2
/sys/class/ec_su_axb35/fan3/                    - System fan
/sys/class/ec_su_axb35/fanX/rpm            (RO) - current speed in rpm
/sys/class/ec_su_axb35/fanX/mode           (RW) - [auto, fixed, curve]
/sys/class/ec_su_axb35/fanX/level          (RW) - [0-5] (0=0%, 1=20%, ..., 5=100%)
/sys/class/ec_su_axb35/fanX/rampup_curve   (RW) - 5 values (°C thresholds for level 1-5)
/sys/class/ec_su_axb35/fanX/rampdown_curve (RW) - 5 values (°C thresholds for level 1-5)

# Temperature device
/sys/class/ec_su_axb35/temp1/                   - CPU temperature in °C
/sys/class/ec_su_axb35/temp1/temp          (RO) - current
/sys/class/ec_su_axb35/temp1/min           (RO) - min temp measured since dirver load
/sys/class/ec_su_axb35/temp1/max           (RO) - amx temp measured since driver load

# APU device
/sys/class/ec_su_axb35/apu/power_mode      (RW) - [quiet, balanced, performance]
```

# Python GUI app (needs root to write to /sys/class/ec_su_axb35/*)
to test:
python ./ec-su_axb35-linux-gui.py

to install:
sudo install -m 755 ec-su_axb35-linux-gui.py /usr/local/bin/ec-su_axb35-linux-gui
cp ec-fan-control.desktop ~/.local/share/applications/


# 自定义开发补充说明

## 风扇曲线
```
# Fan 1
echo "45,53,60,66,70" | sudo tee /sys/class/ec_su_axb35/fan1/rampup_curve
echo "40,48,55,61,65" | sudo tee /sys/class/ec_su_axb35/fan1/rampdown_curve
echo "curve" | sudo tee /sys/class/ec_su_axb35/fan1/mode

# Fan 2
echo "45,53,60,66,70" | sudo tee /sys/class/ec_su_axb35/fan2/rampup_curve
echo "40,48,55,61,65" | sudo tee /sys/class/ec_su_axb35/fan2/rampdown_curve
echo "curve" | sudo tee /sys/class/ec_su_axb35/fan2/mode
```

## 开机服务
```
cd ec-su_axb35-linux
sudo make install
sudo modprobe ec_su_axb35
dmesg | grep 'Sixunited AXB35-02 EC driver loaded'

echo ec_su_axb35 | sudo tee /etc/modules-load.d/ec_su_axb35.conf
cat /etc/modules-load.d/ec_su_axb35.conf

---

sudo vim /etc/systemd/system/fancontrol.service

[Unit]
Description=Set Strix Halo fan curves
After=multi-user.target

[Service]
Type=oneshot
RemainAfterExit=yes

ExecStart=/bin/sh -c 'echo "45,53,60,66,70" > /sys/class/ec_su_axb35/fan1/rampup_curve'
ExecStart=/bin/sh -c 'echo "40,48,55,61,65" > /sys/class/ec_su_axb35/fan1/rampdown_curve'
ExecStart=/bin/sh -c 'echo "curve" > /sys/class/ec_su_axb35/fan1/mode'

ExecStart=/bin/sh -c 'echo "45,53,60,66,70" > /sys/class/ec_su_axb35/fan2/rampup_curve'
ExecStart=/bin/sh -c 'echo "40,48,55,61,65" > /sys/class/ec_su_axb35/fan2/rampdown_curve'
ExecStart=/bin/sh -c 'echo "curve" > /sys/class/ec_su_axb35/fan2/mode'

[Install]
WantedBy=multi-user.target

---

sudo systemctl daemon-reload
sudo systemctl enable --now fancontrol.service
```

## 内核设置
```
sudo vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="ipv6.disable=1 amdgpu.dcdebugmask=0x1000 amdgpu.cwsr_enable=0 amd_iommu=off"
GRUB_CMDLINE_LINUX_DEFAULT="ipv6.disable=1 amdgpu.dc=0 amdgpu.cwsr_enable=0 amd_iommu=off"
```
