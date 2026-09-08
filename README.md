# Config Template Card

📝 Templatable Configuration Card

[![GitHub Release][releases-shield]][releases]
[![License][license-shield]](LICENSE.md)
[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg?style=for-the-badge)](https://github.com/hacs/integration)

![Project Maintenance][maintenance-shield]
[![GitHub Activity][commits-shield]][commits]

[![Discord][discord-shield]][discord]
[![Community Forum][forum-shield]][forum]

This card is for [Lovelace](https://www.home-assistant.io/lovelace) on [Home Assistant](https://www.home-assistant.io/) that allows you to use pretty much any valid Javascript on the hass object in your configuration

## About this fork

This repository is a maintained fork of [iantrich/config-template-card](https://github.com/iantrich/config-template-card). It starts from upstream's Lit 3 rebuild (2.0.0-b2) and adds the fixes that upstream left in open pull requests: the card no longer disappears when Home Assistant delivers the card helpers after the first render (upstream #201), templates embedded inside longer strings work again (#190), multi-statement `${ }` blocks and templated `entities` lists work again (#191, #192), and the wrapped card fills its grid or stack cell (#158). The reasoning behind each change is in [docs/README.md](docs/README.md).

## Minimum Home Assistant version

The fork is built and tested against Home Assistant 2026.9. `hacs.json` states that floor. Older releases are not tested; the card uses only `window.loadCardHelpers`, so it may work on them, but that is unverified.

## Installation

Add `https://github.com/trooperthorn/ha_card_config-template` to HACS as a custom repository of type Dashboard, then download it. If the upstream card is already installed, remove it first so both do not register `config-template-card`. HACS serves the file at `/hacsfiles/ha_card_config-template/config-template-card.js`. Without HACS, copy `dist/config-template-card.js` to `<config>/www/` and register it as a module resource.

## Options

| Name      | Type   | Requirement  | Description                                                                                                      |
| --------- | ------ | ------------ | ---------------------------------------------------------------------------------------------------------------- |
| type      | string | **Required** | `custom:config-template-card`                                                                                    |
| entities  | list   | **Required** | List of entity strings that should be watched for updates. Templates can be used here                            |
| variables | list   | **Optional** | List of variables, which can be templates, that can be used in your `config` and indexed using `vars` or by name |
| card      | object | **Optional** | Card configuration. (A card, row, or element configuaration must be provided)                                    |
| row       | object | **Optional** | Row configuration. (A card, row, or element configuaration must be provided)                                     |
| element   | object | **Optional** | Element configuration. (A card, row, or element configuaration must be provided)                                 |
| style     | object | **Optional** | Style configuration.                                                                                             |

### Available variables for templating

| Variable    | Description                                                                                                                                                                                                                                                                                                                                                                                           |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `this.hass` | The [hass](https://developers.home-assistant.io/docs/frontend/data/) object                                                                                                                                                                                                                                                                                                                           |
| `states`    | The [states](https://developers.home-assistant.io/docs/frontend/data/#hassstates) object                                                                                                                                                                                                                                                                                                              |
| `user`      | The [user](https://developers.home-assistant.io/docs/frontend/data/#hassuser) object                                                                                                                                                                                                                                                                                                                  |
| `vars`      | Defined by `variables` configuration and accessible in your templates to help clean them up. If `variables` in the configuration is a yaml list, then `vars` is an array starting at the 0th index as your firstly defined variable. If `variables` is an object in the configuration, then `vars` is a string-indexed map and you can also access the variables by name without using `vars` at all. |
## Examples

```yaml
type: 'custom:config-template-card'
variables:
  LIGHT_STATE: states['light.bed_light'].state
  GARAGE_STATE: states['cover.garage_door'].state
entities:
  - light.bed_light
  - cover.garage_door
  - alarm_control_panel.alarm
  - climate.ecobee
card:
  type: "${LIGHT_STATE === 'on' ? 'glance' : 'entities'}"
  entities:
    - entity: alarm_control_panel.alarm
      name: "${GARAGE_STATE === 'open' && states['alarm_control_panel.alarm'].state === 'armed_home' ? 'Close the garage!' : ''}"
    - entity: binary_sensor.basement_floor_wet
    - entity: climate.ecobee
      name: "${states['climate.ecobee'].attributes.current_temperature > 22 ? 'Cozy' : 'Too Hot/Cold'}"
    - entity: cover.garage_door
    - entity: "${LIGHT_STATE === 'on' ? 'light.bed_light' : 'climate.ecobee'}"
      icon: "${GARAGE_STATE === 'open' ? 'mdi:hotel' : '' }"
```

What this example does:

- Uses `variables` as named values (`LIGHT_STATE`, `GARAGE_STATE`) so templates are easier to read.
- Uses `entities` as the watch list; when any listed entity changes, the card re-evaluates templates.
- Uses `card` as the render target, and even templates the `card.type` itself (`glance` vs `entities`).
- Templates specific row properties (`name`, `entity`, `icon`) so content changes live based on state.
- Demonstrates that nearly any string field inside the target card config can be templated.

### Templated entities example

```yaml
type: 'custom:config-template-card'
variables:
  - states['sensor.light']
entities:
  - '${vars[0].entity_id}'
card:
  type: light
  entity: '${vars[0].entity_id}'
  name: "${vars[0].state === 'on' ? 'Light On' : 'Light Off'}"
```

What this example does:

- Uses `variables` in array mode, so values are referenced by index (`vars[0]`).
- Uses a templated item in `entities`, meaning the watched entity itself is dynamic.
- Renders a `card` target (`light` card) whose `entity` and `name` come from templates.
- Shows a common pattern where one sensor (or helper) points to the active entity to display.

### Picture-elements card example

```yaml
type: picture-elements
image: https://upload.wikimedia.org/wikipedia/commons/4/49/Koala_climbing_tree.jpg
elements:
  - type: 'custom:config-template-card'
    variables:
      - states['light.bed_light'].state
    entities:
      - light.bed_light
      - sensor.light_icon_color
    element:
      type: icon
      icon: "${vars[0] === 'on' ? 'mdi:home' : 'mdi:circle'}"
      style:
        '--paper-item-icon-color': '${ states[''sensor.light_icon_color''].state }'
    style:
      top: 47%
      left: 75%
```

What this example does:

- Embeds `custom:config-template-card` inside a native `picture-elements` card.
- Uses `element` (not `card`/`row`) as the render target, because picture-elements expects elements.
- Uses wrapper-level `style` (`top`, `left`) to position the template card in the image canvas.
- Uses `element.style` to style the inner rendered element (icon color), independent of wrapper placement.
- Uses `entities` to watch both the light state and color source sensor for live icon updates.

Note: The `style` object on the element configuration is applied to the element itself, the `style` object on the `config-template-card` is applied to the surrounding card, both can contain templated values. For example, in order to place the card properly, the `top` and `left` attributes must always be configured on the `config-template-card`.

### Entities card example

```yaml
type: entities
entities:
  - type: 'custom:config-template-card'
    variables:
      - states['light.bed_light'].state
    entities:
      - light.bed_light
    row:
      type: section
      label: "${vars[0] === 'on' ? 'Light On' : 'Light Off'}"
  - entity: light.bed_light
```

What this example does:

- Embeds `custom:config-template-card` as an item inside a native `entities` card list.
- Uses `row` as the render target, which is the correct mode for rows inside entities cards.
- Uses one watched entity and one array variable to keep logic minimal.
- Dynamically changes the section label so the row text reflects live entity state.

## Defining global functions in variables

If you find yourself having to rewrite the same logic in multiple locations, you can define global methods inside Config Template Card's variables, which can be called anywhere within the scope of the card:

```yaml
type: 'custom:config-template-card'
  variables:
    setTempMessage: |
      temp => {
        if (temp <= 19) {
            return 'Quick, get a blanket!';
        }
        else if (temp >= 20 && temp <= 22) {
          return 'Cozy!';
        }
        return 'It's getting hot in here...';
      }
    currentTemp: states['climate.ecobee'].attributes.current_temperature
  entities:
    - climate.ecobee
  card:
    type: entities
    entities:
      - entity: climate.ecobee
        name: '${ setTempMessage(currentTemp) }'
````

What this example does:

- Uses object-style `variables` to define both a reusable function (`setTempMessage`) and data (`currentTemp`).
- Demonstrates that templates can call custom functions, not just simple expressions.
- Uses `entities` to watch only what the template depends on (`climate.ecobee`).
- Keeps card config clean by moving conditional text logic into a reusable variable function.

## Dashboard wide variables

If you need to use the same variable in multiple cards, then instead of defining it in each card's `variables` you can do that once for the entire dashboard.

```yaml
title: My dashboard

config_template_card_vars:
  - states['sensor.light'].state

views:
```

What this example does:

- Defines `config_template_card_vars` at dashboard root so many cards can reuse shared variables.
- Avoids repeating identical variable expressions in every individual card config.
- Supports both array and object forms, same as local `variables`.
- When both dashboard and local variables are arrays, dashboard entries come first in `vars`.

Both arrays and objects are supported, just like in card's local variables. It is allowed to mix the two types, i.e. use an array in dashboard variables and an object in card variables, or the other way around. If both definitions are arrays, then dashboard variables are put first in `vars`. In the mixed mode, `vars` have array indices and as well as variable names.

### Note: All templates must be enclosed by `${}`

[Troubleshooting](https://github.com/thomasloven/hass-config/wiki/Lovelace-Plugins)

## Developers

Clone the repository, run `corepack enable`, then `yarn install --immutable` and `yarn build`. The build runs lint, typecheck, the vitest suite, and rollup in that order. `dist/config-template-card.js` is committed; CI fails when it differs from a fresh build, so rebuild and commit it with every source change.

## Versioning and releases

Versions are CalVer `YYYY.MM.DD.N`; tags carry the bare number, matching the tag style upstream used. The root `VERSION` file and `CARD_VERSION` in `src/const.ts` must agree, and `.release.json` names both. `package.json` stays at an inert `0.0.0` because Yarn requires SemVer there.

A merge to `main` is the only release path. `Release` runs on every push to `main`: it validates the version, rebuilds the card and checks the committed dist matches, creates the tag, drafts the release with `dist/config-template-card.js` and its SHA-256 attached, and publishes it. `Prepare release` runs after every successful `Release` and, when release-bearing files changed since the last tag, writes the next version, rebuilds `dist`, and opens an auto-merging PR through the release GitHub App (variable `RELEASE_AUTOMATION_CLIENT_ID`, secret `RELEASE_AUTOMATION_PRIVATE_KEY`). Without those credentials, run `python scripts/set_version.py --next-from-tags`, `yarn rollup`, commit both, and open a PR; the merge publishes.

[commits-shield]: https://img.shields.io/github/commit-activity/y/trooperthorn/ha_card_config-template.svg?style=for-the-badge
[commits]: https://github.com/trooperthorn/ha_card_config-template/commits/main
[discord]: https://discord.gg/Qa5fW2R
[discord-shield]: https://img.shields.io/discord/330944238910963714.svg?style=for-the-badge
[forum-shield]: https://img.shields.io/badge/community-forum-brightgreen.svg?style=for-the-badge
[forum]: https://community.home-assistant.io/t/100-templatable-lovelace-configuration-card/105241
[license-shield]: https://img.shields.io/github/license/trooperthorn/ha_card_config-template.svg?style=for-the-badge
[maintenance-shield]: https://img.shields.io/badge/maintainer-trooperthorn-blue.svg?style=for-the-badge
[releases-shield]: https://img.shields.io/github/release/trooperthorn/ha_card_config-template.svg?style=for-the-badge
[releases]: https://github.com/trooperthorn/ha_card_config-template/releases
