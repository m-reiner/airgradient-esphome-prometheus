# AirGradient - Using ESPHome with a Prometheus Exporter

## Background
AirGradient has a [DIY air sensor](https://www.airgradient.com/diy/) with parts that can be purchased directly from them in the form of a kit or individually via sites like AliExpress, eBay, and Amazon.  If you want to support AirGradient's project, I strongly suggest you purchase one of their kits, though I have been able to find the parts cheaper by sourcing them directly through some of the sites mentioned.  AirGradient has their own Arduino firmware version you can compile and install on the ESP8266 microntroller at the heart of this project, and if you purchased a kit from them you can have it ship the data off to their servers so you can view it at any time online.  It looks like that dashboard access is only free for the first six (basic), twelve (pro), or 24 (pro pre-soldered) months.  Of course, if you want to use this hardware, whether purchased in a kit from AirGraident or sourced by yourself, read on!

After seeing Jeff Geerling post a [YouTube video](https://www.youtube.com/watch?v=Cmr5VNALRAg) about the air quality in his office and how it negatively impacted him, it inspired me to build my own.  Not only did he make a great video, but he created the [airgradient-prometheus project](https://github.com/geerlingguy/airgradient-prometheus) to export your sensor data to a small web page run on your ESP8266 which could then be scraped using a Prometheus job.  From there, you can create good looking and useful dashboards by leveraging tools like Grafana.  I even took it a step further and added REST entries in my HomeAssistant's configuration.yaml to scrape the values and make them available to create notifications when those sensor values went beyond certain limits.  It wasn't super efficient to do it this way, but it worked.

Fast forward to this week when I got a newsletter form AirGradient letting me know they had updated one of their Arduino libraries to make their CO2 sensor readings more accurate.  This was something I wanted, but it meant I had to go to each of my sensors, take them down, download the updated library in the Arduino IDE, re-compile the code, flash it over to each sensor via USB, and hang each sensor back up.  This wasn't ideal, so I set out to make things easier in the future.  After all, I already had the sensors down anyway.

Since first building these sensors, I have massively expanded my [Home Assistant](https://www.home-assistant.io/) setup and had begun dabbling in creating new sensors using [ESPHome](https://esphome.io/).  One of the huge benefits to ESPHome is its tight integration into Home Assistant, allowing devices flashed using their firmware to easily report data back to Home Assistant to be displayed and actioned upon.  This seemed ideal to me, but I really liked my existing Grafana dashboard, so I didn't want to lose the Prometheus exporter in the conversion.  

Using [ajfriesen/ESPHome-AirGradient](https://github.com/ajfriesen/ESPHome-AirGradient) as a model to help me set up my sensors, mixing in some of [geerlingguy/airgradient-prometheus](https://github.com/geerlingguy/airgradient-prometheus) exporter magic, and lots of internet searching, I built this.

## How it Works

If you're using one of the official [AirGradient Arduino firmwares](https://www.airgradient.com/open-airgradient/instructions/basic-setup-skills-and-equipment-needed-to-build-our-airgradient-diy-sensor/), you can configure it to enable WiFi and send data to a remote server every 9 seconds as it cycles through the display of PM2.5, CO2, temperature, and humidity values.  By default, it sends a small JSON payload to AirGradient's servers, and you can monitor the data via their service.  As mentioned previously, their serivce is only free for between six and 24 months depending on which kit you purchased from them.

Like [geerlingguy/airgradient-prometheus](https://github.com/geerlingguy/airgradient-prometheus), this sketch provides stats upon request to the Prometheus server during scrapes.  The twist I put on it was to leverage ESPHome's already tight integration with Home Assistant to also expose your sensors directly to your HA instance.  Additionally, as ESPHome supports wireless flashing over WiFi, future updates should not require you to take down your sensor to upgrade it.

## How to Use

First, edit the configuration options in the `Config` section of `AirGradient-DIY.ino` to match your desired settings.

Make sure you enter the SSID and password for your WiFi network in the relevant variables, otherwise the sensor won't connect to your network.

Make sure you have the "AirGradient" library installed (in Arduino IDE, go to Tools > Manage Libraries...).

Upload the sketch to the AirGradient sensor ([instructions here](https://www.jeffgeerling.com/blog/2021/airgradient-diy-air-quality-monitor-co2-pm25#flashing)) using Arduino IDE, make sure it connects to your network, then test connecting to it by running this `curl` command:

```sh
$ curl http://airgradient-ip-address:9926/metrics
# HELP pm02 Particulate Matter PM2.5 value
# TYPE pm02 gauge
pm02{id="Airgradient"}6
# HELP rco2 CO2 value, in ppm
# TYPE rco2 gauge
rco2{id="Airgradient"}862
# HELP atmp Temperature, in degrees Celsius
# TYPE atmp gauge
atmp{id="Airgradient"}31.6
# HELP rhum Relative humidity, in percent
# TYPE rhum gauge
rhum{id="Airgradient"}38
```

Once you've verified it's working, configure Prometheus to scrape the sensor's endpoint: `http://sensor-ip:9926/metrics`.

Example job configuration in `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'airgradient-bedroom'
    metrics_path: /metrics
    scrape_interval: 30s
    static_configs:
      - targets: ['airgradient-ip-address:9926']
```

### Static IP address

You can configure a static IP address for the sensor by uncommenting the `#define staticip` line and entering your own IP information in the following lines.

## License

MIT.

## Authors

  - [Jeff Geerling](https://www.jeffgeerling.com)
  - [Jordan Jones](https://github.com/kashalls)
