# Dashboard Contracts

Read this file only when changing energy/carbon charts, Sankeys, panel layouts, themes, responsive behaviour or accessibility.

## Domestic energy-and-carbon chart: non-negotiable visual contract

The large “Energy flow and carbon” chart is a **comparative footprint**, not an energy-accounting or composition chart. Keep it as overlaid line/area profiles:

- Above zero: independently overlaid gas burned, Emporia domestic electricity, and solar generation. EV charging and battery charging may appear as separate overlays for context, but must not turn the chart into a stack.
- Below zero: the actual and unmitigated carbon footprints. “Unmitigated” is gas combustion plus the counterfactual electricity carbon had Emporia domestic use (including EV, excluding battery charge) **and exported solar** come from the grid at the intensity at the time of consumption/export. Actual is physical: imported-electricity carbon plus gas combustion. Each positive `sensor.feed_in_energy_total` delta is valued at the contemporaneous intensity in a separate ledger and added only to Unmitigated; it represents the grid generation avoided elsewhere, not a subtraction from emissions actually caused by the house.
- Gas is an exception to the usual live-meter approach: its Octopus source exposes timestamped half-hourly `charges`. Treat those intervals—not the changing cumulative state—as canonical for gas profiles and its physical `0.185 kgCO₂e/kWh` combustion contribution. Gas carbon belongs in both Actual and Unmitigated at the same interval.
- Do **not** use stacked columns, Sankey-style conservation logic, or a component stack in this chart. Those belong in the Sankey visualisation.
- If a requested breakdown conflicts with the overlay model, retain the main profiles and add the component as an independent line instead of changing the chart's semantic purpose.
- When showing the optional grid → EV → battery composition, imply the stack with cumulative, back-to-front lines: grid; grid + EV; grid + EV + battery. Use progressively darker/stronger lines behind the main overlays—never columns or an actual chart stack.

## Sankey: energy integrity and decision-size detail

The Sankey is the **energy-accounting** visualisation. It should make the physical balance legible, rather than concealing discrepancies for a tidier layout.

- Treat an unlabeled gap, unexpected node height, or unbalanced flow as a testable data/logic defect. Trace it to source entities and arithmetic before removing or relabelling it.
- Electricity balance: `property electricity = grid import + solar production + battery discharge`. `home energy = property electricity - solar export`. Battery charge is one outgoing sink from Home; never add it to the Home node as well as linking it out, or it will be double counted.
- Small residuals may represent conversion losses, measurement boundaries, or rounding, but only describe them as such once the arithmetic has been validated. Do not turn a large unexplained discrepancy into an “unmetered” visual placeholder.
- Both current Sankeys end at the compact fifth-column destinations: gas, battery charge, EV, cooking/washing, computers, entertainment, lighting, and other. Do not restore circuit-level column six unless it supports a specific decision that the compact view cannot.
- Prefer “decision-size detail”: show the level at which a person can make a useful decision, and place forensic circuit detail in a separate inspection view rather than making the overview dense.

## Dashboard visual usability and colour accessibility

Optimise for **human comprehensibility** across phone, tablet/Kindle, desktop browser, and HA native-app views. Colour is useful but must never be the only carrier of meaning.

- Use a stable Okabe–Ito semantic palette where a card accepts explicit hex colours: grid `#0072B2`, low-carbon grid `#009E73`, solar `#E69F00`, gas/combustion `#D55E00`, battery/storage `#000000`, forecast/counterfactual `#CC79A7`, and passive reference `#999999`. Confirm custom-card support for hex colours before relying on it.
- Avoid Okabe–Ito yellow `#F0E442` for text or thin lines on a light background; it is suitable only for large outlined fills. Maintain strong text/background contrast in both light and dark themes.
- Pair every semantic colour with a stable redundant cue: direct label, distinctive icon, direction arrow, line style, fill opacity, or fixed position. A grayscale or colour-blind rendering must still be interpretable without its legend.
- Do not use red/green alone for normal/warning/critical status. Pair status with plain-language text and an icon; reserve urgent colours for genuinely urgent states.
- For charts, use no more colours than the semantic categories require. Use solid/dashed/dotted strokes and opacity to distinguish actual, forecast, counterfactual, and cumulative series. Do not add a hue merely to decorate a series.
- Design responsive information hierarchy rather than merely shrinking a desktop layout: phone prioritises current state and safe actions; tablet and Kindle show overview and trends; desktop can add explanatory/inspection detail. Keep controls comfortably tappable and avoid scroll-dependent critical information.
- Before settling a significant visual change, inspect it at the smallest expected width, in light/dark contexts, and with grayscale plus common red/green colour-vision-deficiency simulation. A dashboard should be usable without a legend or perfect colour perception.
- `themes/okabe_ito.yaml` is the tracked panel theme. Keep its semantic tokens as the source of truth rather than adding arbitrary chart colours.
- `ui-panel.yaml` defines the labelled `kiosk_navigation` button-card template. All four panel views use it as their in-dashboard navigation; preserve it so the Kindle browser can remain in kiosk mode without sacrificing navigation.
- Panel views use responsive named grid areas: four columns for landscape Kindle/desktop, two columns for tablet widths, and one column on a phone. Retain that reflow model rather than restoring fixed pixel-column layouts.
- `layout-card` applies the **first** matching media-query rule. Put the narrowest phone rule before broader tablet rules, otherwise a phone silently receives the compressed tablet layout. Below 801px, hide in-panel kiosk navigation and use dedicated mobile cards for groups that cannot remain legible when compressed. iPads use the same chrome-navigated layout via `(max-aspect-ratio: 159/100)`; the Kindle's wider 16:10 kiosk display retains its in-panel navigation.
- The Power panel is an **operational** view. Its primary visual is live inverter routing—solar/grid/battery/house with explicit arrows and kW—not a daily Sankey. Daily Sankey detail belongs in Sandpit. Its carbon chart plots daily cumulative actual and unmitigated CO₂ directly; never derive a displayed hourly carbon flow from a total that mixes a lifetime accumulator with a daily-resetting gas meter.
