# SPEC-001: Automotive dashboard context

## Objective
Let users select a simulated dashboard item and pass the correct context to JourneyHelp.

## Requirements
- Display tire pressure, engine indicator, rear defogger, and oil pressure.
- Exactly one item is selected at initialization.
- Selection updates without reload.
- Every request carries product, model year, page, locale, and current item.
- The demo states it is simulated and has no real vehicle connection.

## Scenarios
- Default: tire pressure is selected and supplied as context.
- Change: selecting rear defogger updates context to `rear-defogger`.
- Submit: request contains the current item.
- Reselect: subsequent request uses the latest selection.
