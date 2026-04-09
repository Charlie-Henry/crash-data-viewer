# Crash Data Viewer

An interactive, single-file web map for visualizing traffic crash data. Built with [Mapbox GL JS](https://docs.mapbox.com/mapbox-gl-js/) and [PapaParse](https://www.papaparse.com/). No build step required — just a browser and a web server.

---

## Live Demo

[Clark County, NV deployment](https://clark-county-vz-viewer.onrender.com/)

---

## Features

- Interactive map with crash point markers colored by severity
- Sidebar filters: severity, involvement flags, date range, time of day, day of week
- Click any crash marker to open a popup with full crash details
- Time-of-day bar chart and day-of-week heatmap, both clickable as filters
- All filters are independent and composable
- Responsive layout (mobile-friendly)

---

## Quick Start

### 1. Get a Mapbox token

Sign up at [mapbox.com](https://account.mapbox.com/auth/signup/) — the free tier covers up to 50,000 map loads per month.

In your Mapbox account, restrict the token to your deployment domain to prevent unauthorized use.

### 2. Configure `config.js`

Edit `config.js`:

```js
window.ENV = {
  MAPBOX_TOKEN: 'pk.eyJ1...'   // add your own Mapbox token
};
```

### 3. Configure `index.html`

Open `index.html` and find the `CONFIG` block near the top of the `<script>` section. Update the values for your deployment:

```js
const CONFIG = {
  mapboxToken: (window.ENV && window.ENV.MAPBOX_TOKEN) || 'YOUR_MAPBOX_TOKEN_HERE',

  // URL to your publicly accessible CSV file
  csvUrl: 'https://storage.googleapis.com/your-bucket/your-data.csv',

  // Map your CSV column names to the expected internal names
  columns: {
    lat:            'lat',
    lng:            'lng',
    severity:       'severity',
    date:           'date',
    numFatalities:  'NumFatalities',
    numInjured:     'NumInjured',
    crashType:      'VehCrashType',
    vehicleFactors: 'vehicle_factors',
    crashNum:       'CrashNum',
    hitAndRun:      'HitandRun',
    alcohol:        'AlcoholInvolved',
    pedestrian:     'PedestrianInvolved',
    motorcycle:     'MotorcycleInvolved',
    extras: []  // optional extra fields to show in the popup
  },

  // Map raw severity values from your CSV to internal categories
  severityMap: {
    'Fatal':          'fatal',
    'Serious Injury': 'serious',
  },

  // Starting map view
  initialCenter: [-115.1398, 36.1699],
  initialZoom:   10,

  // Hard pan/zoom limits — set to null to allow unrestricted panning
  maxBounds: [
    [-116.790925, 34.277257],  // [minLon, minLat]
    [-113.148819, 37.578262],  // [maxLon, maxLat]
  ],

  // UI text — update for your region
  ui: {
    eyebrow:       'Clark County, NV',
    title:         'Crash Data Viewer',
    attribution:   'Data from <a href="...">Source</a> — Visualization by Your Name',
    fatalLabel:    'Fatalities',
    seriousLabel:  'Serious Injury',
  },
};
```

### 4. Host the CSV publicly

The browser fetches the CSV directly, so it must be publicly hosted somewhere. I have used Google Cloud Storage,
but an AWS S3 bucket would also work.

You will likely need to configure CORS headers for your bucket
- [GCS](https://docs.cloud.google.com/storage/docs/using-cors)
- [AWS](https://docs.aws.amazon.com/AmazonS3/latest/userguide/enabling-cors-examples.html)

### 5. Run locally

The file must be served over HTTP (not opened as `file://`). Any of these options work to spin it up locally:

```bash
# Python (no install needed)
python3 -m http.server 8080

# Node
npx serve .

# VS Code
Install the "Live Server" extension, then right-click index.html → Open with Live Server
```

Then open `http://localhost:8080`.

---

## CSV Column Reference

All column names are configurable in `CONFIG.columns`. The table below shows the default expected names and their definitions.

### Required columns

| Column | Default name | Type | Description |
|---|---|---|---|
| Latitude | `lat` | Float | Decimal latitude of the crash location (WGS84) |
| Longitude | `lng` | Float | Decimal longitude of the crash location (WGS84) |
| Severity | `severity` | String | Crash severity category. Must match a key in `CONFIG.severityMap`. Default values: `Fatal`, `Serious Injury` |
| Date | `date` | Datetime | Date and time of the crash. Any format parseable by JavaScript's `Date()` constructor is accepted, e.g. `2023-06-15 14:32:00` |

### Recommended columns (shown in popups)

| Column | Default name | Type | Description |
|---|---|---|---|
| Crash number | `CrashNum` | String | Unique identifier for the crash record |
| Crash type | `VehCrashType` | String | Collision type description, e.g. `Rear End`, `Angle`, `Head On` |
| Fatalities | `NumFatalities` | Integer | Number of people killed in the crash. Used to compute the total fatalities count in the sidebar |
| Injuries | `NumInjured` | Integer | Number of people injured in the crash |
| Vehicle factors | `vehicle_factors` | String | Semicolon-separated list of contributing factors, e.g. `Speeding;Failure to Yield`. Rendered as a bulleted list in the popup. Leave blank if none |

### Involvement flag columns (boolean filters)

These columns drive the Involvement filter buttons in the sidebar. Each should contain `True` or `False` (case-insensitive).

| Column | Default name | Description |
|---|---|---|
| Alcohol involved | `AlcoholInvolved` | Whether alcohol was a contributing factor |
| Pedestrian involved | `PedestrianInvolved` | Whether a pedestrian was involved |
| Motorcycle involved | `MotorcycleInvolved` | Whether a motorcycle was involved |
| Hit and run | `HitandRun` | Whether the crash was a hit-and-run |

### Optional extras

You can add any number of additional columns to display in the popup by adding entries to `CONFIG.columns.extras`:

```js
extras: [
  { label: 'Road Name', col: 'road_name' },
  { label: 'Weather',   col: 'weather_condition' },
]
```

---

## Adapting for a New Region

To deploy this viewer for a different area:

1. Update `CONFIG.csvUrl` to point to your hosted crash data
2. Update `CONFIG.columns` to match your CSV headers (or just match the schema provided)
3. Update `CONFIG.severityMap` to match your severity values
4. Update `CONFIG.initialCenter` and `CONFIG.initialZoom` for your geography
5. Update `CONFIG.maxBounds` to restrict panning to your region (`[minLon, minLat], [maxLon, maxLat]`) — or set to `null` to remove the restriction
6. Update `CONFIG.ui` with your region name, title, and attribution text

Everything else (filters, charts, popups, map styling) works automatically from those values.

---

## Customization Reference

### Adding a new involvement filter

1. Add a column entry to `CONFIG.columns` pointing to your boolean CSV column
2. Add an entry to the `FLAGS` array in `index.html`:

```js
{ id: 'your-id', label: 'Your Label', faClass: 'fa-solid fa-icon-name', col: 'your-column-key' }
```

`faClass` must be a [FontAwesome 6 free](https://fontawesome.com/search?o=r&m=free) icon class string.

### Adding a new severity category

1. Add a mapping to `CONFIG.severityMap`
2. Add an entry to the `SEVERITIES` array:

```js
{ id: 'your-id', label: 'Your Label', color: '#hexcolor', radius: 6 }
```

### Changing the color scheme

Edit the CSS custom properties at the top of the `<style>` block:

```css
:root {
  --accent:  #2563eb;  /* filter highlight, links */
  --fatal:   #FE343B;  /* fatal crash marker color */
  --serious: #FEBC40;  /* serious injury marker color */
}
```

---

## Dependencies

All loaded from public CDNs — no npm install required.

| Library | Version | Purpose |
|---|---|---|
| [Mapbox GL JS](https://docs.mapbox.com/mapbox-gl-js/) | 3.3.0 | Map rendering |
| [PapaParse](https://www.papaparse.com/) | 5.4.1 | CSV parsing |
| [Font Awesome](https://fontawesome.com/) | 6.5.1 | Involvement filter icons |

---

## Data Processing

I supplied the Jupyter notebook I used to process the Clark County data I have received from the Nevada Department of Transportation. This is going to be highly tailored to that particular dataset which may or may not be helpful if you are looking to use this on a different agency's data.

---

## License

As a derivative of the [City of Austin's Vision Zero Viewer](https://visionzero.austin.gov/viewer/), this project is also released into the public domain.
