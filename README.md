# vallox_websocket_api 2.0
New async version of WebSocket API for Vallox ventilation units

Old sync version is available [1.5.x](https://pypi.org/project/vallox-websocket-api/1.5.2/)

## Releases

Release notes for the new versions are [here](https://github.com/yozik04/vallox_websocket_api/releases)

## Requirements
Python 3.6.0+

If you want to use Python 2.7, then use old sync version [1.5.x](https://pypi.org/project/vallox-websocket-api/1.5.2/)

Current code can:
* Set unit profile and fan speeds
* Fetch and decode metrics
* Fetch temperature logs

[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/yozik04/vallox_websocket_api/Test)](https://github.com/yozik04/vallox_websocket_api/actions/workflows/python-test.yml)

Your ventilation unit should be connected to LAN.

Code will use websockets API that Valox's own web interface uses.

## Supported units

Tested only with **Vallox 145 MV** and **Vallox 270 MV**.

But API should also work with next units:

    "Vallox 096 MV",
    "Vallox 110 MV",
    "Vallox 145 MV",
    "Vallox 245 MV",
    "ValloPlus 270 MV",
    "ValloPlus 350 MV",
    "ValloPlus 510 MV",
    "ValloPlus 850 MV",
    "Vallox TSK Multi 50 MV",
    "Vallox TSK Multi 80 MV",
    "ValloMulti 200 MV",
    "ValloMulti 300 MV",
    "DV96 Adroit",
    "DV110 Adroit",
    "DV145 Adroit",
    "DV245 Adroit",
    "DV TSK Multi 50 Adroit",
    "DV TSK Multi 80 Adroit"


# Dependencies

Code uses next libraries

* construct
* websockets

# Installation

    pip install vallox-websocket-api
    
    # or

    pip install git+https://github.com/yozik04/vallox_websocket_api
    
    # for old version for Python 2.7
    pip install 'vallox-websocket-api>=1.5.0,<2.0.0'

# Usage

Changing unit profile and reading metrics

## Profile control:

```python
import asyncio
from vallox_websocket_api import Vallox, PROFILE
client = Vallox('192.168.1.10') # Vallox unit IP

async def run():
    await client.get_profile() # RETURNS a PROFILE.* value
    await client.set_profile(PROFILE.HOME) # Permanently HOME profile
    await client.set_profile(PROFILE.AWAY) # Permanently AWAY profile
    await client.set_profile(PROFILE.FIREPLACE) # FIREPLACE mode for configured timeout
    await client.set_profile(PROFILE.FIREPLACE, 120) # FIREPLACE mode for 120 min
    await client.set_profile(PROFILE.FIREPLACE, 65535) # FIREPLACE mode, never TIMEOUT
    await client.set_profile(PROFILE.EXTRA) # EXTRA mode for configured timeout
    await client.set_profile(PROFILE.EXTRA, 120) # EXTRA mode for 120 min
    await client.set_profile(PROFILE.EXTRA, 65535) # EXTRA mode, never TIMEOUT

asyncio.get_event_loop().run_until_complete(run())
```

## Metrics

Reading metrics and setting values

## Reading Metrics
```python
import asyncio
from vallox_websocket_api import Client

client = Client('192.168.1.2')

async def run():
    metrics = await client.fetch_metrics()
    
    from pprint import pprint
    pprint(metrics)
    
asyncio.get_event_loop().run_until_complete(run())
```

Or if you want just a subset of metrics:

```python
import asyncio
from vallox_websocket_api import Client

client = Client('192.168.1.2')
async def run():
    metrics = await client.fetch_metrics([
      'A_CYC_TEMP_EXHAUST_AIR',
      'A_CYC_TEMP_EXTRACT_AIR',
      'A_CYC_TEMP_OUTDOOR_AIR',
      'A_CYC_TEMP_SUPPLY_AIR',
      'A_CYC_TEMP_SUPPLY_CELL_AIR'
    ])
    
    from pprint import pprint
    pprint(metrics)
asyncio.get_event_loop().run_until_complete(run())
```

## Reading historical data

Device stores some historical data in its memory. Code to retrieve it:

```python
import asyncio
from vallox_websocket_api import Client

client = Client('192.168.1.2')
async def run():
    data = await client.fetch_raw_logs()
    
    from pprint import pprint
    pprint(data)
asyncio.get_event_loop().run_until_complete(run())
```

## Setting values

Textual values will be converted to integers before sending to the unit.
```python
import asyncio
from vallox_websocket_api import Client

client = Client('192.168.9.106')

async def run():
    # Setting Home profile fan speed
    await client.set_values({'A_CYC_HOME_SPEED_SETTING': 10})
    
    # Setting Home profile target temperature
    await client.set_values({'A_CYC_HOME_AIR_TEMP_TARGET': 15})
    
    
    # Setting Away profile fan speed
    await client.set_values({'A_CYC_AWAY_SPEED_SETTING': 10})
    
    # Setting Away profile target temperature
    await client.set_values({'A_CYC_AWAY_AIR_TEMP_TARGET': 15})
    
    
    # Setting Boost profile fan speed
    await client.set_values({'A_CYC_BOOST_SPEED_SETTING': 10})
    
    # Setting Boost profile target temperature
    await client.set_values({'A_CYC_BOOST_AIR_TEMP_TARGET': 15})
    
    # Setting Boost profile timer
    await client.set_values({
      'A_CYC_BOOST_TIMER': 30, #Minutes
    })
    
    # Setting Fireplace profile fan speeds
    await client.set_values({
      'A_CYC_FIREPLACE_EXTR_FAN': 50,
      'A_CYC_FIREPLACE_SUPP_FAN': 50
    })
    
    # Setting Fireplace profile timer
    await client.set_values({
      'A_CYC_FIREPLACE_TIMER': 15, #Minutes
    })
    
asyncio.get_event_loop().run_until_complete(run())
```

Not all addresses are settable.

**If you think that a currently unsettable address needs to become settable, please create a new issue on the github.**

As a temporary solution you can make any address settable:
```python
client.set_settable_address('A_CYC_REMAINING_TIME_FOR_FILTER', int)
```

**int** in the end is accepted value type


## Details

### Available Metrics

Basically all that you can get via Modbus connection
```
'A_CYC_12_HOUR_CLOCK_ENABLED': 0,
 'A_CYC_ACCESS_LEVEL': 0,
 'A_CYC_ANALOG_CTRL_INPUT': 0,
 'A_CYC_ANALOG_INPUT_MODE': 0,
 'A_CYC_ANALOG_SENSOR_INPUT': 68,
 'A_CYC_APPL_SW_SIZE_0': 0,
 'A_CYC_APPL_SW_SIZE_1': 0,
 'A_CYC_APPL_SW_VERSION': 0,
 'A_CYC_APPL_SW_VERSION_1': 0,
 'A_CYC_APPL_SW_VERSION_2': 0,
 'A_CYC_APPL_SW_VERSION_3': 0,
 'A_CYC_APPL_SW_VERSION_4': 0,
 'A_CYC_APPL_SW_VERSION_5': 0,
 'A_CYC_APPL_SW_VERSION_6': 0,
 'A_CYC_APPL_SW_VERSION_7': 512,
 'A_CYC_APPL_SW_VERSION_8': 0,
 'A_CYC_APPL_SW_VERSION_9': 512,
 'A_CYC_AWAY_AIR_TEMP_TARGET': 14.9,
 'A_CYC_AWAY_CO2_CTRL_ENABLED': 0,
 'A_CYC_AWAY_RH_CTRL_ENABLED': 0,
 'A_CYC_AWAY_SPEED_SETTING': 25,
 'A_CYC_BETA_STATE': 0,
 'A_CYC_BG_LIGHT_LEVEL': 50,
 'A_CYC_BOOST_AIR_TEMP_TARGET': 14.9,
 'A_CYC_BOOST_CO2_CTRL_ENABLED': 0,
 'A_CYC_BOOST_RH_CTRL_ENABLED': 0,
 'A_CYC_BOOST_SPEED_SETTING': 65,
 'A_CYC_BOOST_TIME': 30,
 'A_CYC_BOOST_TIMER': 0,
 'A_CYC_BOOST_TIMER_ENABLED': 1,
 'A_CYC_BOOT_SW_VERSION': 0,
 'A_CYC_BRANDING': 0,
 'A_CYC_BYPASS_LOCKED': 0,
 'A_CYC_BY_PASS_TEST': 0,
 'A_CYC_CELL_STATE': 0,
 'A_CYC_CELL_TYPE': 0,
 'A_CYC_CLOUD_STATUS': 0,
 'A_CYC_CO2_0_ADDRESS': 0,
 'A_CYC_CO2_1_ADDRESS': 0,
 'A_CYC_CO2_2_ADDRESS': 0,
 'A_CYC_CO2_3_ADDRESS': 0,
 'A_CYC_CO2_4_ADDRESS': 0,
 'A_CYC_CO2_5_ADDRESS': 0,
 'A_CYC_CO2_LEVEL': 0,
 'A_CYC_CO2_SENSOR_0': 65535,
 'A_CYC_CO2_SENSOR_1': 65535,
 'A_CYC_CO2_SENSOR_2': 65535,
 'A_CYC_CO2_SENSOR_3': 65535,
 'A_CYC_CO2_SENSOR_4': 65535,
 'A_CYC_CO2_SENSOR_5': 65535,
 'A_CYC_CO2_THRESHOLD': 900,
 'A_CYC_CO2_VALUE': 0,
 'A_CYC_COMMAND': 0,
 'A_CYC_CONFIGURATION_CHECKSUM': 86,
 'A_CYC_CONFIGURATION_LSW': 817,
 'A_CYC_CONFIGURATION_MSW': 47440,
 'A_CYC_CONFIGURATION_NA_0': 0,
 'A_CYC_CONFIGURATION_NA_1': 0,
 'A_CYC_CONFIGURATION_NA_2': 0,
 'A_CYC_CONFIGURATION_NA_3': 0,
 'A_CYC_CURRENT_UP_TIME_HOURS': 35,
 'A_CYC_DAY': 26,
 'A_CYC_DEFROSTING': 0,
 'A_CYC_DEFROST_COMP_LIMIT': 630,
 'A_CYC_DEFROST_COUNT_IN_24H': 0,
 'A_CYC_DEFROST_COUNT_IN_WEEK': 0,
 'A_CYC_DEFROST_EXH_OFFSET': 11,
 'A_CYC_DEFROST_MODE': 0,
 'A_CYC_DEFROST_RH_OFFSET': 20,
 'A_CYC_DEFROST_RH_PARAM': 55,
 'A_CYC_DEFROST_SUPERMELT_THRESHOLD': 30,
 'A_CYC_DEFROST_TEMP_PARAM': -273.1,
 'A_CYC_DEWPOINT_LIMIT_IN_USE': 0,
 'A_CYC_DIGITAL_INPUT': 0,
 'A_CYC_DIGITAL_INPUT_1_MODE': 0,
 'A_CYC_DIGITAL_INPUT_2_MODE': 0,
 'A_CYC_DIP_SWITCH_0': 0,
 'A_CYC_DIP_SWITCH_1': 0,
 'A_CYC_DIP_SWITCH_2': 0,
 'A_CYC_DIP_SWITCH_3': 0,
 'A_CYC_EFFICIENCY_TEST': 0,
 'A_CYC_ENABLED': 1,
 'A_CYC_ETH_CLOUD_ENABLED': 0,
 'A_CYC_EXTRACT_EFFICIENCY': 82,
 'A_CYC_EXTRA_AIR_TEMP_TARGET': 14.9,
 'A_CYC_EXTRA_ENABLED': 0,
 'A_CYC_EXTRA_EXTR_FAN': 50,
 'A_CYC_EXTRA_HEATER_TEST': 0,
 'A_CYC_EXTRA_HEATER_TYPE': 1,
 'A_CYC_EXTRA_SUPP_FAN': 50,
 'A_CYC_EXTRA_TIME': 10,
 'A_CYC_EXTRA_TIMER': 0,
 'A_CYC_EXTRA_TIMER_ENABLED': 1,
 'A_CYC_EXTR_FAN_BALANCE_BASE': 30,
 'A_CYC_EXTR_FAN_SPEED': 1498,
 'A_CYC_EXTR_FAN_TEST': 0,
 'A_CYC_FAN_SPEED': 50,
 'A_CYC_FAULT_ACTIVITY': 0,
 'A_CYC_FAULT_ACTIVITY_10': 0,
 'A_CYC_FAULT_ACTIVITY_11': 0,
 'A_CYC_FAULT_ACTIVITY_12': 0,
 'A_CYC_FAULT_ACTIVITY_13': 0,
 'A_CYC_FAULT_ACTIVITY_14': 0,
 'A_CYC_FAULT_ACTIVITY_15': 0,
 'A_CYC_FAULT_ACTIVITY_16': 0,
 'A_CYC_FAULT_ACTIVITY_17': 0,
 'A_CYC_FAULT_ACTIVITY_18': 0,
 'A_CYC_FAULT_ACTIVITY_19': 0,
 'A_CYC_FAULT_ACTIVITY_2': 0,
 'A_CYC_FAULT_ACTIVITY_20': 0,
 'A_CYC_FAULT_ACTIVITY_21': 0,
 'A_CYC_FAULT_ACTIVITY_22': 0,
 'A_CYC_FAULT_ACTIVITY_23': 0,
 'A_CYC_FAULT_ACTIVITY_24': 0,
 'A_CYC_FAULT_ACTIVITY_25': 0,
 'A_CYC_FAULT_ACTIVITY_26': 0,
 'A_CYC_FAULT_ACTIVITY_27': 0,
 'A_CYC_FAULT_ACTIVITY_28': 0,
 'A_CYC_FAULT_ACTIVITY_29': 0,
 'A_CYC_FAULT_ACTIVITY_3': 0,
 'A_CYC_FAULT_ACTIVITY_30': 0,
 'A_CYC_FAULT_ACTIVITY_31': 0,
 'A_CYC_FAULT_ACTIVITY_32': 0,
 'A_CYC_FAULT_ACTIVITY_33': 0,
 'A_CYC_FAULT_ACTIVITY_4': 0,
 'A_CYC_FAULT_ACTIVITY_5': 0,
 'A_CYC_FAULT_ACTIVITY_6': 0,
 'A_CYC_FAULT_ACTIVITY_7': 0,
 'A_CYC_FAULT_ACTIVITY_8': 0,
 'A_CYC_FAULT_ACTIVITY_9': 0,
 'A_CYC_FAULT_CODE': 0,
 'A_CYC_FAULT_CODE_10': 0,
 'A_CYC_FAULT_CODE_11': 0,
 'A_CYC_FAULT_CODE_12': 0,
 'A_CYC_FAULT_CODE_13': 0,
 'A_CYC_FAULT_CODE_14': 0,
 'A_CYC_FAULT_CODE_15': 0,
 'A_CYC_FAULT_CODE_16': 0,
 'A_CYC_FAULT_CODE_17': 0,
 'A_CYC_FAULT_CODE_18': 0,
 'A_CYC_FAULT_CODE_19': 0,
 'A_CYC_FAULT_CODE_2': 0,
 'A_CYC_FAULT_CODE_20': 0,
 'A_CYC_FAULT_CODE_21': 0,
 'A_CYC_FAULT_CODE_22': 0,
 'A_CYC_FAULT_CODE_23': 0,
 'A_CYC_FAULT_CODE_24': 0,
 'A_CYC_FAULT_CODE_25': 0,
 'A_CYC_FAULT_CODE_26': 0,
 'A_CYC_FAULT_CODE_27': 0,
 'A_CYC_FAULT_CODE_28': 0,
 'A_CYC_FAULT_CODE_29': 0,
 'A_CYC_FAULT_CODE_3': 0,
 'A_CYC_FAULT_CODE_30': 0,
 'A_CYC_FAULT_CODE_31': 0,
 'A_CYC_FAULT_CODE_32': 0,
 'A_CYC_FAULT_CODE_33': 0,
 'A_CYC_FAULT_CODE_4': 0,
 'A_CYC_FAULT_CODE_5': 0,
 'A_CYC_FAULT_CODE_6': 0,
 'A_CYC_FAULT_CODE_7': 0,
 'A_CYC_FAULT_CODE_8': 0,
 'A_CYC_FAULT_CODE_9': 0,
 'A_CYC_FAULT_COUNT': 0,
 'A_CYC_FAULT_COUNT_10': 0,
 'A_CYC_FAULT_COUNT_11': 0,
 'A_CYC_FAULT_COUNT_12': 0,
 'A_CYC_FAULT_COUNT_13': 0,
 'A_CYC_FAULT_COUNT_14': 0,
 'A_CYC_FAULT_COUNT_15': 0,
 'A_CYC_FAULT_COUNT_16': 0,
 'A_CYC_FAULT_COUNT_17': 0,
 'A_CYC_FAULT_COUNT_18': 0,
 'A_CYC_FAULT_COUNT_19': 0,
 'A_CYC_FAULT_COUNT_2': 0,
 'A_CYC_FAULT_COUNT_20': 0,
 'A_CYC_FAULT_COUNT_21': 0,
 'A_CYC_FAULT_COUNT_22': 0,
 'A_CYC_FAULT_COUNT_23': 0,
 'A_CYC_FAULT_COUNT_24': 0,
 'A_CYC_FAULT_COUNT_25': 0,
 'A_CYC_FAULT_COUNT_26': 0,
 'A_CYC_FAULT_COUNT_27': 0,
 'A_CYC_FAULT_COUNT_28': 0,
 'A_CYC_FAULT_COUNT_29': 0,
 'A_CYC_FAULT_COUNT_3': 0,
 'A_CYC_FAULT_COUNT_30': 0,
 'A_CYC_FAULT_COUNT_31': 0,
 'A_CYC_FAULT_COUNT_32': 0,
 'A_CYC_FAULT_COUNT_33': 0,
 'A_CYC_FAULT_COUNT_4': 0,
 'A_CYC_FAULT_COUNT_5': 0,
 'A_CYC_FAULT_COUNT_6': 0,
 'A_CYC_FAULT_COUNT_7': 0,
 'A_CYC_FAULT_COUNT_8': 0,
 'A_CYC_FAULT_COUNT_9': 0,
 'A_CYC_FAULT_FIRST_DATE': 0,
 'A_CYC_FAULT_FIRST_DATE_10': 0,
 'A_CYC_FAULT_FIRST_DATE_11': 0,
 'A_CYC_FAULT_FIRST_DATE_12': 0,
 'A_CYC_FAULT_FIRST_DATE_13': 0,
 'A_CYC_FAULT_FIRST_DATE_14': 0,
 'A_CYC_FAULT_FIRST_DATE_15': 0,
 'A_CYC_FAULT_FIRST_DATE_16': 0,
 'A_CYC_FAULT_FIRST_DATE_17': 0,
 'A_CYC_FAULT_FIRST_DATE_18': 0,
 'A_CYC_FAULT_FIRST_DATE_19': 0,
 'A_CYC_FAULT_FIRST_DATE_2': 0,
 'A_CYC_FAULT_FIRST_DATE_20': 0,
 'A_CYC_FAULT_FIRST_DATE_21': 0,
 'A_CYC_FAULT_FIRST_DATE_22': 0,
 'A_CYC_FAULT_FIRST_DATE_23': 0,
 'A_CYC_FAULT_FIRST_DATE_24': 0,
 'A_CYC_FAULT_FIRST_DATE_25': 0,
 'A_CYC_FAULT_FIRST_DATE_26': 0,
 'A_CYC_FAULT_FIRST_DATE_27': 0,
 'A_CYC_FAULT_FIRST_DATE_28': 0,
 'A_CYC_FAULT_FIRST_DATE_29': 0,
 'A_CYC_FAULT_FIRST_DATE_3': 0,
 'A_CYC_FAULT_FIRST_DATE_30': 0,
 'A_CYC_FAULT_FIRST_DATE_31': 0,
 'A_CYC_FAULT_FIRST_DATE_32': 0,
 'A_CYC_FAULT_FIRST_DATE_33': 0,
 'A_CYC_FAULT_FIRST_DATE_4': 0,
 'A_CYC_FAULT_FIRST_DATE_5': 0,
 'A_CYC_FAULT_FIRST_DATE_6': 0,
 'A_CYC_FAULT_FIRST_DATE_7': 0,
 'A_CYC_FAULT_FIRST_DATE_8': 0,
 'A_CYC_FAULT_FIRST_DATE_9': 0,
 'A_CYC_FAULT_LAST_DATE': 0,
 'A_CYC_FAULT_LAST_DATE_10': 0,
 'A_CYC_FAULT_LAST_DATE_11': 0,
 'A_CYC_FAULT_LAST_DATE_12': 0,
 'A_CYC_FAULT_LAST_DATE_13': 0,
 'A_CYC_FAULT_LAST_DATE_14': 0,
 'A_CYC_FAULT_LAST_DATE_15': 0,
 'A_CYC_FAULT_LAST_DATE_16': 0,
 'A_CYC_FAULT_LAST_DATE_17': 0,
 'A_CYC_FAULT_LAST_DATE_18': 0,
 'A_CYC_FAULT_LAST_DATE_19': 0,
 'A_CYC_FAULT_LAST_DATE_2': 0,
 'A_CYC_FAULT_LAST_DATE_20': 0,
 'A_CYC_FAULT_LAST_DATE_21': 0,
 'A_CYC_FAULT_LAST_DATE_22': 0,
 'A_CYC_FAULT_LAST_DATE_23': 0,
 'A_CYC_FAULT_LAST_DATE_24': 0,
 'A_CYC_FAULT_LAST_DATE_25': 0,
 'A_CYC_FAULT_LAST_DATE_26': 0,
 'A_CYC_FAULT_LAST_DATE_27': 0,
 'A_CYC_FAULT_LAST_DATE_28': 0,
 'A_CYC_FAULT_LAST_DATE_29': 0,
 'A_CYC_FAULT_LAST_DATE_3': 0,
 'A_CYC_FAULT_LAST_DATE_30': 0,
 'A_CYC_FAULT_LAST_DATE_31': 0,
 'A_CYC_FAULT_LAST_DATE_32': 0,
 'A_CYC_FAULT_LAST_DATE_33': 0,
 'A_CYC_FAULT_LAST_DATE_4': 0,
 'A_CYC_FAULT_LAST_DATE_5': 0,
 'A_CYC_FAULT_LAST_DATE_6': 0,
 'A_CYC_FAULT_LAST_DATE_7': 0,
 'A_CYC_FAULT_LAST_DATE_8': 0,
 'A_CYC_FAULT_LAST_DATE_9': 0,
 'A_CYC_FAULT_SEVERITY': 0,
 'A_CYC_FAULT_SEVERITY_10': 0,
 'A_CYC_FAULT_SEVERITY_11': 0,
 'A_CYC_FAULT_SEVERITY_12': 0,
 'A_CYC_FAULT_SEVERITY_13': 0,
 'A_CYC_FAULT_SEVERITY_14': 0,
 'A_CYC_FAULT_SEVERITY_15': 0,
 'A_CYC_FAULT_SEVERITY_16': 0,
 'A_CYC_FAULT_SEVERITY_17': 0,
 'A_CYC_FAULT_SEVERITY_18': 0,
 'A_CYC_FAULT_SEVERITY_19': 0,
 'A_CYC_FAULT_SEVERITY_2': 0,
 'A_CYC_FAULT_SEVERITY_20': 0,
 'A_CYC_FAULT_SEVERITY_21': 0,
 'A_CYC_FAULT_SEVERITY_22': 0,
 'A_CYC_FAULT_SEVERITY_23': 0,
 'A_CYC_FAULT_SEVERITY_24': 0,
 'A_CYC_FAULT_SEVERITY_25': 0,
 'A_CYC_FAULT_SEVERITY_26': 0,
 'A_CYC_FAULT_SEVERITY_27': 0,
 'A_CYC_FAULT_SEVERITY_28': 0,
 'A_CYC_FAULT_SEVERITY_29': 0,
 'A_CYC_FAULT_SEVERITY_3': 0,
 'A_CYC_FAULT_SEVERITY_30': 0,
 'A_CYC_FAULT_SEVERITY_31': 0,
 'A_CYC_FAULT_SEVERITY_32': 0,
 'A_CYC_FAULT_SEVERITY_33': 0,
 'A_CYC_FAULT_SEVERITY_4': 0,
 'A_CYC_FAULT_SEVERITY_5': 0,
 'A_CYC_FAULT_SEVERITY_6': 0,
 'A_CYC_FAULT_SEVERITY_7': 0,
 'A_CYC_FAULT_SEVERITY_8': 0,
 'A_CYC_FAULT_SEVERITY_9': 0,
 'A_CYC_FILTER_CHANGED_DAY': 23,
 'A_CYC_FILTER_CHANGED_MONTH': 3,
 'A_CYC_FILTER_CHANGED_YEAR': 17,
 'A_CYC_FILTER_CHANGE_INTERVAL': 120,
 'A_CYC_FIREPLACE_EXTR_FAN': 50,
 'A_CYC_FIREPLACE_SUPP_FAN': 50,
 'A_CYC_FIREPLACE_SWITCH': 0,
 'A_CYC_FIREPLACE_TIME': 15,
 'A_CYC_FIREPLACE_TIMER': 0,
 'A_CYC_FIREPLACE_TIMER_ENABLED': 1,
 'A_CYC_GW_ADDRESS_1': 49320,
 'A_CYC_GW_ADDRESS_2': 2305,
 'A_CYC_HEATER_TEST': 0,
 'A_CYC_HOME_AIR_TEMP_TARGET': 14.9,
 'A_CYC_HOME_CO2_CTRL_ENABLED': 0,
 'A_CYC_HOME_RH_CTRL_ENABLED': 1,
 'A_CYC_HOME_SPEED_SETTING': 50,
 'A_CYC_HOUR': 23,
 'A_CYC_INSTALLATION_DONE': 0,
 'A_CYC_IN_BYPASS': 0,
 'A_CYC_IN_ERROR': 1,
 'A_CYC_IN_EXTRACT_FAN': 50,
 'A_CYC_IN_EXTRA_HEATER': 0,
 'A_CYC_IN_HEATER': 0,
 'A_CYC_IN_SUPPLY_FAN': 50,
 'A_CYC_IO_BYPASS': 0,
 'A_CYC_IO_ERROR': 1,
 'A_CYC_IO_EXTRACT_FAN': 148,
 'A_CYC_IO_EXTRA_HEATER': 0,
 'A_CYC_IO_HEATER': 1,
 'A_CYC_IO_SUPPLY_FAN': 148,
 'A_CYC_IP_ADDRESS_1': 44526,
 'A_CYC_IP_ADDRESS_2': 3186,
 'A_CYC_LANGUAGE': 0,
 'A_CYC_LIMP_MODE': 0,
 'A_CYC_MACHINE_MODEL': 3,
 'A_CYC_MACHINE_TYPE': 3,
 'A_CYC_MASK_ADDRESS_1': 65535,
 'A_CYC_MASK_ADDRESS_2': 65280,
 'A_CYC_MASTER_PASSWORD': 1222,
 'A_CYC_METRICS': 0,
 'A_CYC_MINUTE': 43,
 'A_CYC_MIN_DEFROST_TIME': 3,
 'A_CYC_MLV_AUTO_MANUAL': 0,
 'A_CYC_MLV_MODES': 0,
 'A_CYC_MLV_STATE': 0,
 'A_CYC_MLV_SUMMER_HYSTERESIS': 1,
 'A_CYC_MLV_SUMMER_SETPOINT': 28815,
 'A_CYC_MLV_SUPPLY_LOWER_LIMIT': 29115,
 'A_CYC_MLV_WINTER_HYSTERESIS': 5,
 'A_CYC_MLV_WINTER_SETPOINT': 26815,
 'A_CYC_MODBUS_ADDRESS': 1,
 'A_CYC_MODBUS_BAUD_X100': 192,
 'A_CYC_MODBUS_FRAME': 257,
 'A_CYC_MODE': 0,
 'A_CYC_MONTH': 3,
 'A_CYC_MULTISENSOR_CO2': 0,
 'A_CYC_MULTISENSOR_RH': 46,
 'A_CYC_MULTISENSOR_TEMP': 29550,
 'A_CYC_OPT_TEMP_SENSOR_MODE': -273.1,
 'A_CYC_PARENTAL_CTRL_ENABLED': 0,
 'A_CYC_PARENTAL_PASSWORD': 2345,
 'A_CYC_PARTIAL_BYPASS': 0,
 'A_CYC_POST_HEATER_TYPE': 1,
 'A_CYC_POST_HEATER_WINTER_SETPOINT': 29215,
 'A_CYC_RELAY_MODE': 0,
 'A_CYC_REMAINING_TIME_FOR_FILTER': 117,
 'A_CYC_RH_0_ADDRESS': 0,
 'A_CYC_RH_1_ADDRESS': 0,
 'A_CYC_RH_2_ADDRESS': 0,
 'A_CYC_RH_3_ADDRESS': 0,
 'A_CYC_RH_4_ADDRESS': 0,
 'A_CYC_RH_5_ADDRESS': 0,
 'A_CYC_RH_BASIC_LEVEL': 69,
 'A_CYC_RH_LEVEL': 1,
 'A_CYC_RH_LEVEL_MODE': 0,
 'A_CYC_RH_SENSOR_0': 65535,
 'A_CYC_RH_SENSOR_1': 65535,
 'A_CYC_RH_SENSOR_2': 65535,
 'A_CYC_RH_SENSOR_3': 65535,
 'A_CYC_RH_SENSOR_4': 65535,
 'A_CYC_RH_SENSOR_5': 65535,
 'A_CYC_RH_VALUE': 68,
 'A_CYC_SCHEDULE_FRIDAY_00': 0,
 'A_CYC_SCHEDULE_FRIDAY_01': 0,
 'A_CYC_SCHEDULE_FRIDAY_02': 0,
 'A_CYC_SCHEDULE_FRIDAY_03': 0,
 'A_CYC_SCHEDULE_FRIDAY_04': 0,
 'A_CYC_SCHEDULE_FRIDAY_05': 0,
 'A_CYC_SCHEDULE_FRIDAY_06': 0,
 'A_CYC_SCHEDULE_FRIDAY_07': 0,
 'A_CYC_SCHEDULE_FRIDAY_08': 0,
 'A_CYC_SCHEDULE_FRIDAY_09': 0,
 'A_CYC_SCHEDULE_FRIDAY_10': 0,
 'A_CYC_SCHEDULE_FRIDAY_11': 0,
 'A_CYC_SCHEDULE_FRIDAY_12': 0,
 'A_CYC_SCHEDULE_FRIDAY_13': 0,
 'A_CYC_SCHEDULE_FRIDAY_14': 0,
 'A_CYC_SCHEDULE_FRIDAY_15': 0,
 'A_CYC_SCHEDULE_FRIDAY_16': 0,
 'A_CYC_SCHEDULE_FRIDAY_17': 0,
 'A_CYC_SCHEDULE_FRIDAY_18': 0,
 'A_CYC_SCHEDULE_FRIDAY_19': 0,
 'A_CYC_SCHEDULE_FRIDAY_20': 0,
 'A_CYC_SCHEDULE_FRIDAY_21': 0,
 'A_CYC_SCHEDULE_FRIDAY_22': 0,
 'A_CYC_SCHEDULE_FRIDAY_23': 0,
 'A_CYC_SCHEDULE_MONDAY_00': 0,
 'A_CYC_SCHEDULE_MONDAY_01': 0,
 'A_CYC_SCHEDULE_MONDAY_02': 0,
 'A_CYC_SCHEDULE_MONDAY_03': 0,
 'A_CYC_SCHEDULE_MONDAY_04': 0,
 'A_CYC_SCHEDULE_MONDAY_05': 0,
 'A_CYC_SCHEDULE_MONDAY_06': 0,
 'A_CYC_SCHEDULE_MONDAY_07': 0,
 'A_CYC_SCHEDULE_MONDAY_08': 0,
 'A_CYC_SCHEDULE_MONDAY_09': 0,
 'A_CYC_SCHEDULE_MONDAY_10': 0,
 'A_CYC_SCHEDULE_MONDAY_11': 0,
 'A_CYC_SCHEDULE_MONDAY_12': 0,
 'A_CYC_SCHEDULE_MONDAY_13': 0,
 'A_CYC_SCHEDULE_MONDAY_14': 0,
 'A_CYC_SCHEDULE_MONDAY_15': 0,
 'A_CYC_SCHEDULE_MONDAY_16': 0,
 'A_CYC_SCHEDULE_MONDAY_17': 0,
 'A_CYC_SCHEDULE_MONDAY_18': 0,
 'A_CYC_SCHEDULE_MONDAY_19': 0,
 'A_CYC_SCHEDULE_MONDAY_20': 0,
 'A_CYC_SCHEDULE_MONDAY_21': 0,
 'A_CYC_SCHEDULE_MONDAY_22': 0,
 'A_CYC_SCHEDULE_MONDAY_23': 0,
 'A_CYC_SCHEDULE_SATURDAY_00': 0,
 'A_CYC_SCHEDULE_SATURDAY_01': 0,
 'A_CYC_SCHEDULE_SATURDAY_02': 0,
 'A_CYC_SCHEDULE_SATURDAY_03': 0,
 'A_CYC_SCHEDULE_SATURDAY_04': 0,
 'A_CYC_SCHEDULE_SATURDAY_05': 0,
 'A_CYC_SCHEDULE_SATURDAY_06': 0,
 'A_CYC_SCHEDULE_SATURDAY_07': 0,
 'A_CYC_SCHEDULE_SATURDAY_08': 0,
 'A_CYC_SCHEDULE_SATURDAY_09': 0,
 'A_CYC_SCHEDULE_SATURDAY_10': 0,
 'A_CYC_SCHEDULE_SATURDAY_11': 0,
 'A_CYC_SCHEDULE_SATURDAY_12': 0,
 'A_CYC_SCHEDULE_SATURDAY_13': 0,
 'A_CYC_SCHEDULE_SATURDAY_14': 0,
 'A_CYC_SCHEDULE_SATURDAY_15': 0,
 'A_CYC_SCHEDULE_SATURDAY_16': 0,
 'A_CYC_SCHEDULE_SATURDAY_17': 0,
 'A_CYC_SCHEDULE_SATURDAY_18': 0,
 'A_CYC_SCHEDULE_SATURDAY_19': 0,
 'A_CYC_SCHEDULE_SATURDAY_20': 0,
 'A_CYC_SCHEDULE_SATURDAY_21': 0,
 'A_CYC_SCHEDULE_SATURDAY_22': 0,
 'A_CYC_SCHEDULE_SATURDAY_23': 0,
 'A_CYC_SCHEDULE_SUNDAY_00': 0,
 'A_CYC_SCHEDULE_SUNDAY_01': 0,
 'A_CYC_SCHEDULE_SUNDAY_02': 0,
 'A_CYC_SCHEDULE_SUNDAY_03': 0,
 'A_CYC_SCHEDULE_SUNDAY_04': 0,
 'A_CYC_SCHEDULE_SUNDAY_05': 0,
 'A_CYC_SCHEDULE_SUNDAY_06': 0,
 'A_CYC_SCHEDULE_SUNDAY_07': 0,
 'A_CYC_SCHEDULE_SUNDAY_08': 0,
 'A_CYC_SCHEDULE_SUNDAY_09': 0,
 'A_CYC_SCHEDULE_SUNDAY_10': 0,
 'A_CYC_SCHEDULE_SUNDAY_11': 0,
 'A_CYC_SCHEDULE_SUNDAY_12': 0,
 'A_CYC_SCHEDULE_SUNDAY_13': 0,
 'A_CYC_SCHEDULE_SUNDAY_14': 0,
 'A_CYC_SCHEDULE_SUNDAY_15': 0,
 'A_CYC_SCHEDULE_SUNDAY_16': 0,
 'A_CYC_SCHEDULE_SUNDAY_17': 0,
 'A_CYC_SCHEDULE_SUNDAY_18': 0,
 'A_CYC_SCHEDULE_SUNDAY_19': 0,
 'A_CYC_SCHEDULE_SUNDAY_20': 0,
 'A_CYC_SCHEDULE_SUNDAY_21': 0,
 'A_CYC_SCHEDULE_SUNDAY_22': 0,
 'A_CYC_SCHEDULE_SUNDAY_23': 0,
 'A_CYC_SCHEDULE_THURSDAY_00': 0,
 'A_CYC_SCHEDULE_THURSDAY_01': 0,
 'A_CYC_SCHEDULE_THURSDAY_02': 0,
 'A_CYC_SCHEDULE_THURSDAY_03': 0,
 'A_CYC_SCHEDULE_THURSDAY_04': 0,
 'A_CYC_SCHEDULE_THURSDAY_05': 0,
 'A_CYC_SCHEDULE_THURSDAY_06': 0,
 'A_CYC_SCHEDULE_THURSDAY_07': 0,
 'A_CYC_SCHEDULE_THURSDAY_08': 0,
 'A_CYC_SCHEDULE_THURSDAY_09': 0,
 'A_CYC_SCHEDULE_THURSDAY_10': 0,
 'A_CYC_SCHEDULE_THURSDAY_11': 0,
 'A_CYC_SCHEDULE_THURSDAY_12': 0,
 'A_CYC_SCHEDULE_THURSDAY_13': 0,
 'A_CYC_SCHEDULE_THURSDAY_14': 0,
 'A_CYC_SCHEDULE_THURSDAY_15': 0,
 'A_CYC_SCHEDULE_THURSDAY_16': 0,
 'A_CYC_SCHEDULE_THURSDAY_17': 0,
 'A_CYC_SCHEDULE_THURSDAY_18': 0,
 'A_CYC_SCHEDULE_THURSDAY_19': 0,
 'A_CYC_SCHEDULE_THURSDAY_20': 0,
 'A_CYC_SCHEDULE_THURSDAY_21': 0,
 'A_CYC_SCHEDULE_THURSDAY_22': 0,
 'A_CYC_SCHEDULE_THURSDAY_23': 0,
 'A_CYC_SCHEDULE_TUESDAY_00': 0,
 'A_CYC_SCHEDULE_TUESDAY_01': 0,
 'A_CYC_SCHEDULE_TUESDAY_02': 0,
 'A_CYC_SCHEDULE_TUESDAY_03': 0,
 'A_CYC_SCHEDULE_TUESDAY_04': 0,
 'A_CYC_SCHEDULE_TUESDAY_05': 0,
 'A_CYC_SCHEDULE_TUESDAY_06': 0,
 'A_CYC_SCHEDULE_TUESDAY_07': 0,
 'A_CYC_SCHEDULE_TUESDAY_08': 0,
 'A_CYC_SCHEDULE_TUESDAY_09': 0,
 'A_CYC_SCHEDULE_TUESDAY_10': 0,
 'A_CYC_SCHEDULE_TUESDAY_11': 0,
 'A_CYC_SCHEDULE_TUESDAY_12': 0,
 'A_CYC_SCHEDULE_TUESDAY_13': 0,
 'A_CYC_SCHEDULE_TUESDAY_14': 0,
 'A_CYC_SCHEDULE_TUESDAY_15': 0,
 'A_CYC_SCHEDULE_TUESDAY_16': 0,
 'A_CYC_SCHEDULE_TUESDAY_17': 0,
 'A_CYC_SCHEDULE_TUESDAY_18': 0,
 'A_CYC_SCHEDULE_TUESDAY_19': 0,
 'A_CYC_SCHEDULE_TUESDAY_20': 0,
 'A_CYC_SCHEDULE_TUESDAY_21': 0,
 'A_CYC_SCHEDULE_TUESDAY_22': 0,
 'A_CYC_SCHEDULE_TUESDAY_23': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_00': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_01': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_02': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_03': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_04': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_05': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_06': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_07': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_08': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_09': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_10': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_11': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_12': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_13': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_14': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_15': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_16': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_17': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_18': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_19': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_20': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_21': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_22': 0,
 'A_CYC_SCHEDULE_WEDNESDAY_23': 0,
 'A_CYC_SERIAL_NUMBER_LSW': 98327,
 'A_CYC_SERIAL_NUMBER_MSW': 32794,
 'A_CYC_SIDEDNESS': 1,
 'A_CYC_SLEEP_DELAY': 10,
 'A_CYC_STATE': 0,
 'A_CYC_SUMMER_TIME_AUTO_ENAB': 1,
 'A_CYC_SUPPLY_EFFICIENCY': 59,
 'A_CYC_SUPPLY_HEATING_ADJUST_MODE': 0,
 'A_CYC_SUPP_FAN_BALANCE_BASE': 30,
 'A_CYC_SUPP_FAN_SPEED': 1500,
 'A_CYC_SUPP_FAN_TEST': 0,
 'A_CYC_TEMP_EXHAUST_AIR': 6.9,
 'A_CYC_TEMP_EXTRACT_AIR': 16.9,
 'A_CYC_TEMP_OPTIONAL': -49.1,
 'A_CYC_TEMP_OUTDOOR_AIR': 5.9,
 'A_CYC_TEMP_SUPPLY_AIR': 14.9,
 'A_CYC_TEMP_SUPPLY_CELL_AIR': 11.9,
 'A_CYC_TOTAL_FAULT_COUNT': 0,
 'A_CYC_TOTAL_UP_TIME_HOURS': 69,
 'A_CYC_TOTAL_UP_TIME_YEARS': 0,
 'A_CYC_TYP_APPL_SW_SIZE_0': 0,
 'A_CYC_TYP_APPL_SW_SIZE_1': 0,
 'A_CYC_TYP_APPL_SW_VERSION': 0,
 'A_CYC_TYP_APPL_SW_VERSION_1': 0,
 'A_CYC_TYP_APPL_SW_VERSION_2': 0,
 'A_CYC_TYP_APPL_SW_VERSION_3': 0,
 'A_CYC_TYP_APPL_SW_VERSION_4': 0,
 'A_CYC_TYP_APPL_SW_VERSION_5': 0,
 'A_CYC_TYP_APPL_SW_VERSION_6': 0,
 'A_CYC_TYP_APPL_SW_VERSION_7': 0,
 'A_CYC_TYP_APPL_SW_VERSION_8': 0,
 'A_CYC_TYP_APPL_SW_VERSION_9': 0,
 'A_CYC_TYP_BOOT_SW_VERSION': 0,
 'A_CYC_TYP_PANEL_MODEL': 0,
 'A_CYC_TYP_SERIAL_NUMBER_LSW': 0,
 'A_CYC_TYP_SERIAL_NUMBER_MSW': 0,
 'A_CYC_UPD_ADDRESS_1': 0,
 'A_CYC_UPD_ADDRESS_2': 0,
 'A_CYC_USED_SETTINGS_VARIABLES': 76,
 'A_CYC_USER_PASSWORD': 0,
 'A_CYC_UUID0': 25432,
 'A_CYC_UUID1': 25432,
 'A_CYC_UUID2': 25432,
 'A_CYC_UUID3': 25432,
 'A_CYC_UUID4': 25432,
 'A_CYC_UUID5': 25432,
 'A_CYC_UUID6': 25432,
 'A_CYC_UUID7': 25432,
 'A_CYC_VOC_LEVEL': 1000,
 'A_CYC_VOC_SENSOR_0': 65535,
 'A_CYC_VOC_SENSOR_1': 65535,
 'A_CYC_VOC_SENSOR_2': 65535,
 'A_CYC_VOC_SENSOR_3': 0,
 'A_CYC_VOLTAGE_LOW': 0,
 'A_CYC_WATERHEATER_STORED_I': 35340,
 'A_CYC_WEEKDAY': 7,
 'A_CYC_WEEKLY_TIMER_ENABLED': 0,
 'A_CYC_YEAR': 17,
 'C_COMMAND_CHECK_STORAGES': 0,
 'C_COMMAND_CLEARSCRATCHFLASH': 0,
 'C_COMMAND_CLEAR_ALL': 0,
 'C_COMMAND_CLEAR_AND_REBOOT': 0,
 'C_COMMAND_CREATE_NEW_UUID': 0,
 'C_COMMAND_DEBUGGING_GET': 0,
 'C_COMMAND_DEBUGGING_START': 0,
 'C_COMMAND_DUMPSCRATCHFLASH': 0,
 'C_COMMAND_FORCE_FACTORY_SETTINGS': 0,
 'C_COMMAND_FORCE_UPDATE': 0,
 'C_COMMAND_RESET': 0,
 'C_COMMAND_RESTART': 817,
 'C_COMMAND_RESTORE_FACTORY_SETTINGS': 0,
 'C_COMMAND_RESTORE_INSTALLATION_SETTINGS': 0,
 'C_COMMAND_RESTORE_USER_SETTINGS': 0,
 'C_COMMAND_REVERSE_NETDEBUG': 0,
 'C_COMMAND_SAVE_CLIENT_DATA': 0,
 'C_COMMAND_SAVE_FACTORY_SETTINGS': 0,
 'C_COMMAND_SAVE_INSTALLATION_SETTINGS': 0,
 'C_COMMAND_SAVE_USER_SETTINGS': 0,
 'C_COMMAND_START_APPLICATION': 47440,
 'C_COMMAND_START_CONTROL': 0,
 'C_COMMAND_STOP_CONTROL': 0,
 'C_COMMAND_TYPHOON_READY_FOR_UPDATE': 0,
 'C_COMMAND_TYPHOON_USB_TRANSFER': 0,
 'C_COMMAND_UPDATE_CHECK_CLOUD_VERSION': 0,
 'C_COMMAND_UPDATE_CLOUD_ALPHA': 0,
 'C_COMMAND_UPDATE_CLOUD_BETA': 86,
 'C_COMMAND_UPDATE_CLOUD_FORCE_MASTER': 0,
 'C_COMMAND_UPDATE_CLOUD_INSIDER': 0,
 'C_COMMAND_UPDATE_GET': 8255,
 'C_COMMAND_UPDATE_GET_ETH': 0,
 'C_COMMAND_UPDATE_GET_ETH_CLOUD': 0,
 'C_COMMAND_UPDATE_REQUEST': 3,
 'C_COMMAND_UPDATE_SEND': 3,
 'C_CYC_ACCESS_FREE': 0,
 'C_CYC_ACCESS_LIMITED': 0,
 'C_CYC_ACCESS_VERY_LIMITED': 0,
 'C_CYC_AFFIRMATIVE': 0,
 'C_CYC_ANA_IN_NONE': 0,
 'C_CYC_ANA_IN_POST_HEAT': 0,
 'C_CYC_ANA_IN_SITUATION': 0,
 'C_CYC_ANA_IN_SPEED': 0,
 'C_CYC_BYPASS_MODE': 0,
 'C_CYC_CELL_STATE_BYPASS': 0,
 'C_CYC_CELL_STATE_COOLRECO': 0,
 'C_CYC_CELL_STATE_DEFROST': 0,
 'C_CYC_CELL_STATE_HEATRECO': 0,
 'C_CYC_DIG_IN_BOOST': 0,
 'C_CYC_DIG_IN_BYPASS': 0,
 'C_CYC_DIG_IN_EXTRA': 256,
 'C_CYC_DIG_IN_FIREPLACE': 0,
 'C_CYC_DIG_IN_FIRE_ALARM': 0,
 'C_CYC_DIG_IN_FP_DIRECT': 0,
 'C_CYC_DIG_IN_HOME_STATE': 0,
 'C_CYC_DIG_IN_NONE': 0,
 'C_CYC_DIG_IN_SCHEDULER': 0,
 'C_CYC_ELECTRIC': 0,
 'C_CYC_EVENT_AWAY': 0,
 'C_CYC_EVENT_BOOST': 0,
 'C_CYC_EVENT_HOME': 0,
 'C_CYC_EVENT_NONE': 0,
 'C_CYC_FAILED': 0,
 'C_CYC_FAN_STOP_MODE': 0,
 'C_CYC_HEATING_COOLING': 0,
 'C_CYC_HEATING_EXTRACT': 0,
 'C_CYC_HEATING_SUPPLY': 0,
 'C_CYC_HRC_ALUMINIUM': 0,
 'C_CYC_HRC_ENTALPHY': 0,
 'C_CYC_HRC_PLASTIC': 0,
 'C_CYC_LEFT': 0,
 'C_CYC_MLV': 0,
 'C_CYC_MLV_OUTDOOR': 0,
 'C_CYC_MLV_SUPPLY': 0,
 'C_CYC_MODE_DEFROST': 0,
 'C_CYC_MODE_EMC_TEST': 0,
 'C_CYC_MODE_MANUAL': 0,
 'C_CYC_MODE_NORMAL': 0,
 'C_CYC_MODE_OFF': 0,
 'C_CYC_MODE_SELF_TEST': 0,
 'C_CYC_MODE_TESTING': 0,
 'C_CYC_NEGATIVE': 0,
 'C_CYC_NONE': 0,
 'C_CYC_NOT_TESTABLE': 0,
 'C_CYC_NOT_TESTED': 0,
 'C_CYC_OPT_NONE': 0,
 'C_CYC_OPT_TEMP_EXTRACT': -273.1,
 'C_CYC_OPT_TEMP_MLV': -273.1,
 'C_CYC_OPT_TEMP_NONE': -273.1,
 'C_CYC_RELAY_AIRHEATER': 0,
 'C_CYC_RELAY_BYPASS_STATE': 0,
 'C_CYC_RELAY_ERROR': 0,
 'C_CYC_RELAY_ERR_SERVICE': 0,
 'C_CYC_RELAY_FIRE_ALARM': 0,
 'C_CYC_RELAY_MLV': 0,
 'C_CYC_RELAY_NONE': 0,
 'C_CYC_RELAY_RUNSTATE': 512,
 'C_CYC_RELAY_SERVICE_REM': 0,
 'C_CYC_RIGHT': 0,
 'C_CYC_STATE_AWAY': 0,
 'C_CYC_STATE_BOOST': 0,
 'C_CYC_STATE_HOME': 0,
 'C_CYC_TESTED': 0,
 'C_CYC_TESTING': 0,
 'C_CYC_WATER': 0,
 'EXT_ACCESSRIGHTS_ARRAY': 0,
 'EXT_ANALOG_INPUT_MODE': 0,
 'EXT_AWAY_AIR_TEMP_TARGET': -273.1,
 'EXT_BOOST_AIR_TEMP_TARGET': -273.1,
 'EXT_BOOST_TIME': 0,
 'EXT_BOOST_TIMER_ENABLED': 0,
 'EXT_BOOT_FINISHED': 0,
 'EXT_BROWSER_IP': 0,
 'EXT_BROWSER_PORT': 0,
 'EXT_BYPASS_LOCKED': 0,
 'EXT_CLEAR_WEEK_EVENTS': 0,
 'EXT_CLOUD_CONNECT': 0,
 'EXT_CLOUD_CONNECTION_STATUS': 0,
 'EXT_CLOUD_DISCONNECT': 0,
 'EXT_CONFIG_BUTTON': 0,
 'EXT_CONFIG_DONE': 0,
 'EXT_CYC_ACTIVE_PROFILE': 0,
 'EXT_CYC_APPL_SW_VERSION': 0,
 'EXT_CYC_CONFIG_NUMBER': 0,
 'EXT_CYC_CURRENT_UP_TIME': 0,
 'EXT_CYC_DATE': 0,
 'EXT_CYC_DEFROSTING': 0,
 'EXT_CYC_DEFROST_RH_OFFSET': 0,
 'EXT_CYC_DEFROST_TEMP_PARAM': -273.1,
 'EXT_CYC_EXTRA_HEATER_TYPE': 0,
 'EXT_CYC_EXTR_FAN_SPEED': 0,
 'EXT_CYC_FAULT_ARRAY': 0,
 'EXT_CYC_FILTER_CHANGED_DATE': 0,
 'EXT_CYC_MLV_SUMMER_SETPOINT': 0,
 'EXT_CYC_MLV_SUPPLY_LOWER_LIMIT': 0,
 'EXT_CYC_MLV_WINTER_SETPOINT': 0,
 'EXT_CYC_OLD_WEEKLY_EVENTS': 0,
 'EXT_CYC_POST_HEATER_TYPE': 0,
 'EXT_CYC_POWER': 0,
 'EXT_CYC_PROFILE_OUTDOOR_AIR': 0,
 'EXT_CYC_PROFILE_SUPPLY_AIR': 0,
 'EXT_CYC_RESTORE_FACTORY_SETTINGS': 0,
 'EXT_CYC_RESTORE_INSTALLATION_SETTINGS': 0,
 'EXT_CYC_RESTORE_SERVICE_MODE': 0,
 'EXT_CYC_RESTORE_USER_SETTINGS': 0,
 'EXT_CYC_SAVE_INSTALLATION_SETTINGS': 0,
 'EXT_CYC_SAVE_USER_SETTINGS': 0,
 'EXT_CYC_SERIAL_NUMBER': 0,
 'EXT_CYC_START_SELFTEST': 0,
 'EXT_CYC_SUPP_FAN_SPEED': 0,
 'EXT_CYC_TIME': 0,
 'EXT_CYC_TOTAL_UP_TIME': 0,
 'EXT_CYC_WEEKLY_EVENTS': 0,
 'EXT_DATA_POLLING': 0,
 'EXT_DATA_POLLING_ENABLED': 0,
 'EXT_DATA_POLLING_INTERVAL': 0,
 'EXT_DATA_POLLING_UI_UPDATE_ENABLED': 0,
 'EXT_DATA_SC': 0,
 'EXT_DEFROST_ON': 0,
 'EXT_DIGITAL_INPUT_1_MODE': 0,
 'EXT_DIGITAL_INPUT_2_MODE': 0,
 'EXT_ERROR_SOLVED': 0,
 'EXT_EXTRA_AIR_TEMP_TARGET': -273.1,
 'EXT_EXTRA_EXTR_FAN': 0,
 'EXT_EXTRA_SUPP_FAN': 0,
 'EXT_EXTRA_TIME': 0,
 'EXT_EXTRA_TIMER_ENABLED': 0,
 'EXT_FILTER_CHANGE_INTERVAL': 0,
 'EXT_GATEWAY_ADDRESS': 0,
 'EXT_GRANT_ACCESS_EMAIL': 0,
 'EXT_GRANT_ACCESS_EMAIL_LIST': 0,
 'EXT_GRANT_ACCESS_EMAIL_VALID': 0,
 'EXT_GRANT_ACCESS_SEND': 0,
 'EXT_GRAPH_DURATION': 0,
 'EXT_GRAPH_DURATION_IN_DAYS': 0,
 'EXT_HOME_AIR_TEMP_TARGET': -273.1,
 'EXT_INFOBANNER_CONTENT': 0,
 'EXT_IP_ADDRESS': 0,
 'EXT_LANGUAGE': 0,
 'EXT_LC_CHANGED': 0,
 'EXT_LISTENER_ENABLED': 0,
 'EXT_LISTENER_SKIPPER': 0,
 'EXT_MACHINE_TYPE': 0,
 'EXT_MASK_ADDRESS': 0,
 'EXT_MODBUS_PARITY': 0,
 'EXT_MODBUS_STOPBIT': 0,
 'EXT_NUM_OF_CO2_SENSORS': 0,
 'EXT_NUM_OF_RH_SENSORS': 0,
 'EXT_OFFLINE_MODE': 0,
 'EXT_PAGES_LOADED': 0,
 'EXT_POST_HEATER_WINTER_SETPOINT': 0,
 'EXT_REFRESH_GRAPHS': 0,
 'EXT_SEN_ANALOG_SENSOR_INPUT': 0,
 'EXT_SEN_CO2_SENSOR_0': 0,
 'EXT_SEN_CO2_SENSOR_1': 0,
 'EXT_SEN_CO2_SENSOR_2': 0,
 'EXT_SEN_CO2_SENSOR_3': 0,
 'EXT_SEN_CO2_SENSOR_4': 0,
 'EXT_SEN_CO2_SENSOR_5': 0,
 'EXT_SEN_RH_SENSOR_0': 0,
 'EXT_SEN_RH_SENSOR_1': 0,
 'EXT_SEN_RH_SENSOR_2': 0,
 'EXT_SEN_RH_SENSOR_3': 0,
 'EXT_SEN_RH_SENSOR_4': 0,
 'EXT_SEN_RH_SENSOR_5': 0,
 'EXT_SERVICE_MODE': 0,
 'EXT_SHOW_WIZARD': 0,
 'EXT_TEMP_NOW_EXHAUST_AIR': -273.1,
 'EXT_TEMP_NOW_EXTRACT_AIR': -273.1,
 'EXT_TEMP_NOW_OUTDOOR_AIR': -273.1,
 'EXT_TEMP_NOW_SUPPLY_AIR': -273.1,
 'EXT_TEST_MODE_SWITCH': 0,
 'EXT_USERNAME': 0,
 'EXT_USER_PASSWORD': 0,
 'EXT_UUID_STRING': 0,
 'EXT_WIZARD_DONE': 0,
 'RANGE_END_g_cyclone_config': 58610,
 'RANGE_END_g_cyclone_general_info': 0,
 'RANGE_END_g_cyclone_hw_state': 0,
 'RANGE_END_g_cyclone_input': 0,
 'RANGE_END_g_cyclone_output': 0,
 'RANGE_END_g_cyclone_settings': 0,
 'RANGE_END_g_cyclone_sw_state': 0,
 'RANGE_END_g_cyclone_time': 7,
 'RANGE_END_g_cyclone_weekly_schedule': 0,
 'RANGE_END_g_faults': 0,
 'RANGE_END_g_self_test': 0,
 'RANGE_END_g_typhoon_general_info': 0,
 'RANGE_END_g_typhoon_settings': 0,
 'RANGE_START_g_cyclone_config': 0,
 'RANGE_START_g_cyclone_general_info': 0,
 'RANGE_START_g_cyclone_hw_state': 0,
 'RANGE_START_g_cyclone_input': 0,
 'RANGE_START_g_cyclone_output': 0,
 'RANGE_START_g_cyclone_settings': 0,
 'RANGE_START_g_cyclone_sw_state': 0,
 'RANGE_START_g_cyclone_time': 0,
 'RANGE_START_g_cyclone_weekly_schedule': 0,
 'RANGE_START_g_faults': 0,
 'RANGE_START_g_self_test': 0,
 'RANGE_START_g_typhoon_general_info': 0,
 'RANGE_START_g_typhoon_settings': 0,
 'WS_LOG_EXHAUST_AIR_TEMP': 0,
 'WS_LOG_EXTRACT_AIR_TEMP': 0,
 'WS_LOG_FAN_SPEED': 512,
 'WS_LOG_MAX_CO2': 0,
 'WS_LOG_MAX_HUMIDITY': 0,
 'WS_LOG_METRICS_1': 0,
 'WS_LOG_OUTDOOR_AIR_TEMP': 0,
 'WS_LOG_SUPPLY_AIR_TEMP': 0,
 'WS_LOG_SUPPLY_CELL_AIR_TEMP': 0,
 'WS_WEB_UI_ACK': 0,
 'WS_WEB_UI_COMMAND_LOG': 0,
 'WS_WEB_UI_COMMAND_LOG_LIMITED': 0,
 'WS_WEB_UI_COMMAND_LOG_RAW': 0,
 'WS_WEB_UI_COMMAND_READ_DATA': 0,
 'WS_WEB_UI_COMMAND_READ_TABLES': 0,
 'WS_WEB_UI_COMMAND_WRITE_DATA': 0,
 'WS_WEB_UI_DATA_BOUNDARY_ERROR': 0,
 'WS_WEB_UI_DATA_CHECKSUM_ERROR': 0,
 'WS_WEB_UI_DATA_ERROR': 0,
 'WS_WEB_UI_DATA_INVALID_COMMAND': 0,
 'WS_WEB_UI_DATA_LENGTH_ERROR': 0,
 'WS_WEB_UI_DATA_OK': 0,
 'WS_WEB_UI_DATA_REQUEST': 0,
 'WS_WEB_UI_DATA_RW_ERROR': 0,
 'WS_WEB_UI_DATA_SEND_REPLY': 0,
 'WS_WEB_UI_DATA_TABLE_BOUNDARY_ERROR': 0}
 ```

# Reading
* https://www.vallox.com/files/1092/Modbus-instructions_ENG-250915.pdf
* https://www.vallox.com/files/1092/Manual_Modbus_ENG_20180307_PRINT.pdf
