# Home Assistant - Amber Electric Usage Charts
### Usage charts in Home Assistant for Amber Electric pricing

This is used in Home Assistant with the Amber Electric integration to estimate usage in kilowatt hours and dollars. Example charts using the [ApexCharts Card HACS frontend integration](https://github.com/RomRider/apexcharts-card) are included.

Using prices from the Amber Electric integration, import and export prices are recorded at the end of each 30 minute period, as well as the cost/profit for each 30 minute period.



_Amber-Styled Charts_  
![image](https://github.com/melvanderwal/HA-Amber-Electric-Usage-Charts/assets/25993713/5f0f80a9-9b96-4edb-92a8-9d8163116f15)

_Combined Energy and Cost Charts_  
![image](https://github.com/melvanderwal/HA-Amber-Electric-Usage-Charts/assets/25993713/32751934-cba7-440b-94a3-0b7d386ff2f4)

*Prerequisites:* 
* The [Amber Electric Home Assistant integration](https://www.home-assistant.io/integrations/amberelectric).
* A Home Assistant inverter integration which provides import and export power (or a power sensor from which import and export can be derived).
  * The example code uses the [GoodWe integration](https://www.home-assistant.io/integrations/goodwe/).
* The [ApexCharts Card HACS frontend integration](https://github.com/RomRider/apexcharts-card).
  * This integration is not required if you choose to build your own charts with a different integration.

Please note that the values will not match exactly with Amber's reporting, as:
* The power values reported by your inverter are unlikely to exactly match what is recorded by your smart meter and sent to Amber
* Any Home Assistant downtime will not have logging of power or prices
* The clock on your HA computer may not be in sync with Amber's clock, so the 30 minute period may be shifted slightly

For example, my inverter typically reports exported power about 3.5% lower than what is received by Amber. In the `inverter_import_power` and `inverter_export_power` template sensors there is a `correctionFactor` variable which helps to compensate for this - I have it set to 1.035.

_Sensor Flow_  
need a new diagram

The YAML is provided as a package. It can be implemented by adding the following to `configuration.yaml`:
```
# Use packages in /package folder
homeassistant:
  packages: !include_dir_named package
```
and then copying `amber_usage.yaml` to `config/package/amber_usage.yaml`. Do not copy `amber_usage_part2.yaml` to the package folder.

To get it running, the only changes you should need to make are in the template sensors and the automation.
* Implement and test the template sensors first by adding `amber_usage.yaml` as a package and updating the code as described below.
  * Do whatever is required to have `inverter_import_power` and `inverter_export_power` return power as positive units in kW.
    * Start with a `correctionFactor` value of 1.
    * Confirm that the sensors are returning correct import and export values as a positive number in kW.
* Add the remaining code by appending the content of `amber_usage_part2.yaml` to `amber_usage.yaml`.
  * Update `amber_30_minute_import_cost_export_price` by updating `sensor.your_amber_general_price` and `sensor.your_amber_feed_in_price` to the corresponding sensor names from your Amber Electric integration.
  * Use Developer Tools to confirm that no errors are being raised.
* Restart Home Assistant.
  * Reimann Sum sensors
    * They won't exist until you have restarted Home Assistant.
    * The Riemann Sum sensors won't start logging data until non-zero data is coming from your template sensors. Give it some time.
  * Utility Meter sensors.
    * These are derived from the Riemann Sum sensors, so they won't exist until you have restarted Home Assistant and the Riemann Sum sensors have data.
* Add chart cards to a dashboard.
  * The provided charts are only intended as examples - build your own to suit your needs.
* Over a few days, note the difference between what your inverter has reported in kW to what is reported in Amber's app.
  * In the power template sensors, `correctionFactor` is a multiplier to adjust the kW of your inverter sensor to be closer to what is reported by your smart meter to Amber.  Adjust the value accordingly.

Notes:  
* If you look at the built-in HA state graph for the 30 minute utility meter, it will be smoothed and not represent the data very well.  Click the _Show more_ link to see it resetting to zero every 30 minutes.
* I use a package folder for my configuration - you may have to implement differently to work with how your configuration.yaml is set up.
* If you wish to have a utility meter with a different duration (e.g. hourly or weekly), it's simple enough to add another using one of the existing meters as an example.
* Number Inputs are used rather than sensors because upon a HA restart, sensors will create a new value at the last recorded value. In the case of Import Cost and Export Profit, the daily charts sum the values and this would result in a duplication of values in the 30 minute period, falsely inflating the daily sum.
