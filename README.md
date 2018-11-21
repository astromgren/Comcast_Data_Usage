# Comcast_Data_Usage
Python script designed to return Comcast internet usage data for us in [Home Assistant](https://www.home-assistant.io).  Based on https://github.com/ntalekt/Cox_Data_Usage and https://github.com/lachesis/comcast

## Home Assistant Card Example

![Alt text](/img/Card.PNG?raw=true)

## Configuration

Modify the `username` and `password` variables to match your Comcast main account. Update the `json_file` variable if required.

Put `comcast.py` somewhere inside your Home Assistant configuration directory.

```
username = "USERNAME"
password = "PASSWORD"
json_file = "/home/homeassistant/.homeassistant/comcast/comcast_usage.json"
```

## Automation

```
automation:
  - alias: Query Comcast Data Usage
  trigger:
     platform: time
     minutes: '/60'
     seconds: 00
  action: 
    service: shell_command.query_comcast_data_usage
```

## Shell Command Component

```
shell_command:
  query_comcast_data_usage: 'python /home/homeassistant/.homeassistant/comcast.py'
```

## Sensor Component

```
  - platform: command_line
    command: date +"%d"
    name: Day Of Current Month
    scan_interval: 3600

  - platform: command_line
    command: cal $(date +"%m %Y") | awk 'NF {DAYS = $NF}; END {print DAYS}'
    name: Days In Current Month
    scan_interval: 3600
    
  - platform: file
    name: Comcast Utilization
    file_path: /home/homeassistant/.homeassistant/comcast/comcast_usage.json
    value_template: >
      {% if value_json is defined %}
        {% if value_json.used | int == 0 and value_json.total | int == 0 %}
          stats unavailable
        {% else %}
          {{ value_json.used | int }} / {{ value_json.total | int }} GB ({{ ((value_json.used /  value_json.total)*100) | round(1) }}%) 
        {% endif %}
      {% else %}
        undefined
      {% endif %}

  - platform: file
    name: Comcast Avg GB Current
    file_path: /home/homeassistant/.homeassistant/comcast/comcast_usage.json
    value_template: >
      {% if value_json is defined %}
        {% if value_json.used | int == 0 and value_json.total | int == 0 %}
          stats unavailable
        {% else %}
         {{ ((value_json.used | int) / (states.sensor.day_of_current_month.state | int)) | round(1) }} GB per day
        {% endif %}
      {% endif %}

  - platform: file
    name: Comcast Avg GB Left
    file_path: /home/homeassistant/.homeassistant/comcast/comcast_usage.json
    value_template: >
      {% if value_json is defined %}
        {% if value_json.used | int == 0 and value_json.total | int == 0 %}
          stats unavailable
        {% else %}
          {{ (((value_json.total | int) - (value_json.used | int)) / ((states.sensor.days_in_current_month.state | int ) - (states.sensor.day_of_current_month.state | int))) | round(1) }} GB per day
        {% endif %}
      {% else %}
        undefined
      {% endif %}

```

## Customize
```
sensor.comcast_utilization:
  icon: mdi:percent
  friendly_name: Utilization
sensor.comcast_template:
  icon: mdi:calendar-clock
  friendly_name: Time Left
sensor.comcast_avg_gb_current:
  icon: mdi:chart-line
  friendly_name: Current Daily Avg.
sensor.comcast_avg_gb_left:
  icon: mdi:chart-line-stacked
  friendly_name: Remaining Daily Avg.
```
## Required dependencies
The only requirement for the Python script is the requests library.
