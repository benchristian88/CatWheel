# CatWheel
## ESP Home Cat Wheel Speed and Distance Project
This respository is here to show how I made a speedo for my Cat Wheel using ESP home, Home Assistant, and a few parts from the local electronics shop. 

This is **Coco**, our beautiful Tonkinese girl. She is the kind of cat who follows you around the house and the moment you stop, she is in your lap.  She also loves her [Ferris Cat Wheel International](https://ferriscatwheelinternational.com), and uses it most days and all through the night.  

![IMG_3776](https://github.com/user-attachments/assets/e44443c8-30f7-4ac7-b027-35624550737f)

## The Goal
The goal of this project was to recreate a bike speedo and get the measurements into Home Assistant so that we knew how fast Coco ran on wheel and the total distance she had gone.  Then we can give Coco a reward from the Xiaomi Cat Feeder.  She figured it out really quickly too.  

## The Project

First things, we needed a trip to the local electronics store to get an ESP32 Development Board, a Hall Sensor, and small plastic box to put it all in.  Needed to get some wire too to cable the ESP to the Hall Sensor and a magnet.

### Parts List
1. [ESP32 Board](https://www.jaycar.co.nz/duinotech-esp32-main-board-with-wi-fi-and-bluetooth/p/XC3800)
2. [ESP Compatible Hall Sensor](https://www.jaycar.co.nz/duinotech-arduino-compatible-hall-effect-sensor/p/XC4434)
3. [Magnet](https://www.jaycar.co.nz/rare-earth-magnet-medium/p/LM1618)
4. [Plastic Box](https://www.jaycar.co.nz/jiffy-case-imac-blue-ub5/p/HB6004)
5. [Light Gauge Wire](https://www.jaycar.co.nz/light-duty-hook-up-wire-pack-8-colours/p/WH3009)
6. USB cable to power the ESP.

### Wiring
What we needed to do here is wire up the Hall Sensor +,-, and S to the ESP.  

1. We cabled the S pin on the Hall to the D13 on the ESP. You can pick any pin, but note in the Sensor Code below, you need to pick the Pin Number you wire up. I picked Pin 13, lucky for some.
2. Wire the negative - Hall pin to GND on the ESP
3. And the + to the VIN   (i think probably should wire to the 3.3V and not VIN, thinking about that, but it works, so oh well)

Three cables, cant really go wrong here. I used hookup wire that plugged into the boards.  If you have a soldering iron, do it that way.  Up to you.

![IMG_4781](https://github.com/user-attachments/assets/28c6fbcd-0b07-4b06-87eb-8014a235ad8b)

Now put it all in the Platic Box you got, using some tape or glue to align the Hall Sensor to the outer edge of the box so when you install it on your wheel base, the Hall Sensor is as close as possible to the rotating magnet you install on the wheel.

![IMG_4778](https://github.com/user-attachments/assets/72dbbfbe-83e1-4644-96bb-ae0ae0378af0)

  

## Coding
This isnt going to be a lesson on ESPHome, as there are many good tutorials out there. like here.
[Getting Started with ESP Home](https://esphome.io/guides/getting_started_hassio.html)
Get ESPHome Installed, plug in your device via a usb cable, and follow the instructions to initialize the ESP board.   Once that is done you are ready to create your sensors.  The Adding Features section of the ESP Home Getting started is where it explains how to add the sensors you want.  The below code can likely be copied into your ESP Home YAML as it is.  

### How it works
The key feature we are using here is the Pulse Meter, that counts every time the wheel goes around, with a magnet attached to the wheel, going past the Hall Sensor.  This time between pulses is then able to be determined and that tells us how fast the wheel is going.  To work that out though, we need to figure out how big your wheel is, and create a multiplier in the ESPHome code to convert pulses to Speed.  We have added an [Excel Spreadsheet](https://github.com/benchristian88/CatWheel/blob/main/ESP32%20Circumference%20Calculator.xlsx) to the resposity to calculate these for you.

The Spreadsheet gives you a Speed Multipler, to add to the *Cat Wheel Speed Sensor*, and a Distance Multipler, to add to the *total* sensor.

My wheel is 120cm wide, so I would put that into the calculator, which would give me a mutiplier of 0.2262.    I added **two** magnets to my wheel so I divided the mutliplier by two.  Hence the code below shows 0.114.  You dont need to do that, and if you only have one magnet, just use the multipler provided.

You need to note down the multiplier for the Total distance and add that in as well.   

I also added another sensor that gives a Yes or No to there being activity.  eg. it shows Yes when the wheel is in use.  I found this easier to use for Automations.  

There are a few other settings in the code that deal with things like spin up time of the wheel. When Coco first gets on, and starts running, there you want to wait for few revolutions till you start working out the speed.  The skip_initial waits for three pulses before starting to work it out. The Delta filter and the timeout 10s work together to stop recording when the wheel slows to zero.  Otherwise the ESP will keep thinking its just going really slow, and be waiting for the next reading. This way after 10 seconds of no activity, it shuts down. 

### ESP Sensors
These are the ESP Home sensors that I added to the ESP Yaml.  Full ESPHome code in the files.
```
sensor:
  - platform: pulse_meter
    pin: 13
    name: 'Cat Wheel Speed'
    id: 'catwheel'
    unit_of_measurement: 'km/hr'
    accuracy_decimals: 2
    filters:
      - multiply: 0.114
      - skip_initial: 3
      - delta: 1%
    timeout: 10s
    total: 
      unit_of_measurement: 'km'
      name: 'Cat Wheel Distance'
      filters:
        - multiply: 0.0019
      accuracy_decimals: 2


binary_sensor:
  - platform: template
    name: "Cat Wheel Activity"
    lambda: |-
      if (id(catwheel).state > 1.0) {
        // wheel is in use
        return true;
      } else {
        // not in use
        return false;
      }  
```

You will also need to Add the ESP Home Integration to Home Assistant if you havent already done that so the sensors show up and you should be ready to go.

## Installation
The whole project was then installed on the cat wheel with some double sided tape, on the feet, so that when the wheel rolled around there was still enough clearance, but close.  We then tape the magnet to the wheel, so every time the wheel goes around the magnet passes the hall sensor and triggers a pulse.  This is then read by the ESP code and sent to Home Assistant.  You can see in this photo, the magnet is triggering the Hall Sensor and the light is coming on.  This shows each rotation.
![IMG_4786](https://github.com/user-attachments/assets/222e008a-54ab-4604-acd9-19707f521127)

## Time for a Beer
Assuming you got it all working, you should now be able to create a few cards in Home Assistant, add some automations, and now be having fun watching how fast your cats can run.  Coco maxes out about 8km/hour, which isnt bad for a shortarse.  This is what it looks like on my home assistant dashboard.
![IMG_3778](https://github.com/user-attachments/assets/cd16356b-1e30-4564-a57b-9446d26a1d2b)


## Roadmap
Like any good cat project, we need a roadmap.  Right now, the way this works, the total distance measure isn't persistent, so when you update the ESPHome version, the total distance is reset to zero. This makes Coco very sad as she doesnt know how far she has been.  Here are a few other ideas to make it better. I am hoping the community jumps in and we make this thing awesome.  
**Roadmap Items**
1. Persistant Total Distance to survive reboots
2. More sensors - acceleration
3. Custom Made 3D Printed Slimline Box 
4. Local Speed Display
5. Grafana Dashboards (half done already)
6. Microchip Reader to identify cat present
7. Coco Youtube channel - online footage and alerts when she is running on the wheel

## Finished Product
A happy Coco, enjoying her time on the wheel, keeping fit, and inside away from the big bad world.
![IMG_3775](https://github.com/user-attachments/assets/2ea086f3-e726-45d9-81dd-2381aa6705d7)


   
