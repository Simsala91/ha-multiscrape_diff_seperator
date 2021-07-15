# HA Multiscrape

[![GitHub Release][releases-shield]][releases]
[![License][license-shield]](LICENSE)

[![pre-commit][pre-commit-shield]][pre-commit]
[![Black][black-shield]][black]

[![hacs][hacsbadge]][hacs]
[![Project Maintenance][maintenance-shield]][user_profile]
[![BuyMeCoffee][buymecoffeebadge]][buymecoffee]

[![Discord][discord-shield]][discord]
[![Community Forum][forum-shield]][forum]

## BREAKING CHANGE in release 4.0.0

If you are upgrading to release 4.0.0, see the [upgrade notes](https://github.com/danieldotnl/ha-multiscrape/wiki/Upgrade-notes-for-3.x-to-4.x).

# HA MultiScrape custom component

This Home Assistant custom component can scrape multiple fields (using CSS selectors) from a single HTTP request (the existing scrape sensor can scrape a single field only). The scraped data becomes available in separate sensors.

It is based on both the existing [Rest sensor](https://www.home-assistant.io/integrations/rest/) and the [Scrape sensor](https://www.home-assistant.io/integrations/scrape). Most properties of the Rest and Scrape sensor apply.

<a href="https://www.buymeacoffee.com/danieldotnl" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-blue.png" alt="Buy Me A Coffee" style="height: 51px !important;width: 217px !important;" ></a>

## Installation

[![hacs][hacsbadge]][hacs]

Install via HACS (default store) or install manually by copying the files in a new 'custom_components/multiscrape' directory.

## Example configuration (YAML)

```yaml
multiscrape:
  - resource: https://www.home-assistant.io
    scan_interval: 3600
    sensor:
      - unique_id: ha_latest_version
        name: Latest version
        select: ".current-version > h1:nth-child(1)"
        value_template: '{{ (value.split(":")[1]) }}'
      - unique_id: ha_release_date
        icon: >-
          {% if is_state('binary_sensor.ha_version_check', 'on') %}
            mdi:alarm-light
          {% else %}
            mdi:bat
          {% endif %}
        name: Release date
        select: ".release-date"
    binary_sensor:
      - unique_id: ha_version_check
        name: Latest version == 2021.7.0
        select: ".current-version > h1:nth-child(1)"
        value_template: '{{ (value.split(":")[1]) | trim == "2021.7.0" }}'
        attributes:
          - name: Release notes link
            select: "div.links:nth-child(3) > a:nth-child(1)"
            attribute: href
```

## Options

Based on latest (pre) release.

| name              | description                                                                                                               | required | default | type          |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------- | -------- | ------- | ------------- |
| name              | The name for the integration.                                                                                             | False    |         | string        |
| resource          | The url for retrieving the site or a template that will output an url. Not required when `resource_template` is provided. | True     |         | string        |
| resource_template | A template that will output an url after being rendered. Only required when `resource` is not provided.                   | True     |         | template      |
| authentication    | Configure HTTP authentication. `basic` or `digest`. Use this with username and password fields.                           | False    |         | string        |
| username          | The username for accessing the url.                                                                                       | False    |         | string        |
| password          | The password for accessing the url.                                                                                       | False    |         | string        |
| headers           | The headers for the requests.                                                                                             | False    |         | string - list |
| params            | The query params for the requests.                                                                                        | False    |         | string - list |
| method            | The method for the request. Either `POST` or `GET`.                                                                       | False    | GET     | string        |
| payload           | Optional payload to send with a POST request.                                                                             | False    |         | string        |
| verify_ssl        | Verify the SSL certificate of the endpoint.                                                                               | False    | True    | boolean       |
| timeout           | Defines max time to wait data from the endpoint.                                                                          | False    | 10      | int           |
| scan_interval     | Determines how often the url will be requested.                                                                           | False    | 60      | int           |
| parser            | Determines the parser to be used with beautifulsoup. Either `lxml` or `html.parser`.                                      | False    | lxml    | string        |
| form_submit       | See [Form-submit](#form-submit)                                                                                           | False    |         |               |
| sensor            | See [Sensor](#sensorbinary-sensor)                                                                                        | False    |         | list          |
| binary_sensor     | See [Binary sensor](#sensorbinary-sensor)                                                                                 | False    |         | list          |

### Sensor/Binary Sensor

Configure the sensors that will scrape the data.

| name                | description                                                                                                                                                                       | required | default | type            |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- | --------------- |
| unique_id           | Will be used as entity_id and enables editing the entity in the UI                                                                                                                | False    |         | string          |
| name                | Friendly name for the sensor                                                                                                                                                      | False    |         | string          |
| select              | CSS selector used for retrieving the value of the sensor                                                                                                                          | True     |         | string          |
| attribute           | Attribute from the selected element to read as value                                                                                                                              | False    |         | string          |
| index               | The occurence to read when the selector returns multiple results                                                                                                                  | False    | 0       | int             |
| value_template      | Defines a template applied on the result of the selector to extract the value. For binary sensors, the sensor is on if the template evaluates as True                             | False    |         | string/template |
| attributes          | See [Sensor attributes](#sensor-attributes)                                                                                                                                       | False    |         | list            |
| unit_of_measurement | Defines the units of measurement of the sensor                                                                                                                                    | False    |         | string          |
| device_class        | Sets the device_class for [sensors](https://www.home-assistant.io/integrations/sensor/) or [binary sensors](https://www.home-assistant.io/integrations/binary_sensor/)            | False    |         | string          |
| icon                | Defines the icon or a template for the icon of the sensor. The value of the selector is provided as input for the template. For binary sensors, the value is parsed in a boolean. | False    |         | string/template |
| force_update        | Sends update events even if the value hasn’t changed. Useful if you want to have meaningful value graphs in history.                                                              | False    | False   | boolean         |

### Sensor attributes

Configure the attributes on the sensor that can be set with additional scraping values.

| name           | description                                                                   | required | default | type            |
| -------------- | ----------------------------------------------------------------------------- | -------- | ------- | --------------- |
| name           | Name of the attribute (will be slugified)                                     | True     |         | string          |
| select         | CSS selector used for retrieving the value of the attribute                   | True     |         | string          |
| attribute      | Attribute from the selected element to read as value                          | False    |         | string          |
| index          | The occurence to read when the selector returns multiple results              | False    | 0       | int             |
| value_template | Defines a template applied on the result of the selector to extract the value | False    |         | string/template |

### Form-submit

Configure the form-submit functionality which enables you to submit a (login) form before scraping a site. More details on how this works [can be found on the wiki.](https://github.com/danieldotnl/ha-multiscrape/wiki/Form-submit-functionality)

| name              | description                                                                          | required | default | type          |
| ----------------- | ------------------------------------------------------------------------------------ | -------- | ------- | ------------- |
| resource          | The url for the site with the form                                                   | False    |         | string        |
| select            | CSS selector used for selecting the form in the html                                 | True     |         | string        |
| input             | A dictionary with name/values which will be merged with the input fields on the form | False    |         | string - list |
| submit_once       | Submit the form only once on startup instead of each scan interval                   | False    | False   | boolean       |
| resubmit_on_error | Resubmit the form after a scraping error is encountered                              | False    | True    | boolean       |

## Services

For each integration instance, a service will be created to trigger a "manual" scrape run.
The services are named `multiscrape.trigger_{name of integration}`.

### Contributions are welcome!

If you want to contribute to this please read the [Contribution guidelines](CONTRIBUTING.md)

### Credits

This project was generated from [@oncleben31](https://github.com/oncleben31)'s [Home Assistant Custom Component Cookiecutter](https://github.com/oncleben31/cookiecutter-homeassistant-custom-component) template.

Code template was mainly taken from [@Ludeeus](https://github.com/ludeeus)'s [integration_blueprint][integration_blueprint] template

---

[integration_blueprint]: https://github.com/custom-components/integration_blueprint
[black]: https://github.com/psf/black
[black-shield]: https://img.shields.io/badge/code%20style-black-000000.svg?style=for-the-badge
[buymecoffee]: https://www.buymeacoffee.com/danieldotnl
[buymecoffeebadge]: https://img.shields.io/badge/buy%20me%20a%20coffee-donate-yellow.svg?style=for-the-badge
[commits-shield]: https://img.shields.io/github/commit-activity/y/danieldotnl/ha-multiscrape.svg?style=for-the-badge
[commits]: https://github.com/danieldotnl/ha-multiscrape/commits/main
[hacs]: https://hacs.xyz
[hacsbadge]: https://img.shields.io/badge/HACS-Default-orange.svg?style=for-the-badge
[discord]: https://discord.gg/Qa5fW2R
[discord-shield]: https://img.shields.io/discord/330944238910963714.svg?style=for-the-badge
[forum-shield]: https://img.shields.io/badge/community-forum-brightgreen.svg?style=for-the-badge
[forum]: https://community.home-assistant.io/t/scrape-sensor-improved-scraping-multiple-values/218350
[license-shield]: https://img.shields.io/github/license/danieldotnl/ha-multiscrape.svg?style=for-the-badge
[maintenance-shield]: https://img.shields.io/badge/maintainer-%40danieldotnl-blue.svg?style=for-the-badge
[pre-commit]: https://github.com/pre-commit/pre-commit
[pre-commit-shield]: https://img.shields.io/badge/pre--commit-enabled-brightgreen?style=for-the-badge
[releases-shield]: https://img.shields.io/github/release/danieldotnl/ha-multiscrape.svg?style=for-the-badge
[releases]: https://github.com/danieldotnl/multiscrape/releases
[user_profile]: https://github.com/danieldotnl
