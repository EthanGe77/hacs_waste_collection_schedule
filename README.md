# Waste Collection Schedule

Framework to easily integrate appointment schedule services, e.g. waste collection services into Home Assistant. The entity state and details can be easily customized using templates. New sources can be added very easily.

## Showroom

Entity state is highly customizable using templates:

<img src="https://github.com/mampfes/hacs_waste_collection_schedule/blob/master/doc/value_template.png">

Details view showing the list of upcoming events:

<img src="https://github.com/mampfes/hacs_waste_collection_schedule/blob/master/doc/upcoming_details.png">

Date format is also customizable using templates:

<img src="https://github.com/mampfes/hacs_waste_collection_schedule/blob/master/doc/date_template_details.png">

Alternative details view showing the list of appointment types and their next event:

<img src="https://github.com/mampfes/hacs_waste_collection_schedule/blob/master/doc/appointment_types_details.png">

[Button Cards](https://github.com/custom-cards/button-card) can be used to create individual widgets:

## List of Sources

Currently the following sources are available:

- [Abfall_Kreis_Tuebingen.de](./doc/source/abfall_kreis_tuebingen_de.md)
- [AbfallNavi / RegioIT.de](./doc/source/regioit_de.md)
- [AbfallPlus.de / Abfall.IO](./doc/source/abfall_io.md)

## Configuration via configuration.yaml

The configuration consists of 2 entries in configuration.yaml.

1. Source configuration<br>
   A source is a used to retrieve the data from a web service. Multiple source can be created at the same time. Even a single source can be created multiple times (with different arguments), e.g. if you want to see the waste collection schedule for multple districts.
  
2. Sensor configuration<br>
   For every Source, one or more sensors can be defined to visualize the retrieved data. For each sensor, the entity state format, date format, details view and other properties can be customized.

## 1. Configure the Source(s)

```yaml
waste_collection_schedule:
  sources:
    - name: SOURCE
      fetch_time: FETCH_TIME
      day_switch_time: DAY_SWITCH_TIME
      separator: SEPARATOR
      args:
        SOURCE_SPECIFIC_ARGUMENTS
      customize:
        - type: TYPE
          alias: ALIAS
          icon: ICON
          picture: PICTURE
```

### Configuration Variables

**source**<br>
*(list) (required)*

List of sources (see below).

**name**<br>
*(string) (required)*

Name of the source. Equates to the file name (without ```.py```) of the source. See [List of Sources](#list-of-sources) for a list of available sources.

**fetch_time**<br>
*(time) (optional, default: ```"01:00"```)*

Time of day when to fetch new data from the source. Data will be fetched once per day.

**day_switch_time**<br>
*(time) (optional, default: ```"10:00"```)*

Time of day when to remove the viewed appointments for today.

How it works: If you set the ```day_switch_time``` to 10:00 the sensor will view today's appointments until 10:00. After 10:00 the sensor will remove the today's appointments and only show the upcoming events.

**separator**<br>
*(string) (optional, default: ```", "```)*

Used to join multiple appointments for either entity state or appointment-types list (n/a if value_templates are used).

**args**<br>
*(dict) (optional)*

Arguments for source, e.g. city, district, street, waste type, etc. See [List of Sources](#list-of-sources) for details.

**customize**<br>
*(dict) (optional)*

See [Customize Source](#customize-source) for details.

### Customize Source

Used to customize the retrieved data from a source.

**type**<br>
*(dict) (required)*

Appointment type name as is has been retrieved by the source.

**alias**<br>
*(string) (optional, default: ```None```)*

Alternative name for appointment type which shall be used instead of ```type```.

**icon**<br>
*(string) (optional, default: ```None```)*

Alternative entity icon for appointment type which sall be used instead of the default icon.

**picture**<br>
*(string) (optional, default: ```None```)*

Optional entity picture for appointment type.

## 2. Add one or more Sensors to the Source(s)

Add the following lines to your `configuration.yaml` file:

```yaml
sensor:
  - platform: waste_collection_schedule
    source_index: SOURCE_INDEX
    name: NAME
    details_format: DETAILS_FORMAT
    count: COUNT
    leadtime: LEADTIME
    value_template: VALUE_TEMPLATE
    date_template: DATE_TEMPLATE
    types:
      - Appointment Type 1
      - Appointment Type 2
```

### Configuration Variables

**source_index**<br>
*(integer) (optional, default: ```0```)*

Reference to source. Required to assign a sensor to a specific source if there are multiple sources defined. Can be omitted if there is only one source defined. The first source has the source_index 0, the second source has 1, ...

**name**<br>
*(string) (required)*

Name of the sensor.

**details_format**<br>
*(string) (optional, default: ```"upcoming"```)*

Used to specify the format of the device-state-attribute, which are shown in the details for in the Lovelace UI.

Possible choices:

- ```upcoming``` shows a list of upcoming events.<br>
  ![Upcoming](./doc/upcoming_details.png "Upcoming")
- ```appointment_types``` shows a list of appointment types and their next event.<br>
  ![Appointment Types](./doc/appointment_types_details.png "Appointment Types")
- ```generic``` provides all attributes as generic Python data types. This can be used by a specialized Lovelace card (which doesn't exist so far).<br>
  ![Generic](./doc/generic_details.png "Generic")

**count**<br>
*(integer) (optional, default = infinite)*

Used to limit the number of displayed appointments in the details view of an entity by ```count```.

**leadtime**<br>
*(integer) (optional, default = infinite)*

Used to limit the number of displayed appointments in the details view of an entity. Only appointment within the next ```leadtime``` days will be shown.

**value_template**<br>
*(string) (optional)*

Template string used to render the state of an entity.

[List of available variables](#list-of-available-variables-within-templates) within the template.

**date_template**<br>
*(string) (optional)*

Template string used to render appointment dates within the details view.

[List of available variables](#list-of-available-variables-within-templates) within the template.

**types**<br>
*(list of strings) (optional)*

Filter for appointment types. Can be used to create a sensor for a single or dedicated list of appointment types. Can be also used to sort the list of appointments, especially for the appointment_types details view.

## List of available Variables within Templates

The following variables are available:

- ```value.date``` Appointment date, Type: datetime.date
- ```value.daysTo``` Days to appointment, 0 = today, 1 = tomorrow, ..., Type: int
- ```value.types``` Appointment types, Type: list of strings

Examples:

```yaml
# returns "Type1 and Type2 in 7 days"
'{{value.types|join(" + ")}} in {{value.daysTo}} days'

# returns "Type1, Type2 on Fri, 03/20/2020"
'{{value.types|join(", ")}} on {{value.date.strftime("%a, %m/%d/%Y")}}'
```

## Examples

### Button Cards

[Button Cards](https://github.com/custom-cards/button-card) can be used to create individual widgets:

![Button Cards](./doc/button-cards.png "Button Cards")

```yaml
# configuration.yaml
sensor:
  - platform: waste_collection_schedule
    name: MyButtonCardSensor
    value_template: '{{value.types|join(", ")}}|{{value.daysTo}}|{{value.date.strftime("%d.%m.%Y")}}|{{value.date.strftime("%a")}}'
```

```yaml
# button-card configuration
type: 'custom:button-card'
entity: sensor.mybuttoncardsensor
layout: icon_name_state2nd
show_label: true
label: |
  [[[
    var days_to = entity.state.split("|")[1]
    if (days_to == 0)
    { return "Today" }
    else if (days_to == 1)
    { return "Tomorrow" }
    else
    { return "in " + days_to + " days" }
  ]]]
show_name: true
name: |
  [[[
    return entity.state.split("|")[0]
  ]]]
state:
  - color: red
    operator: template
    value: '[[[ return entity.state.split("|")[1] == 0 ]]]'
  - color: orange
    operator: template
    value: '[[[ return entity.state.split("|")[1] == 1 ]]]'
  - value: default
```

## How to add new sources

1. Create a new source in folder `custom_components/waste_collection_schedule/package/source` with the lower case url of your service provider (e.g. `abc_com.py` for `http://www.abc.com`).
2. Don't forget to add test cases (= sample data to query the service api).
3. Run `test_sources.py` script to ensure that your source works.
4. Add documentation in folder `docs/source` and add a link to your new source on `README.md`.

Example for `abc_com.py`:

```py
import datetime
from ..helpers import CollectionAppointment
from collections import OrderedDict


DESCRIPTION = "Example source for abc.com"  # Describe your source
URL = "abc.com"    # Insert url to service homepage
TEST_CASES = OrderedDict( # Insert arguments for test cases using test_sources.py script
    [("TestName", {"arg1": 100, "arg2": "street"})]
)


class Source:
    def __init__(self, arg1, arg2): # argX correspond to the args dict in the source configuration
        self._arg1 = arg1
        self._arg2 = arg2

    def fetch(self):
        entries = []

        entries.append(
            CollectionAppointment(
                datetime.datetime(2020, 4, 11),
                "Appointment Type",
            )
        )

        return entries
```

See also: [custom_components/waste_collection_schedule/package/source/example.py](./custom_components/waste_collection_schedule/package/source/example.py)
