# Design

## What the card does

`config-template-card` wraps one card, row, or picture-elements element. On every render it deep-clones the wrapped configuration, walks it, replaces every string containing `${` with the result of evaluating that string as JavaScript, and asks Home Assistant's card helpers to create a fresh element from the result. The `entities` list names the entities whose state changes should trigger that re-evaluation.

## Template evaluation

Every template string is evaluated by a function built with `new Function`. The function's parameters are `hass`, `states`, `user`, `vars`, one parameter per named variable, and finally `__code`; its body is `return eval(__code)`. Two properties of that shape matter.

The `eval` runs inside the function scope, so the named parameters are visible to the user's code by name, which is how a template can write `best_model` instead of `vars.best_model`. The `eval` returns the completion value of the last statement, so both a bare expression (`states['sun.sun'].state`) and a multi-statement block (`let r = []; LIGHTS.forEach(e => r.push(e)); r;`) produce a value. Upstream's beta used `return (${expression})`, which only accepts an expression; that is why 2.0.0-b1 broke every multi-statement variable (upstream issues 191 and 192).

A value that is exactly one `${...}` with no further `${` inside is evaluated bare so its JavaScript type survives: numbers stay numbers, arrays stay arrays, which is what a templated `entities` list or a gauge `max` relies on. Any other string containing `${` is wrapped in backticks and evaluated as a JavaScript template literal, so `sensor.${model}_forecast` interpolates. Backticks inside the string are escaped first. This restores the behaviour of upstream's 1.3.7 line, which the Lit 3 rebuild lost (upstream issue 190).

Variables are evaluated in order: dashboard-level `config_template_card_vars` first, then the card's own `variables`. A card variable can therefore reference a dashboard variable, and a card definition with the same name as a dashboard definition wins.

## When the card re-renders

`shouldUpdate` returns true when the configuration changed, when the card helpers arrived, or when any entity named in `entities` differs between the previous and current `hass` snapshot (state object identity, then `state`, `last_changed`, `last_updated`). Otherwise it returns false and the wrapped element keeps rendering with the `hass` it was created with.

The helpers check is the fix for upstream issue 201. `loadCardHelpers` is asynchronous and stores the resolved helpers in a reactive `_helpers` field. When Home Assistant delivers the first `hass` before the helpers resolve, `render` returns an empty template, and the later `_helpers` change must be allowed through `shouldUpdate` or the card stays blank until a watched entity changes. Home Assistant 2026.6 made that ordering common on tab resume, which is why the report count rose then.

`entities` may also be one template string that evaluates to an array; `shouldUpdate` expands it before comparing.

## What the card cannot do

Because `render` returns a live element created by the helpers rather than a Lit template, Lit replaces the whole subtree each time `render` runs. The wrapped card is destroyed and rebuilt on every watched change, and between changes it receives no fresh `hass`. Upstream issues 112, 144, 174, and 45 all follow from that. A fix means caching the created element, pushing `hass` into it when the evaluated configuration is unchanged, and recreating it only when the configuration differs. That is a redesign, recorded in backlog.md.

## The wrapped card fills its cell

The wrapper `div` carries `height: 100%` so a wrapped card inside a horizontal stack or grid stretches to the row height like its siblings (upstream pull request 158).
