# EV-ADED Solar Simulator

Interactive engineering simulator for a van-mounted, sun-tracking solar array with deployable wings, fore/aft slide-out panels, actuator load analysis, and wind loading. Built for the EV-ADED mobile solar platform.

**Live demo:** https://madudek.github.io/Evaded-Solar-Simulator/

![Simulator screenshot](screenshot.png)

## What it models

A delivery-van solar system with three surfaces and seven independent strings (defaults shown — counts and wattages are adjustable in the UI):

| Surface | Stationary panels | Slide-outs | Motion |
|---|---|---|---|
| Roof | 4 × 550 W, fixed flat | 2 × 550 W | Slide fore/aft along the van, front extending over the cab |
| Driver wing | 3 × 440 W portrait | 2 × 440 W | Hinged at the roofline, 15°–165° single-axis sun tracking; slide-outs extend fore/aft in the same parallel row |
| Passenger wing | 3 × 440 W portrait | 2 × 440 W | Mirror of the driver wing, independently tracked |

Default total: 7,700 W across 14 panels.

## Features

### Operating modes
- **Track** — both wings follow the sun through 15°–165°, each at its own angle
- **Awning** — wings locked flat at 90°, forming one 180° plane with the roof
- **Stow** — driving configuration with interlock sequencing (wings deploy before slide-outs extend; slide-outs retract before wings fold)
- Independent slide-out control for roof, driver wing, and passenger wing
- **Run day** animation sweeping sunrise to sunset with every readout live

### Sun model
- Any site: enter a city, zip code, or raw `lat, lon` coordinates (coordinates work fully offline; city/zip lookup uses the free Open-Meteo geocoder)
- Any parking heading (N through NW) with full azimuth geometry — tracked output includes the out-of-plane cosine loss along the van's axis that a single-axis tracker cannot recover
- Seasons: winter solstice / equinox / summer solstice, plus **Custom** — enter the daylight hours for any date and the model solves the matching solar declination at your latitude
- Live sun elevation readout in the scene; instantaneous W and daily kWh for tracking vs. awning-flat vs. stowed, with the full daily curve and expected peak

### Actuator analysis
- Full linkage load model: F = W·r·L(θ)/(a·b) per wing across 2 actuators — the sinθ in gravity torque cancels against the linkage's effective arm, so force rises monotonically with wing angle and peaks at the over-roof end stop, not flat
- Enter your actuator specs: retracted length, extended length, load rating, and the mounting distances from the hinge on the wall and on the panel
- **Best-fit mounting suggestion**: solves L(15°) = retracted and L(165°) = extended exactly, using the full stroke, with one-click apply
- Reach validation: turns red when the entered mounting distances can't cover 15°–165° with the entered actuator lengths
- Per-wing weights, live load, and the true maximum load with the angle at which it occurs

### Wind loading
- Wind speed plus 8 relative directions: driver side, passenger side, van nose, van rear, and four quartering winds — with correct vector physics (axial wind carries no cross-plane pressure; quarters carry half)
- **Live wind**: fetches current conditions (sustained + gust) from Open-Meteo at the set site and auto-resolves the compass direction against the van's nose heading; change the heading and the effective cross-wind recomputes instantly
- Per-wing wind force with plain-language direction (pushes wing closed / lifts wing / gust buffet), load-reversal marker (* = actuator in tension), and a color-coded **safe wind limit** at the current wing angles
- Key engineering result the tool demonstrates: wind load through the linkage grows as cot(tilt), so steep dawn/dusk tracking angles are the danger case — a 30 mph wind that is harmless flat can exceed the actuator rating at the end stops. Stow flat in storms.

## How to use

1. **Set your site** (city, zip, or coordinates) and the **nose heading**. Nose N/S points the wings east–west for the full tracking sweep; nose E/W largely wastes them — the tool quantifies the cost.
2. **Configure hardware**: panel counts and wattages per surface, wing weights, actuator specs and mounting distances (or apply the suggested best fit).
3. **Operate**: Track / Awning / Stow, slide-out checkboxes (they queue while stowed and extend automatically after deploy), time slider or Run day.
4. **Check wind**: fetch live conditions or enter a worst case, and read the safe-wind limit before leaving the array deployed.

## Physics summary

- Sun position: site latitude, declination (±23.44° by season or solved from custom day length), hour angle, full azimuth, solar time
- Irradiance: clear sky, air-mass approximation `sin(elev)^0.3`, 0.85 system derate
- Actuator: gravity torque W·r·sinθ over effective arm a·b·sinθ/L(θ); wind torque q·A·C·d at C = 1.3·cos(tilt) with ±0.15 gust floor, force at the 34.7″ panel centroid — fore/aft slide-outs add area and weight along the hinge line but no lever depth, so they don't change actuator torque
- Conservatisms: gas-strut assist, van shielding, and dynamic gust factors excluded; outputs are clear-sky upper bounds; loads are static requirements

## Running locally

No build step. Download `index.html` and open it in any browser. Everything except city/zip geocoding and live wind works offline.

## License

MIT — see [LICENSE](LICENSE).
