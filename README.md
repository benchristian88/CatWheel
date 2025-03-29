# CatWheel
## ESP Home Cat Wheel Speed and Distance Project
This respository is here to show how I made a speedo for my Cat Wheel using ESP home, Home Assistant, and a few parts from the local electronics shop. 

This is COCO, our beautiful Tonkinese girl. She is the kind of cat who follows you around the house and the moment you stop, she is in your lap.  She also loves her [Ferris Cat Wheel International](https://ferriscatwheelinternational.com), and uses it most days and all through the night.  

![IMG_3776](https://github.com/user-attachments/assets/e44443c8-30f7-4ac7-b027-35624550737f)

## The Goal
The goal of this project was to recreate a bike speedo and get the measurements into Home Assistant so that we knew how fast Coco ran on wheel and the total distance she had gone.  Then we can give Coco a reward from the Xiaomi Cat Feeder.  She figured it out really quickly too.  

## The Project

First things, we needed a trip to the local electronics store to get an ESP32 Development Board, a Hall Sensor, and small plastic box to put it all in.  Needed to get some wire too to cable the ESP to the Hall Sensor and a magnet.

### Parts List
[ESP32 Board](https://www.jaycar.co.nz/duinotech-esp32-main-board-with-wi-fi-and-bluetooth/p/XC3800)
[ESP Compatible Hall Sensor](https://www.jaycar.co.nz/duinotech-arduino-compatible-hall-effect-sensor/p/XC4434)
[Magnet](https://www.jaycar.co.nz/rare-earth-magnet-medium/p/LM1618)
[Plastic Box](https://www.jaycar.co.nz/jiffy-case-imac-blue-ub5/p/HB6004)
[Light Gauge Wire](https://www.jaycar.co.nz/light-duty-hook-up-wire-pack-8-colours/p/WH3009)

What we needed to do here is wire up the Hall Sensor +,-, and S to the ESP.  

1. We cabled the S pin on the Hall to the D13 on the ESP,
2. - to GND
3. + to the VIN

Three cables, cant really go wrong here.

![IMG_4781](https://github.com/user-attachments/assets/28c6fbcd-0b07-4b06-87eb-8014a235ad8b)

## Coding
[Getting Started with ESP Home](https://esphome.io/guides/getting_started_hassio.html)


### ESP Sensors

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

## Installation


## Roadmap
Like any good cat project, we need a roadmap.  Right now, the way this works, the total distance measure isn't persistent, so when you update the ESPHome version, the total distance is reset to zero. This makes Coco very sad as she doesnt know how far she has been.  
