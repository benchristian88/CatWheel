# CatWheel
## ESP Home Cat Wheel Speed and Distance Project
This respository is here to show how I made a speedo for my Cat Wheel using ESP home, Home Assistant, and a few parts from the local electronics shop. 

This is COCO, our beautiful Tonkinese girl. She is the kind of cat who follows you around the house and the moment you stop, she is in your lap.  She also loves her [Ferris Cat Wheel International](https://ferriscatwheelinternational.com), and uses it most days and all through the night.  

![IMG_3776](https://github.com/user-attachments/assets/e44443c8-30f7-4ac7-b027-35624550737f)

## The Goal
The goal of this project was to recreate a bike speedo and get the measurements into Home Assistant so that we knew how fast Coco ran on wheel and the total distance she had gone.  Then we can give Coco a reward from the Xiaomi Cat Feeder.  She figured it out really quickly too.  




## ESP Sensors

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

## Roadmap
Like any good cat project, we need a roadmap.  Right now, the way this works, the total distance measure isn't persistent, so when you update the ESPHome version, the total distance is reset to zero. This makes Coco very sad as she doesnt know how far she has been.  
