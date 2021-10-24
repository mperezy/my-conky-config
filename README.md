# My Conky Configuration
* This is my simple configuration for the Conky System Monitor

## Requirements:
* Conky ([Here intallation](https://github.com/brndnmtthws/conky/wiki/Installation#debian--ubuntu))
* NVIDIA Graphic Card - Only for Monitorize purposes, if you don't have one, just delete or comment the GPU lines on `.conkyrc`
* NVIDIA GPU Driver Installed ([Here installation](https://www.cyberciti.biz/faq/ubuntu-linux-install-nvidia-driver-latest-proprietary-driver/))
* Sensors ([Here installation](https://linoxide.com/monitoring-2/install-lm-sensors-linux/))

## Before Installation:
* Sensors (lm-sensors) is a program that provides a hardware health monitoring driver for Linux such as temps and voltages
  from the CPU, Motherboard or GPU as well, we can check that information running `sensors` in terminal. But I had a problem
  trying to use this in my rig.
  ```bash
  $ sensors
    asus-isa-0000
    Adapter: ISA adapter
    cpu_fan:        0 RPM

    coretemp-isa-0000
    Adapter: ISA adapter
    Package id 0:  +37.0°C  (high = +86.0°C, crit = +100.0°C)
    Core 0:        +34.0°C  (high = +86.0°C, crit = +100.0°C)
    Core 1:        +37.0°C  (high = +86.0°C, crit = +100.0°C)
    Core 2:        +35.0°C  (high = +86.0°C, crit = +100.0°C)
    Core 3:        +36.0°C  (high = +86.0°C, crit = +100.0°C)
    Core 4:        +35.0°C  (high = +86.0°C, crit = +100.0°C)
    Core 5:        +35.0°C  (high = +86.0°C, crit = +100.0°C)
    Core 6:        +35.0°C  (high = +86.0°C, crit = +100.0°C)
    Core 7:        +35.0°C  (high = +86.0°C, crit = +100.0°C)
  ```
* In my rig I have 4 fan configurations but seems that is printing the correct information, my motherboard is an ASUS ROG Maximus XI Hero (WI-FI) Z390. Searching about that issue, I found this [issue thread](https://github.com/lm-sensors/lm-sensors/issues/134#issuecomment-427486133). Following that thread it seems that my motherboard's IO chip it not matching to `sensors` program.

* Running `sudo sensors-detect` detected an unknown Super IO chip as follow:
  ```bash
  $ sudo sensors-detect 
  [sudo] password for manuelperez: 
  # sensors-detect revision 6284 (2015-05-31 14:00:33 +0200)
  # Board: ASUSTeK COMPUTER INC. ROG MAXIMUS XI HERO (WI-FI)
  # Kernel: 5.3.0-28-generic x86_64
  # Processor: Intel(R) Core(TM) i7-9700K CPU @ 3.60GHz (6/158/13)

  Some Super I/O chips contain embedded sensors. We have to write to
  standard I/O ports to probe them. This is usually safe.
  Do you want to scan for Super I/O sensors? (YES/no): 
  Probing for Super-I/O at 0x2e/0x2f
  Trying family `National Semiconductor/ITE'...               No
  Trying family `SMSC'...                                     No
  Trying family `VIA/Winbond/Nuvoton/Fintek'...               Yes
  Found unknown chip with ID 0xd42b
      (logical device B has address 0x290, could be sensors) <-----
  Probing for Super-I/O at 0x4e/0x4f
  Trying family `National Semiconductor/ITE'...               No
  Trying family `SMSC'...                                     No
  Trying family `VIA/Winbond/Nuvoton/Fintek'...               No
  Trying family `ITE'...                                      No
  ``` 

* Keep checking the github issue thread shared before, found [this comment](https://github.com/lm-sensors/lm-sensors/issues/134#issuecomment-442207103), following only the first step found that my Super IO chip is **`Nuvoton NCT6798D`**. So keep checking again this thread, found that I needed to add a boot parameter detailed [here](https://github.com/lm-sensors/lm-sensors/issues/134#issuecomment-518678263) that's why I added `acpi_enforce_resources=lax` in grub file:
  ```bash
  $ sudo nano /etc/default/grub 

  GRUB_DEFAULT=0
  GRUB_TIMEOUT_STYLE=hidden
  GRUB_TIMEOUT=10
  GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
  GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_enforce_resources=lax" <------
  GRUB_CMDLINE_LINUX=""
  ```

  Save changes and then:
  ```bash
  $ sudo update-grub
  $ reboot
  ```

  * Just to mention that this next command: `modprobe -v nct6775` needs to be executed at boot in order to check the fans speed and it needs to be executed as sudo, so we need to add that command into a **rc.local** file in order to not be prompted to enter the sudo password. So, to solve this we must do the next:
  
    ```bash
    $ sudo touch /etc/rc.local # This if the file not exists.
    $ sudo chmod 755 /etc/rc.local
    ```

    Then we need to add this content:
    ```bash
    #!/bin/sh -e

    modprobe -v nct6775

    exit 0
    ```

  Finally it works for me:
  ```bash
  $ sensors
  nct6798-isa-0290
  Adapter: ISA adapter
  in0:                     1.21 V  (min =  +0.00 V, max =  +1.74 V)
  in1:                     1.01 V  (min =  +0.00 V, max =  +0.00 V)  ALARM
  in2:                     3.39 V  (min =  +0.00 V, max =  +0.00 V)  ALARM
  in3:                     3.33 V  (min =  +0.00 V, max =  +0.00 V)  ALARM
  in4:                     1.72 V  (min =  +0.00 V, max =  +0.00 V)  ALARM
  in5:                   216.00 mV (min =  +0.00 V, max =  +0.00 V)  ALARM
  in6:                     0.00 V  (min =  +0.00 V, max =  +0.00 V)
  in7:                     3.39 V  (min =  +0.00 V, max =  +0.00 V)  ALARM
  in8:                     3.15 V  (min =  +0.00 V, max =  +0.00 V)  ALARM
  in9:                   536.00 mV (min =  +0.00 V, max =  +0.00 V)  ALARM
  in10:                  608.00 mV (min =  +0.00 V, max =  +0.00 V)  ALARM
  in11:                  664.00 mV (min =  +0.00 V, max =  +0.00 V)  ALARM
  in12:                    1.06 V  (min =  +0.00 V, max =  +0.00 V)  ALARM
  in13:                  608.00 mV (min =  +0.00 V, max =  +0.00 V)  ALARM
  in14:                  632.00 mV (min =  +0.00 V, max =  +0.00 V)  ALARM
  fan1:                  1013 RPM  (min =    0 RPM)
  fan2:                  1433 RPM  (min =    0 RPM)
  fan3:                   801 RPM  (min =    0 RPM)
  fan4:                   887 RPM  (min =    0 RPM)
  fan5:                     0 RPM  (min =    0 RPM)
  fan6:                     0 RPM  (min =    0 RPM)
  fan7:                  2486 RPM  (min =    0 RPM)
  SYSTIN:                 +25.0°C  (high = +80.0°C, hyst = +75.0°C)  sensor = thermistor
  CPUTIN:                 +33.0°C  (high = +80.0°C, hyst = +75.0°C)  sensor = thermistor
  AUXTIN0:               -128.0°C    sensor = thermistor
  Package id 0:  +36.0°C  (high = +86.0°C, crit = +100.0°C)
  Core 0:        +33.0°C  (high = +86.0°C, crit = +100.0°C)
  ...
  Core 7:        +34.0°C  (high = +86.0°C, crit = +100.0°C)
  ```
* If we didn't execute `modprobe -v nct6775` at every system start-up, we won't be able to see the fans information, that's why I added that command in **`conky-startup.sh`** to be executed at system start-up.

* If those steps shared above didn't work for you, please go to the github issue thread shared before to try to find a solution for your motherboard.

## Installation
* Just copy the `.conkyrc` in `~/.conkyrc` with **`644`** permissions:
  ```bash
  $ cp .conkyrc ~/.conkyrc
  $ chmod 644 ~/.conkyrc
  $ chown $USER:$USER ~/.conkyrc
  ```

## Run your Conky at startup
* You need to create a background process where Conky will be running, in the distro that is use which is **Ubuntu**, I create it in *Startup Applications Preferences*, first we need to create a bash file for example `conky-startup.sh` that is placed in this repo.

* In my case, I prefered to executed it from *$HOME* folder, you can execute wherever place you want. Remember that you need to execute that bash file as an application, so:


  ```bash
   $ chmod a+x conky-startup.sh
  ```

* Then created the background process depending on your distro, here you can find the steps for Ubuntu:
![](./.img/image0.png)

* It will be the first time, so click on **Add**.

* And that's it, you need to restart/logout and when login you will see conky on your Desktop:
![](./.img/image1.png)

*  Feel free to customize it as you desire. Enjoy!!