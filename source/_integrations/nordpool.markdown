---
title: Nord Pool
description: Instructions on how to integrate with the Nord Pool Energy market prices.
ha_category:
  - Energy
  - Finance
  - Sensor
ha_release: 2024.12
ha_iot_class: Cloud Polling
ha_config_flow: true
ha_codeowners:
  - '@gjohansson-ST'
ha_domain: nordpool
ha_platforms:
  - diagnostics
  - sensor
ha_integration_type: hub
---

The **Nord Pool** {% term integration %} integrates [Nord Pool Group](https://www.nordpoolgroup.com/) energy prices into Home Assistant.

The {% term integration %} provides the public market prices displayed on the [Nord Pool Auction page](https://data.nordpoolgroup.com/auction/day-ahead/prices).

{% include integrations/config_flow.md %}

{% tip %}
Only a single configuration entry is supported. Use the reconfigure option from the configuration entry if needed to modify the settings.

EUR is the base currency for market prices. If you choose another currency, you can find the conversion rate in the `Exchange rate` sensor.
All prices are displayed as `selected_currency/kWh`.
{% endtip %}

{% configuration_basic %}
Areas:
  description: Select one or multiple market areas to create sensors for.
Currency:
  description: Currency to display prices in. EUR is the base currency in Nord Pool prices.
{% endconfiguration_basic %}

## Sensors

The integration will create entities showing today's energy prices for the configured market area. Only the base energy price is shown. VAT and other additional costs are not included. 

### Main sensors

| Sensor                    | Type              | Description                                                                       |
| ------------------------- | ----------------- | --------------------------------------------------------------------------------- |
| Current price             | [Currency]/kWh    | The current (hourly) energy price.                                                |
| Previous price            | [Currency]/kWh    | The price of the previous hour.                                                   |
| Next price                | [Currency]/kWh    | The price of the next hour.                                                       |
| Daily average             | [Currency]/kWh    | The average of today's energy prices.                                             |

### Peak & off-peak sensors

Additional sensors are provided for peak and off-peak blocks.

- Peak refers to the price of the period from 8am to 8pm.
- Off-peak 1 refers to the price of the time period from midnight to 8am.
- Off-peak 2 refers to the average price of the time period from 8pm to midnight.

<p class='img'>
  <img src='/images/integrations/nordpool/nordpool-blocks.png' alt='Time blocks'>
</p>

| Sensor                          | Type              | Description                                                                       |
| ------------------------------- | ----------------- | --------------------------------------------------------------------------------- |
| [peak/off-peak] highest price   | [Currency]/kWh    | The hightest hourly price during the given timeframe.                             |
| [peak/off-peak] lowest  price   | [Currency]/kWh    | The lowest hourly price during the given timeframe.                               |
| [peak/off-peak] average         | [Currency]/kWh    | The average price of the given timeframe.                                         |
| [peak/off-peak] time from       | Datetime          | The start date/time of the given timeframe.                                       |
| [peak/off-peak] time until      | Datetime          | The end date/time of the given timeframe.                                         |

The block price sensors are not enabled by default.

### Diagnostic sensors

| Sensor                    | Type              | Description                                                                       |
| ------------------------- | ----------------- | --------------------------------------------------------------------------------- |
| Currency                  | [Currency]        | The configured currency.                                                          |
| Exchange rate             | Integer           | The exchange rate between the configure currency and Euro's.                      |
| Last updated              | Datetime          | The time when the market prices were last updated.                                |

## Examples

A template sensor to add VAT and fixed cost is useful to get the actual energy cost in the energy dashboard. 

### UI Template

Create a helper using the UI.
1. Go to {% my integrations title="**Settings** > **Devices & Services**" %} and at the top, choose the **Helpers** tab.
2. In the bottom right corner, select **Create helper**.
3. Select **Template** and **Template a sensor**.
4. Enter the fields as shown below.

The template below takes the current price attributes, adds 0.1293 EUR as fixed costs and adds 21% VAT.

<p class='img'>
  <img src='/images/integrations/nordpool/nordpool_create_template.png' alt='Screenshot: Create template sensor'>
</p>

### YAML Template

A template sensor to add VAT and a fixed cost from an helper entity `input_number.add_fixed_cost`.

{% raw %}

```yaml
template:
  - sensor:
      - name: "Nordpool"
        unit_of_measurement: "EUR/kWh"
        state_class: measurement
        state: >
          # create a variable with the current price
          {% set cost = states('sensor.nord_pool_nl_current_price') | float(0) %}
          # create a variable with the additional fixed cost
          {% set add_cost = states('input_number.add_fixed_cost') | float(0) %}
          # Add cost and additional fixed cost. Add VAT (25%) by multiplying with 1.25 and round to 2 digits: 
          {{ ((cost + add_cost) * 1.25) | round(2, default=0) }}
```

{% endraw %}

### Energy Dashboard

To use the Nordpool integration in the **Energy** dashboard, when configuring grid consumption and production, use the **Use an entity with current price** option.

<p class='img'>
  <img src='/images/integrations/nordpool/nordpool_energy_dashboard.png' alt='Screenshot: Create template sensor'>
</p>
