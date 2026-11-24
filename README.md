# Line Pilot 737

A flight simulator that runs entirely in the browser. You fly a Boeing 737-800 (and 24 other aircraft types) between real airports over streamed satellite imagery and terrain, with live weather, AI traffic, gate-to-gate ground operations, clickable 3D flight decks, a flight school and an airline career. A separate Vietnam-era military mode adds combat aircraft and helicopters with their own flight models, tutorials and a 1965–72 campaign.

It is written from scratch in plain JavaScript and WebGL 2, with no game engine, no framework and no third-party libraries, and ships as a single `index.html`.

**Play it:** https://dbrokenai.github.io/line-pilot-737/

## What it does

**Civil flying**

- 175 real airports worldwide in an embedded database (runway ends, elevations, ILS, magnetic variation, time zones), with runway markings, approach lighting and PAPI drawn from that data.
- 25 flyable types: 737-800/900ER, 757-200 and 767-300 (passenger and freighter), 777-300ER/777F, 787-9, 747-400/747-8F, MD-11F, A320/A321, A330-300, A350-900, E175, CRJ900, ATR 72-600, Dash 8-400, Gulfstream G650ER, Citation CJ4, Cessna 172S, Pilatus PC-12 and C-130H. Each family has its own 3D flight deck and display suite.
- 737NG systems: autopilot and flight director, autothrottle, FMC with LNAV/VNAV, ILS and autoland, GPWS and warnings, V-speeds, and a working CDU, PFD, ND and overhead panel. Cockpit switches and knobs are picked with the mouse.
- Live weather: METARs fetched from NOAA and parsed in the client, plus forecasts; or preset weather and time of day.
- Gate-to-gate operations: pushback, taxi routing over real taxiway networks, ground ATC and parking at real stands.
- AI traffic on the runways in use, parked aircraft, en-route traffic with contrails, and TCAS.
- Game modes: flight school (a syllabus with a checkride), airline job (career with route unlocks, seniority and bidding, duty-limited trips, pay, reputation, achievements), free flight and landing practice.
- Random malfunctions with QRH-style checklists (engine fire, bird strike, gear problems, depressurization, unreliable airspeed, windshear and others).
- Landing scoring and a debrief after each flight, a logbook, crash investigation reports, and a scrubbable replay.
- Two-player flying over WebRTC, joined with a 6-letter code.
- Seven camera views including a passenger cabin interior, and a moving map.

**Vietnam 1965–72 mode**

- Flyable F-4 Phantom II, F-105 Thunderchief, A-1 Skyraider, UH-1 Huey and AH-1 Cobra, each with a 3D cockpit and gunsight.
- Separate flight models for jets, propeller aircraft and helicopters, including carrier operations.
- Weapons, ballistics, blast and fragmentation damage, AAA and SAM sites, ground units, and AI pilots (MiG-17, MiG-21, wingmen).
- A 17-mission chronological campaign over real places and airfields, a free-mission generator, per-aircraft training, ranks and medals, and three assist levels.

## How it is built

Everything is in `index.html` (about 49,000 lines): roughly 40 JavaScript modules as inline scripts, the airport, taxiway and stand databases, and the icons. The only external requests are for map tiles, weather, building data, web fonts and multiplayer signalling.

Notable engineering in the code:

- **Custom WebGL 2 renderer.** Camera-relative transforms and a logarithmic depth buffer keep precision stable from a cockpit switch to terrain hundreds of kilometres away. The high-detail path renders to a multisampled HDR framebuffer, then applies bloom, tone mapping and grading; cascaded shadow maps handle sun shadows. A low-detail mode and adaptive render resolution cover weaker GPUs.
- **Streamed real-world terrain.** A quadtree LOD system streams elevation tiles and satellite imagery, flattens terrain under runways, and falls back to procedural relief if tile servers are unreachable. Data providers fail over automatically (Esri to USGS imagery; AWS Terrain Tiles to USGS 3DEP), which required writing a small TIFF decoder (LZW and Deflate) in the client.
- **Map projection chosen per route.** The world origin moves to the middle of each route and uses a stereographic projection scaled so that distances are exact at both airports, with grid convergence corrected wherever a heading is shown.
- **6-DOF flight dynamics.** A rigid-body model with aerodynamic coefficients, an engine model, and landing gear simulated as tyre springs in series with gas/oil oleo struts, with antiskid, brake temperature and tyre slip. The 737 model is generalised so the other types are parameter sets over the same core, including turboprop and piston engine models and an Airbus fly-by-wire normal law.
- **Procedural aircraft and cockpits.** Aircraft are generated in code by a construction kit (lofted fuselages and wings, lathed nacelles, landing gear, liveries, LODs). Flight deck panels are drawn to canvas textures at real dimensions and mapped onto 3D geometry with ray-picked controls.
- **Real buildings.** Airport terminals and city buildings come from OpenStreetMap via Overpass. New York streams the city's building footprint dataset in three height bands, with an optional 1990s skyline.
- **Synthesized audio.** No audio files: engines, alerts and cabin sounds are generated with the Web Audio API, and ATC and callouts use speech synthesis.
- **Serverless multiplayer.** Peers connect directly over WebRTC. The handshake uses a public PeerJS broker, with copy-paste codes (deflate plus base64url) as a fallback, and a hand-written minimal MQTT-over-WebSocket client relays state through public brokers when a direct connection is blocked.
- **Persistence.** Career, school progress, logbook, settings and cached OSM data are stored in `localStorage` with versioned, corruption-tolerant saves.

## Running it

Use the hosted version above, or run it locally. There is no build step and nothing to install:

```
git clone https://github.com/DBrokenAI/line-pilot-737.git
cd line-pilot-737
python -m http.server 8000
```

Then open `http://localhost:8000/`. Any static file server works.

Requirements: a desktop browser with WebGL 2 (Chrome, Edge or Firefox) and an internet connection for terrain, imagery and live weather.

## Controls

Keyboard and mouse. Press **F1** in flight for the full list; the main ones for the airliners are:

| Key | Action |
| --- | --- |
| Arrow keys | Pitch and roll (Shift for fine control) |
| Q / E | Rudder; nose-wheel tiller at low speed |
| W / S | Thrust up / down (Shift+W TO/GA, Shift+S idle) |
| R | Reverse thrust on the ground |
| Space / P | Wheel brakes / parking brake |
| F / Shift+F | Flaps down / up one notch |
| G | Landing gear |
| / | Arm speedbrake |
| Home / End | Stabilizer trim |
| A / T | Autopilot / autothrottle |
| L / V / H | LNAV / VNAV / heading select |
| O | Approach mode |
| Y | Mouse yoke |
| 1–7 | Views: cockpit, chase, orbit, tower, wing, cabin, traffic |
| M | Map |
| X | Skip the cruise |
| [ / ] | Time speed |
| Enter | Read back ATC |

In the cockpit, hover a switch to see what it is, click it, or use the mouse wheel on knobs and levers. Right-drag looks around. The military mode has its own controls, taught in its per-aircraft tutorials.

## Status and limitations

- Desktop only; it is designed for keyboard and mouse.
- `index.html` is the published build. The individual source modules and the design notes referenced in code comments are not in this repository.
- It depends on free public services (tile servers, Overpass, weather APIs, PeerJS and MQTT brokers). If terrain or building servers are unreachable the sim falls back to procedural terrain and built-in landmarks; multiplayer needs at least one signalling path to be reachable.
- There are no automated tests in the repository.

## Data sources and credits

- Imagery: Esri World Imagery (Esri, Maxar, Earthstar Geographics) and USGS The National Map
- Elevation: Mapzen / AWS Terrain Tiles and USGS 3DEP
- Weather: NOAA aviationweather.gov; forecasts from Open-Meteo and the US National Weather Service
- Runway data: FAA via AirNav, OurAirports, and OpenStreetMap contributors
- Buildings, taxiways and stands: OpenStreetMap contributors; New York buildings from NYC Open Data
- Fonts: Barlow, served by Google Fonts
