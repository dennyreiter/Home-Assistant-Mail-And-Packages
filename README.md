![GitHub release (latest by date)](https://img.shields.io/github/v/release/moralmunky/Home-Assistant-Mail-And-Packages)
[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/custom-components/hacs)

![GitHub contributors](https://img.shields.io/github/contributors/moralmunky/Home-Assistant-Mail-And-Packages)
![Maintenance](https://img.shields.io/maintenance/yes/2020)
![GitHub commit activity](https://img.shields.io/github/commit-activity/m/moralmunky/Home-Assistant-Mail-And-Packages)
![GitHub last commit](https://img.shields.io/github/last-commit/moralmunky/Home-Assistant-Mail-And-Packages)

# HomeAssistant Mail and Packages

This provides a Card, Custom Component, and Notifications for getting UPS, USPS, and FedEx delivery information in Home Assistant. This component shows a snapshot of the current days packages that are in transit for delivery on the current day or have already been delivered on the current day. It also generates the number of USPS mail pieces and images, if available, for the current day.

The component connects to the email account you supply where your shipment notifications are sent. It looks at the subject lines of the current day's emails from the shipping companies and counts the subject line matches to the known standard subject lines from the shipping companies. For USPS Informed delivery emails, it also downloads the mail images to combine them into a rotating GIF.

All procedures are done locally on your machine.

Supports only Lovelace UI. Last tested in 0.105.x.

<img src="https://github.com/moralmunky/Home-Assistant-Mail-And-Packages/blob/master/mail_card_screenshot.jpg" alt="Preview of the custom mail card" width="350" />  <img src="https://github.com/moralmunky/Home-Assistant-Mail-And-Packages/blob/master/notification_screenshot.jpg" alt="Preview of the custom mail card" width="350" />

## Credits:

* Mail Card based on Bram_Kragten work at [Home Assistant Community: Custom UI weather state card, with a question](https://community.home-assistant.io/t/custom-ui-weather-state-card-with-a-question/23008)
* Mail.py script based on @skalavala work at [skalavala blog](https://blog.kalavala.net/usps/homeassistant/mqtt/2018/01/12/usps.html)
* Package and macros based on happyleavesaoc work at [happyleavesaoc/my-home-automation](https://github.com/happyleavesaoc/my-home-automation)

## Requirements:

[USPS Informed Delivery:](https://informeddelivery.usps.com/) account and all nortifications turned on for email with the email address you will have the component check.

<img src="https://github.com/moralmunky/Home-Assistant-Mail-And-Packages/blob/master/USPS_Delivery_Notifications.jpg" alt="USPS Informed Delivery notification settings."  width="350"/>

[FedEx Delivery Manager:](https://www.fedex.com/apps/fdmenrollment/) account and all nortifications turned on for email with the email address you will have the component check.

<img src="https://github.com/moralmunky/Home-Assistant-Mail-And-Packages/blob/master/FedEx_Delivery_Notifications.jpg" alt="FedEx notification settings."  width="350"/>

[UPS MyChoice:](https://wwwapps.ups.com/mcdp?loc=en_US) account and all nortifications turned on for email with the email address you will have the component check.

<img src="https://github.com/moralmunky/Home-Assistant-Mail-And-Packages/blob/master/UPS_My_Choice_Notifications.jpg" alt="FedEx notification settings."  width="350"/>

## Installation (HACS) - Highly Recommended
0. Have [HACS](https://github.com/custom-components/hacs) installed, this will allow you to easily update
1. Add `https://github.com/moralmunky/Home-Assistant-Mail-And-Packages` as a [custom repository](https://custom-components.github.io/hacs/usage/settings/#add-custom-repositories) as Type: Integration
2. Click install under "Mail and Packages", restart your instance.

## Installation (Manual)
1. Download this repository as a ZIP (green button, top right) and unzip the archive
2. Copy `/custom_components/mail_and_packages` to your `<config_dir>/custom_components/` directory
   * You will need to create the `custom_components` folder if it does not exist
   * On HomeAssistant (formerly hass.io) the final location will be `/config/custom_components/mail_and_packages`
   * On Hassbian the final location will be `/home/homeassistant/.homeassistant/custom_components/mail_and_packages`
3. Copy/move `image-no-mailpieces700.jpg` and `mail_none.gif` to the images directory (recommend `<config_dir>/images/mail_and_packages` as the path)

## Configuration/HASS Set Up
Once you have finished either installing via HACS or manually (and rebooted HASS), go into ```Configuration -> Integration``` select the ```+```and add the ```Mail And Packages``` integration. You will be prompted to input your mail server settings.

## Privacy / Security Note
Please note that anything in the ```www``` folder is techncially publicly accessable unless you have taken security measures like a reverse proxy with authentication enabled. You can place the images in an ```images``` directory and still send notifications via most notification integrations.

### Template
Use the following to create a deliveries summary sensor (under the sensor portion of configuartion.yaml):
```
- platform: template
  sensors:
    mail_deliveries_message:
      friendly_name: "Deliveries Summary"
      entity_id: 
        - sensor.mail_usps_mail
        - sensor.mail_usps_delivering
        - sensor.mail_fedex_delivering
        - sensor.mail_ups_delivering
      value_template: > 
        {# Deliveries Sentence #}
          {% macro deliveries_sentence() -%}
                {%- if states("sensor.mail_usps_mail")|int == 0 -%}
                  No
                {%- else -%}
                  {{states("sensor.mail_usps_mail")|int}}
                {%- endif -%}
              {{' '}} 
                {%- if states("sensor.mail_usps_mail")|int <= 1 -%}
                  pieces of mail
                {%- else -%}
                  pieces of mail
                {%- endif -%}
              {{' '}}will be delivered.{{' '}} 
                {%- if states("sensor.mail_usps_delivering")|int == 0 -%}
                  No
                {%- else -%}
                  {{states("sensor.mail_usps_delivering")|int}}
                {%- endif -%}
              {{' '}} 
                {%- if states("sensor.mail_usps_delivering")|int == 1 -%}
                  USPS package is
                {%- else -%}
                  USPS packages are
                {%- endif -%}
              {{' '}}in transit.{{' '}}
                {%- if states("sensor.mail_fedex_delivering")|int == 0 -%}
                  No
                {%- else -%}
                  {{states("sensor.mail_fedex_delivering")|int}}
                {%- endif -%}
              {{' '}} 
                {%- if states("sensor.mail_fedex_delivering")|int == 1 -%}
                  FedEx package is
                {%- else -%}
                  Fedex packages are
                {%- endif -%}
              {{' '}}in transit.{{' '}}
              {%- if states("sensor.mail_ups_delivering")|int == 0 -%}
                  No
                {%- else -%}
                  {{states("sensor.mail_ups_delivering")|int}}
                {%- endif -%}
              {{' '}} 
                {%- if states("sensor.mail_ups_delivering")|int == 1 -%}
                  UPS package is
                {%- else -%}
                  UPS packages are
                {%- endif -%}
              {{' '}}in transit.{{' '}}
              {%- if states("sensor.mail_amazon_packages")|int == 0 -%}
                  No
                {%- else -%}
                  {{states("sensor.mail_amazon_packages")|int}}
                {%- endif -%}
              {{' '}} 
                {%- if states("sensor.mail_amazon_packages")|int == 1 -%}
                  Amazon package is
                {%- else -%}
                  Amazon packages are
                {%- endif -%}
              {{' '}}in transit.{{' '}}
          {%- endmacro %}
        {{deliveries_sentence()}}
```
### Camera
You may also want to add a camera to display the image.  Add to the camera portion of configuration.yaml.
```
  - platform: local_file
    file_path: /config/images/mail_and_packages/mail_today.gif
    name: mail_usps
```
### Automation
```
- alias: "Mail Notif - Mail Delieveries"
  initial_state: 'on'
  trigger:
  #Trigger if mail or packages get updated
    - platform: state
      entity_id: sensor.mail_usps_mail
    - platform: state
      entity_id: sensor.mail_usps_delivering
  #send only if mail or packages are more than 0
  condition:
    - condition: or
      conditions:
        - condition: template
          value_template: "{{ states('sensor.mail_usps_mail') | int > 0 }}"
        - condition: template
          value_template: "{{ states('sensor.mail_usps_delivering') | int > 0 }}"
  action:
    - service: notify.YOUR_NOTIFY_METHOD
      data_template:
        title: "*Today's Mail and Packages*"
        message: "{{ states('sensor.mail_deliveries_message')}}"
    - service: notify.YOUR_NOTIFY_METHOD
      data:
        message: "Here is the mail"
        data:
          document:
            file: "PATH_TO_FILE/mail_today.gif"
```

## Lovelace Custom Card Setup

A few options here as provided by the community:
Note: The local file camera reloads the file after 10 seconds. If the gif is longer than 10 seconds not all mail with be shown. Please use a [picture entity card](https://www.home-assistant.io/lovelace/picture-entity/) with the camera view set to live.

### Option 1 (requires [vertical stack in card custom card](https://github.com/ofekashery/vertical-stack-in-card))
```
- type: custom:vertical-stack-in-card
      title: Mail & Package Tracking
      cards:
        - aspect_ratio: 50%
          camera_view: live
          entity: camera.mail_usps
          name: Mail
          type: picture-entity
        - type: entity-filter
          state_filter:
            - operator: ">"
              value: '0'
          entities:
           - entity: sensor.mail_usps_mail
             name: "Today's Mail"
           - entity: sensor.mail_packages_in_transit
             name: "Today's Package Delivery"
           - entity: sensor.mail_usps_delivering
             icon: 'mdi:package-variant-closed'
             name: USPS
           - entity: sensor.mail_fedex_delivering
             icon: 'mdi:package-variant-closed'
             name: FedEx
           - entity: sensor.mail_ups_delivering
             icon: 'mdi:package-variant-closed'
             name: UPS
           - entity: sensor.mail_updated
             state_filter:
               - operator: "regex"
                 value: 20
```
### Option 2 (also requires vertical stack in card)
```
cards:
  - cards: null
    entities:
      - entity: sensor.mail_usps_mail
        name: Todays USPS Mail
      - entity: sensor.mail_packages_in_transit
        name: Todays Package Delivery
    type: entities
  - cards: null
    entities:
      - entity: sensor.mail_usps_delivering
        icon: 'mdi:package-variant-closed'
        name: USPS
      - entity: sensor.mail_fedex_delivering
        icon: 'mdi:package-variant-closed'
        name: FedEx
      - entity: sensor.mail_ups_delivering
        icon: 'mdi:package-variant-closed'
        name: UPS
    show_header_toggle: false
    type: glance
  - aspect_ratio: 50%
    camera_view: live
    entity: camera.mail_usps
    name: Mail
    type: picture-entity
  - cards: null
    entities:
      - entity: sensor.mail_updated
    type: entities
title: Mail & Package Tracking
type: 'custom:vertical-stack-in-card'
```
### Option 3 (no custom card required)
```
 - cards:
    - aspect_ratio: 50%
      camera_view: live
      entity: camera.mail_usps
      name: Mail
      type: picture-entity
    - entities:
        - entity: sensor.mail_usps_mail
          name: USPS Mail
        - entity: sensor.mail_usps_packages
          name: USPS Packages
        - entity: sensor.fedex_packages
          name: FedEx Packages
        - sensor.mail_ups_packages
        - sensor.mail_packages_in_transit
        - sensor.mail_packages_delivered
        - entity: sensor.mail_updated
          state_filter:
            - operator: regex
              value: 20
      state_filter:
        - operator: '>'
          value: '0'
      type: entity-filter
  title: Mail Today
  type: vertical-stack
```
