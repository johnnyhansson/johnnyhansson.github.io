---
layout: post
title: Control 433 Mhz switch with Raspberry Pi and Google Calendar
categories: Raspberry Pi
---

In a couple of weeks, most windows here in Sweden will be filled with electric candels.
So why not use a Raspberry Pi, a couple of 433 Mhz remote controlled switches, Google Calendar and some code to decide when they should be turned on or off?

### Parts list

This is the parts I have used to build this project.

* Raspberry Pi Model B (Rasbian Stretch Lite with release date 2017-09-07)
* A breadboard including wires
* A 17.3 cm long wire used as a antenna
* [Remote Controlled switches using the frequency 433 Mhz](https://www.kjell.com/se/sortiment/el-verktyg/smarta-hem/433-mhz/fjarrstrombrytare/utanpaliggande-brytare/luxorparts-fjarrstrombrytare-2000-w-3-pack-p50219)
* [A 433 Mhz receiver and transmitter](https://www.kjell.com/se/sortiment/el-verktyg/elektronik/fjarrstyrning/sandar-och-mottagarmodul-433-mhz-p88905)

### Get transmission codes

Before we can control the switches we need to obtain the codes sent by the hand transmitter. The image below shows how to setup the receiver with the Raspberry Pi.
To extend the range of the receiver or transmitter you can take a 17.3 cm long wire and attach it to the whole named `ANT` on the modules.

<img src="/images/control-433mhz-switch/receiver.png" />

After the receiver have been set up properly we need to clone and build wiringPi 

```bash
git clone git://git.drogon.net/wiringPi
cd wiringPi
./build
```

and then clone and build 433Kit.

```bash
git clone --recursive git://github.com/ninjablocks/433Utils.git
cd 433Utils/RPi_utils
make
```

When 433Kit has been built we have two new applications, `RFSniffer` and `codesend`. 
Start RFSniffer, point the hand transmitter towards the receiver module and push a button on the transmitter. If everything is properly connected, RFSniffer will print a code to the terminal.

```bash
sudo ./RFSniffer
```

### Setup transmitter

When you have all the codes from the hand transmitter, it's time to disconnect the receiver module and connect the transmission module. The image below shows how to setup the transmitter with the Raspberry Pi.

<img src="/images/control-433mhz-switch/transmitter.png" />

If everything is connected according to the image, you should be able to send one of the codes obtained from the hand transmitter to turn a switch on or off.

```bash
sudo ./codesend 1394007
```

### Control the switches with Google Calendar

Now its time to control the switches using a small application I wrote a couple of years ago. The application is written in C# and parses iCalendar data to decide if a switch should be turned on or off, based on the name of events in the calendar. The source for the application can be found on [Github](https://github.com/johnnyhansson/CandleBridgeSteering).

To get it running on the Raspberry Pi, we first need to install Mono.

```bash
sudo apt-get install mono-complete
```

After a few minutes, Mono is installed and we can download the Candle Bridge Steering application.

```bash
wget https://github.com/johnnyhansson/CandleBridgeSteering/releases/download/2017-11-04/CandleBridgeSteering-20171104.tar

mkdir cbs 
tar -xf CandleBridgeSteering-20171104.tar -C cbs
cd cbs
```

In the `cbs` directory you will find a file named `CandleBridgeSteering.exe.config`. This file contain the applications configuration and need to be updated before running the application. 

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <configSections>
    <section name="candleBridgeSettings" type="CandleBridgeSteering.Configuration.CandleBridgeSection, CandleBridgeSteering" />
  </configSections>

  <candleBridgeSettings>
    <candleBridges>
      <candleBridge name="Livingroom" onCode="1394007" offCode="1394004" />
    </candleBridges>
  </candleBridgeSettings>
  
  <appSettings>
    <add key="CalendarUrl" value="https://calendar.google.com/calendar/ical/xxx/basic.ics" />
    <add key="CodesendLocation" value="/home/pi/433Utils/RPi_utils/codesend" />
  </appSettings>
</configuration>
```

The section named `candleBridges` can contain multiple `candleBridge` elements, each element correspond to a single switch. The `name` attribute of a `candleBridge` element must match the name of a calendar event in the calendar. The `onCode` and `offCode` must match the code sent by the hand transmitter when turning the switch on or off.

In the image below the we can see a calendar event with a name matching the candleBridge element in the example configuration. This calendar event will keep the switch on between 4 pm to 11 pm.

<img src="/images/control-433mhz-switch/calendar.png" />

`CalendarUrl` should be set to an URL where the application can obtain the calendar information in iCalendar format. To get the URL for your Google Calendar, go to the calendar settings and scroll down to the `private address`. More information can be found [here](https://support.google.com/calendar/answer/37648?hl=en). `CodesendLocation` should be set to the full path where the codesend application can be found.

When everything is configured you should be able to run the application to either turn on or turn off your switches based on the events in your calendar.

```bash
sudo mono CandleBridgeSteering.exe
```

### Schedule the CandleBridgeSteering application

The last thing we need to do is to set up a recurring job that will execute the CandleBridgeSteering application.

First we will edit our crontab file.

```bash
sudo crontab -e
```

Then add a recurring task. In the example below the CandleBridgeSteering application will be called once every minute.

```bash
* * * * * mono /home/pi/cbs/CandleBridgeSteering.exe
```

Save the changes to your crontab file and enjoy!