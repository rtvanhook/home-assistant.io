---
type: card
title: Conditional Card
sidebar_label: Conditional
description: The Conditional card displays another card based on conditions.
---

The Conditional card displays another card based on conditions.

Note: if there are multiple conditions there will be treated as an 'and' condition. This means that for the card to show, _all_ conditions must be met.

To add the Conditional card to your user interface, click the menu (three dots at the top right of the screen) and then **Edit Dashboard**. Click the **Add Card** button in the bottom right corner and select from the card picker. Note that while editing the dashboard, the card will always be shown, so be sure to exit editing mode to test the conditions.

Most options for this card can be configured via the user interface.

## YAML Configuration

The following YAML options are available when you use YAML mode or just prefer to use YAML in the Code Editor in the UI.

{% configuration %}
type:
  required: true
  description: conditional
  type: string
conditions:
  required: true
  description: List of conditions to check. See [available conditions](/dashboards/conditional/#card-conditions).
  type: list
card:
  required: true
  description: Card to display if all conditions match.
  type: map
{% endconfiguration %}

## Examples

```yaml
type: conditional
conditions:
  - condition: state
    entity: light.bed_light
    state: "on"
  - condition: state
    entity: light.bed_light
    state_not: "off"
  - condition: user
    users:
      - 581fca7fdc014b8b894519cc531f9a04
card:
  type: entities
  entities:
    - device_tracker.demo_paulus
    - cover.kitchen_window
    - group.kitchen
    - lock.kitchen_door
    - light.bed_light
```

## Card conditions

### State

```yaml
condition: "state"
entity: climate.thermostat
state: heat
```

```yaml
condition: "state"
entity: climate.thermostat
state_not: "off"
```

Tests if an entity has a specified state.

{% configuration %}
condition:
  required: true
  description: "`state`"
  type: string
entity:
  required: true
  description: Entity ID.
  type: string
state:
  required: false
  description: Entity state is equal to this value.*
  type: string
state_not:
  required: false
  description: Entity state is unequal to this value.*
  type: string
{% endconfiguration %}

*one is required (`state` or `state_not`)

### Numeric State

Tests if an entity state matches the thresholds.

```yaml
condition: "numeric_state"
entity: sensor.outside_temperature
above: 10
below: 20
```

{% configuration %}
condition:
  required: true
  description: "`numeric_state`"
  type: string
entity:
  required: true
  description: Entity ID.
  type: string
above:
  required: false
  description: Entity state is above this value.*
  type: string
below:
  required: false
  description: Entity state is below to this value.*
  type: string
{% endconfiguration %}

*at least one is required (`above` or `below`)

### Screen

Specify the visibility of the card per screen size. Some screen size presets are available in the UI but you can use any CSS media query you want in YAML.

```yaml
condition: screen
media_query: "(min-width: 1280px)"
```

{% configuration %}
condition:
  required: true
  description: "`screen`"
  type: string
media_query:
  required: true
  description: Media query to check which screen size are allowed to display the card.
  type: string
{% endconfiguration %}

### User

Specify the visibility of the card per user.

```yaml
condition: "user"
users:
  - 581fca7fdc014b8b894519cc531f9a04
```

{% configuration %}
condition:
  required: true
  description: "`user`"
  type: string
users:
  required: true
  description: User ID that can see the card (unique hex value found on the Users configuration page).
  type: list
{% endconfiguration %}
