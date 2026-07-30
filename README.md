# EV-ADED Solar Simulator

Interactive engineering simulator for a van-mounted solar array with deployable sun-tracking wings, a roof that can be flat / tracking / fixed-tilt, fore/aft slide-out panels, actuator load analysis, wind loading, and inter-array self-shading. Built for the EV-ADED mobile solar platform, but flexible enough to model any roof or roof-plus-wing setup.

**Live demo:** https://madudek.github.io/Evaded-Solar-Simulator/

![Simulator screenshot](screenshot.png)

## What it models

A delivery-van solar system with three independent surfaces and seven strings (defaults shown — counts, wattages, dimensions, and presence are all adjustable in the UI):

| Surface | Stationary panels | Slide-outs | Motion |
|---|---|---|---|
| Roof | 4 x 550 W | 2 x 550 W | Flat, single-axis tracking, or fixed seasonal tilt; slide-outs travel fore/aft, front over the cab |
| Driver wing | 3 x 440 W portrait | 2 x 440 W | Hinged at the roofline, 15-165 deg sun tracking; slide-outs extend fore/aft in the same row |
| Passenger wing | 3 x 440 W portrait | 2 x 440 W | Mirror of the driver wing, independently tracked |

Default total: 7,700 W across 14 panels.

## Features

### Pick your hardware
- **Installed** checkboxes turn each array on or off — model a roof-only build by unchecking both wings, a wings-only rig by unchecking the roof, or any combination. Disabled arrays drop out of power, the scene, and the load/wind calcs.
- Per-surface panel counts and wattages; slide-outs optional per surface via checkboxes.
- **Van model** (BrightDrop 400 / 600) and a **wheelbase** field set the drawn length and proportions; the roof and wing layouts stretch to match. Length is visual context only — output is panel-count based, so it doesn't change any numbers.
- **Panel size** inputs, separately for roof (across-van x along-van) and wing (depth x width). Wing dimensions are physical: the depth is the actuator lever arm, so editing it flows through the wing geometry, the gravity and wind centroid, actuator force, the best-fit arm solver, and the wind area. Roof dimensions set the drawn roof footprint (roof output is wattage-based and the roof carries no wind or actuator load).
- Van, wheelbase, and panel-size controls live in the header strip to keep the main dashboard compact.

### Operating modes
- **Track** — wings follow the sun through 15-165 deg, each at its own angle; roof tracks too if set to tracking
- **Awning** — wings locked flat at 90 deg, forming a 180 deg plane with the roof
- **Stow** — driving configuration with interlock sequencing (wings deploy before slide-outs extend; slide-outs retract before wings fold)
- **Run day** animation sweeping sunrise to sunset with every readout live

### Roof modes
A dropdown sets the roof to one of three states:
- **Flat** — panels lie in the roof plane.
- **Single-axis tracking** — an ideal tracker tilting about the fore/aft axis; the gain is concentrated in the morning and evening shoulders and is near zero at midday, when a flat roof is already optimal.
- **Fixed tilt** — a held angle, with **Summer** and **Winter** preset buttons (latitude -/+ ~12 deg) or any angle you type. Lets you compare a parked seasonal tilt against flat: with the sun low, a steep winter tilt can nearly triple flat-roof yield, while summer wants it shallow or flat. The roof tilts in the scene and the energy reflects it.

### Self-shading
- A cross-plane ray-trace computes how much each array is shadowed by the others and derates power accordingly, with shaded panels dimmed in the scene and a live self-shading readout. Toggle it with the **Account for self-shading** checkbox to compare against an idealized no-shadow upper bound.
- Reveals a real design tradeoff: a tracking roof tilts up to face a low sun and shadows the up-pointing wing behind it — exactly during the shoulder hours roof tracking is meant to help — so the gains partially cancel on a co-tracking system.

### Sun model
- Any site: city, zip, or raw `lat, lon` (coordinates work offline; city/zip uses the free Open-Meteo geocoder)
- Any parking heading with full azimuth geometry, including the out-of-plane cosine loss a single-axis tracker can't recover
- Seasons: winter solstice / equinox / summer solstice, plus **Custom** — enter daylight hours for any date and the model solves the matching declination
- Live sun elevation in the scene; instantaneous W and daily kWh for tracking vs. awning-flat vs. stowed, with the full daily curve and peak

### Actuator analysis
- Linkage load model: F = W*r*L(theta)/(a*b) per wing across 2 actuators — gravity's sin(theta) cancels against the linkage arm, so force rises with wing angle and peaks at the over-roof end stop
- Enter actuator retracted/extended lengths, rating, and mounting distances from the hinge; **best-fit** solves the mounting geometry for the full stroke with one click
- Reach validation flags arms that can't cover 15-165 deg; per-wing weights, live load, and the true worst-case load with its angle
- The gravity/wind lever follows the entered wing panel depth (half-depth centroid), so resizing the wing updates the loads

### Wind loading
- Wind speed plus 8 relative directions (driver/passenger/nose/rear and four quarters) with correct vector physics — axial wind carries no cross pressure, quarters carry half
- **Live wind** from Open-Meteo at the set site, using gust speed, auto-resolved against the van heading
- Per-wing wind force with plain-language direction, load-reversal marker (* = actuator in tension), and a color-coded **safe wind limit** at the current angles
- Key result: wind load through the linkage grows as cot(tilt), so steep dawn/dusk angles are the danger case and a wind harmless when flat can exceed the rating at the end stops — stow flat in storms

### Hinge loads
- Planar reaction carried by the hinge line, balancing wing weight, wind drag, and the actuator pair at the worst wind direction — the load your hinge and its fasteners actually see
- Three readouts: the live value at the current angle, and two peak-over-travel envelopes (dead-load, and at the current wind) so you can read the wind amplification directly
- Continuous vs. sectioned hinge: enter a capacity (lb per inch for a continuous hinge, or lb per leaf for N sections) and it distributes the load accordingly
- **Stow above** trigger folds the hinge into the stow decision: it reports the wind speed where either the actuator rating or the hinge capacity is reached first over the travel, and which one governs — the number to drive an auto-stow rule, not just the actuator limit
- Flat-panel uplift isn't modeled (matches the actuator wind convention); steep angles govern

### Display
- Auto-scales to fit the dashboard on one screen across laptops, monitors, and tablets; stacks to a single column on phones.

## How to use

1. **Installed** — check the arrays you actually have.
2. **Van + dimensions** (header strip) — pick BrightDrop 400/600, set the wheelbase, and enter your real roof and wing panel sizes so the geometry and wing loads match your build.
3. **Site + nose heading** — nose N/S aims the wings east-west for the full tracking sweep; nose E/W largely wastes the wings (but is where fixed roof tilt shines), and the tool quantifies the cost.
4. **Configure** — panel counts/wattages, wing weights, actuator specs and mounting (or apply the suggested best fit), roof flat/tracking/fixed-tilt, self-shading on/off.
5. **Operate** — Track / Awning / Stow, slide-out checkboxes, time slider or Run day.
6. **Check wind and hinge** — fetch live or enter a worst case, set your hinge type and capacity, and read the safe-wind limit and the governing stow-above trigger before leaving the array deployed.

## Physics summary

- Sun position: site latitude, declination (+/-23.44 deg by season or solved from custom day length), hour angle, full azimuth, solar time
- Irradiance: clear sky, air-mass sin(elev)^0.3, 0.85 system derate; flat surface proportional to sin(elevation), single-axis tracked to sqrt(E^2+U^2) of the cross-plane sun projection, fixed tilt to the cosine of incidence on the held panel
- Self-shading: 2D cross-plane ray-trace, linear area derate (real series-string losses can exceed this, so shaded figures are optimistic)
- Actuator: gravity torque W*r*sin(theta) over effective arm a*b*sin(theta)/L(theta); wind torque q*A*C*d at C = 1.3*cos(tilt) with +/-0.15 gust floor, force at the panel mid-depth centroid (half the entered wing depth); fore/aft slide-outs add area and weight but no lever
- Hinge: planar force balance at the hinge line — wing weight, wind drag, and the actuator reaction summed and resolved at the worst wind direction; distributed over the fixed-panel span for a per-inch (continuous) or per-leaf (sectioned) load
- Conservatisms: gas-strut assist, van shielding, and dynamic gust factors excluded; outputs are clear-sky upper bounds; loads are static requirements

## Running locally

No build step. Download `index.html` and open it in any browser. Everything except city/zip geocoding and live wind works offline.

## License

MIT — see [LICENSE](LICENSE).
