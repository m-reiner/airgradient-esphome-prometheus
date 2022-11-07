# AirGradient - Using ESPHome with a Prometheus Exporter

## Background
AirGradient has a [DIY air sensor](https://www.airgradient.com/diy/) with parts that can be purchased directly from them in the form of a kit or individually from sites like AliExpress, eBay, and Amazon.  If you want to support AirGradient's project, I strongly suggest you purchase one of their kits, though I have been able to find the parts cheaper by sourcing them directly through some of the sites mentioned.  AirGradient has their own Arduino firmware version you can compile and install on the ESP8266 microntroller at the heart of this project, and if you purchased a kit from them you can have it ship the data off to their servers so you can view it at any time online.  It looks like that dashboard access is only free for the first six (basic), twelve (pro), or 24 (pro pre-soldered) months.  Of course, if you want to use this hardware locally with Home Assistant and Prometheus, whether purchased in a kit from AirGradient or sourced by yourself, read on!

After seeing Jeff Geerling post a [YouTube video](https://www.youtube.com/watch?v=Cmr5VNALRAg) about the air quality in his office and how it negatively impacted him, it inspired me to build my own air quality sensor.  Not only did he make a great video, but he created the [geerlingguy/airgradient-prometheus](https://github.com/geerlingguy/airgradient-prometheus) project to help you export your sensor data to a small web page run on your ESP8266 which could then be scraped using a Prometheus job.  From there, you can create good looking and useful dashboards by leveraging tools like Grafana.  If all you want to do is export your sensor data to be scraped by a local Prometheus instance and you’re comfortable using the Arduino IDE, I suggest you leverage Jeff’s project.  I originally took it a step further and added REST entries in my Home Assistant's configuration.yaml to scrape the values and make them available to create notifications when those sensor values went beyond certain limits.  It wasn't super efficient to do it this way, but it worked.

Fast forward to early November when I received a newsletter form AirGradient letting me know they had updated one of their Arduino libraries to make their CO2 sensor readings more accurate.  This was something I wanted, but it meant I had to go to each of my sensors, take them down, download the updated library in the Arduino IDE, re-compile the code, flash it over to each sensor via USB, and hang each sensor back up.  This wasn't ideal, so I set out to make things easier in the future.  After all, I already had the sensors down anyway.

Since first building these sensors, I have massively expanded my [Home Assistant](https://www.home-assistant.io/) setup and had begun dabbling in creating new sensors using [ESPHome](https://esphome.io/).  One of the huge benefits to ESPHome is its tight integration with Home Assistant, allowing devices flashed using their firmware creation tool to easily report data back to Home Assistant to be displayed and actioned upon.  This seemed ideal to me, but I really liked my existing Grafana dashboard so I didn't want to lose the Prometheus exporter in the conversion.  

Using [ajfriesen/ESPHome-AirGradient](https://github.com/ajfriesen/ESPHome-AirGradient) as a model to help me set up my sensors, mixing in some of [geerlingguy/airgradient-prometheus](https://github.com/geerlingguy/airgradient-prometheus) exporter magic, and after lots of internet searching and research, I managed to build this.

## How it Works

If you're using one of the official [AirGradient Arduino firmwares](https://www.airgradient.com/open-airgradient/instructions/basic-setup-skills-and-equipment-needed-to-build-our-airgradient-diy-sensor/), you can configure it to enable WiFi and send data to a remote server every 9 seconds as it cycles through the display of PM2.5, CO2, temperature, and humidity values.  By default, it sends a small JSON payload to AirGradient's servers, allowing you to monitor the data via their web-based service.  As mentioned previously, their service is only free for between six and 24 months depending on which kit you purchased from them.

Like [geerlingguy/airgradient-prometheus](https://github.com/geerlingguy/airgradient-prometheus), my sketch provides stats upon request to the Prometheus server during scrapes.  The twist I put on it was to leverage ESPHome's already tight integration with Home Assistant to also expose your sensors directly to your HA instance.  Additionally, as ESPHome supports wireless flashing over WiFi, future updates should not require you to take down your sensor to upgrade it.

## How to Use

For the purposes of this guide, I will assume that you have a [working instance of Home Assistant](https://www.home-assistant.io/installation/) and that you were able to set up ESPHome on your Home Assistant server.  If you need help installing ESPHome, you can click the button below to begin the installation process or follow [this link](https://esphome.io/guides/getting_started_hassio.html) to read more.

[![Open the ESPHome Add-n.](https://my.home-assistant.io/badges/supervisor_addon.svg)](https://my.home-assistant.io/redirect/supervisor_addon/?addon=5c53de3b_esphome&repository_url=https%3A%2F%2Fgithub.com%2Fesphome%2Fhome-assistant-addon)

### Prep work:
  1. Navigate to ESPHome from within your Home Assistant server's homepage
  2. Click the green "+ New Device" button
  3. In the 'New device' wizard that pops up, select the following:
     - Click 'Continue' on the first screen
     - Put whatever name you'd like when prompted and click 'Next'
     - Select ‘ESP8266’ on the 'Select your device type' screen and click 'Next'
     - Click the 'SKIP' button when the wizard offers to install your new configuration
     - Click the 'EDIT' button under your new device configuration 
     - Copy everything in that YAML file and paste it into Notepad or somewhere else for safekeeping
        - You’ll use some of the default/generated information later

### Configure your new ESPHome device to be an air quality sensor with a Prometheus exporter:
  1. Ensure you have copied the auto-generated skeleton code ESPHome created after the previous section to Notepad and that your device's YAML file in ESPHome is now empty
  2. Copy the code in my example [office-aq-sensor.yaml](https://github.com/m-reiner/airgradient-esphome-prometheus/blob/master/office-aq-sensor.yaml) and paste it into your device's YAML file within ESPHome  
  3. Edit the fields below as needed.  Most of this will come from the auto-generated skeleton configuration you copied out of your device's YAML file into Notepad during the prep work phase above.  I suggest you keep anything that is between quotation marks in my example within quotation marks in your configuration.  
     **NOTE: be careful with your spacing, as improper spacing can lead to compiler errors in ESPHome!**
     
  ```yaml
  - name: office-aq-sensor     # (Line 3) Paste in the name from the skeleton code you copied into Notepad 
  - key: "PleaseChangeMe"      # (Line 13) Paste the key from the skeleton code you copied into Notepad
  - password: "PleaseChangeMe" # (Line 16) Paste the password from the skeleton code you copied into Notepad
  - ssid: PleaseChangeMe       # (Line 27) Enter in your WiFi network name (SSID)
  - password: PleaseChangeMe   # (Line 28) Enter your WiFi network password
  - ssid: "PleaseChangeMe"     # (Line 38) Paste in the network name (SSID) of the fallback hotspot from the skeleton code you copied into Notepad
  - password: "PleaseChangeMe" # (Line 39) Paste in the password of the fallback hotspot from the skeleton code you copied into Notepad
  ```
  4. Click "SAVE" in the upper right-hand corner and "X" out of the editor
  
### Install the updated code in your air quality sensor:
  1. Connect your sensor via USB to the computer you have running your Home Assistant server
  2. Click the three vertical dots below your sensor name within your ESPHome page
  3. Select 'Install'
  4. Select 'Plug into the computer running ESPHome Dashboard'
  5. Select the correct device from the list 

Please address any errors the compiler may throw at you if ESPHome has changed things since this guide was written.  After the firmware compiles and installs, which can take several minutes depending on how fast the computer you have running Home Assistant is, you should start seeing log output data from your sensor that includes the wireless network connection and air quality values.  If you get an error after the installation completes successfully and you don't see logs, you can leave that window, click the 'LOGS' button for your sensor on the ESPHome page and select 'Plug into computer running ESPHome Dashboard' if the sensor is still connected to the computer you have running Home Assistant (otherwise, you can select 'Wirelessly' provided your sensor successfully connected to WiFi).

To validate that your device is on WiFi and the Prometheus portion is working correctly, connect to your sensor by running this `curl` command:

```sh
#TYPE esphome_sensor_value GAUGE
#TYPE esphome_sensor_failed GAUGE
esphome_sensor_failed{id="temperature",name="Temperature"} 0
esphome_sensor_value{id="temperature",name="Temperature",unit="°C"} 24.0
esphome_sensor_failed{id="humidity",name="Humidity"} 0
esphome_sensor_value{id="humidity",name="Humidity",unit="%"} 40.2
esphome_sensor_failed{id="particulate_matter_10m_concentration",name="Particulate Matter <1.0µm Concentration"} 0
esphome_sensor_value{id="particulate_matter_10m_concentration",name="Particulate Matter <1.0µm Concentration",unit="µg/m³"} 0
esphome_sensor_failed{id="particulate_matter_25m_concentration",name="Particulate Matter <2.5µm Concentration"} 0
esphome_sensor_value{id="particulate_matter_25m_concentration",name="Particulate Matter <2.5µm Concentration",unit="µg/m³"} 0
esphome_sensor_failed{id="particulate_matter_100m_concentration",name="Particulate Matter <10.0µm Concentration"} 0
esphome_sensor_value{id="particulate_matter_100m_concentration",name="Particulate Matter <10.0µm Concentration",unit="µg/m³"} 0
esphome_sensor_failed{id="senseair_co2_value",name="SenseAir CO2 Value"} 0
esphome_sensor_value{id="senseair_co2_value",name="SenseAir CO2 Value",unit="ppm"} 602
#TYPE esphome_switch_value GAUGE
#TYPE esphome_switch_failed GAUGE
esphome_switch_failed{id="flash_mode_safe_mode",name="Flash Mode (Safe Mode)"} 0
esphome_switch_value{id="flash_mode_safe_mode",name="Flash Mode (Safe Mode)"} 0
```

You'll notice that the output from the ESPHome version of this sensor is different than the output from [geerlingguy/airgradient-prometheus](https://github.com/geerlingguy/airgradient-prometheus).  I'm not well versed enough in the ESPHome Prometheus component at the moment to know if there's a way to have it format its output like Jeff's project, but it shouldn't be too big a deal and will really only require some minor updates to your queries in Grafana if you're migrating from his project to mine.  

### Prometheus

If you're setting up the Prometheus exporter, there's a good chance that you'll also be setting up Prometheus to regularly scrape the output of the /metrics page your new sensor is now sharing.  To help you along, here is an example [Prometheus scrape configuration](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config):

```yaml
scrape_configs:
  - job_name: AQS-Office
    honor_timestamps: true
    scrape_interval: 30s
    scrape_timeout: 10s
    metrics_path: /metrics
    scheme: http
    static_configs:
    - targets:
      - 'airgradient-ip-address:9926'
```
### Grafana

If you're using Prometheus, there's a good chance you're also going to use [Grafana](https://grafana.com/) (or something similar) to make pretty dashboards to display all that data you're collecting.  To help you get started, you'll find the queries I'm currently using in Grafana below.  Just replace the `AQS-Office` job name with whatever job name you gave your sensor(s) in the scrape_configs portion of your Prometheus configuration file.  I am currently using the queries below within gauges and time series graphs so I can see the data in different ways.

  1. Carbon Dioxide concentration (PPM)
     - `esphome_sensor_value{job="AQS-Office",id="senseair_co2_value"}`
  2. Particulate Matter (PM2.5)
     - 24 hour average: `avg_over_time(esphome_sensor_value{job="AQS-Office",id="particulate_matter_25m_concentration"}[24h])`
     - Current value:   `esphome_sensor_value{job="AQS-Office",id="particulate_matter_25m_concentration"}`
  3. Relative humidity (%)
     - `esphome_sensor_value{job="AQS-Office",id="humidity"}`
  4. Temperature
     - Celsius:    `esphome_sensor_value{job="AQS-Office",id="temperature"}`
     - Fahrenheit: `sum(esphome_sensor_value{job="AQS-Office",id="temperature"}*1.8+32)`

## License

MIT.

## Credits

  - [Matt Reiner](https://github.com/m-reiner) - this cobbled-together mess
  - [Jeff Geerling](https://www.jeffgeerling.com) - original [airgradient-prometheus project](https://github.com/geerlingguy/airgradient-prometheus)
  - [Andrej Friesen](https://github.com/ajfriesen) - original [ESPHome-AirGradient project](https://github.com/ajfriesen/ESPHome-AirGradient)
