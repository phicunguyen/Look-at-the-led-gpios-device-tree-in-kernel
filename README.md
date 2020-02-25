# Look-at-the-led-gpios-device-tree-in-kernel
My previous "Raspberry-P4-enable-gpio-to-do-heartbeat-using-device-tree" that shows how to enable heartbeat on gpio5/6 using device tree. Now we're going to look at how kernel adding the MYLED0/ MYLED1/ to the sys/class/leds. 

                  /dts-v1/;
                  /plugin/;

                  / {
                      compatible = "brcm,bcm2708";

                      fragment@0 {
                          target = <&leds>;
                          __overlay__ {
                              my_led0: myled0 {
                                  label = "MYLED0";
                                  linux,default-trigger = "heartbeat";
                                  gpios = <&gpio 5 0>;
                              };
                          };
                      };
                      fragment@1 {
                          target = <&leds>;
                          __overlay__ {
                               my_led1: myled1 {
                                  label = "MYLED1";
                                  linux,default-trigger = "heartbeat";
                                  gpios = <&gpio 6 0>;
                              };
                          };
                      };
                  };
      
      pi@raspberrypi:~ $ ls /sys/class/leds/
      led0/   led1/   MYLED0/ MYLED1/ 
      
The gpio_led_probe function in leds-gpio.c in linux/drivers/leds/ will be called.

      static struct platform_driver gpio_led_driver = {
              .probe          = gpio_led_probe,
              .shutdown       = gpio_led_shutdown,
              .driver         = {
                      .name   = "leds-gpio",
                      .of_match_table = of_gpio_leds_match,
              },
      };

then, it calls the gpio_leds_create.  

      gpio_leds_create
         device_for_each_child_node    
            fwnode_property_read_string         //"label", &led.name
            devm_fwnode_get_gpiod_from_child    //led.name
            fwnode_property_read_string         //"linux,default-trigger"
               fwnode_property_read_string      //"default-state"
            create_gpio_led
            
    ret = fwnode_property_read_string(child, "label", &led.name);      

     device_for_each_child_node
              led.name: MYLED1
              offset: 6
              led.name: MYLED0
              offset: 5
              led.name: led0
              offset: 29
              led.name: led1
              offset: 2

