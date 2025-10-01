# Open EO Viewer

Open EO Viewer is a lightweight Leaflet application for exploring NASA's Global Imagery Browse Services (GIBS) layers. It lets you switch between common true-color products, pick a date, and sketch areas of interest directly on the map. Drawn shapes can be exported to GeoJSON for further analysis or sharing with other tools.

## Features

- **Leaflet + GIBS integration** – browse daily NASA imagery without needing an API key.
- **Layer switching** – toggle between VIIRS and MODIS true-color layers with a single dropdown.
- **Date selection** – start from the latest NOAA-20 true-color image (2025-06-06) or choose any other available day.
- **Drawing tools** – capture polygons or rectangles on top of the imagery and export them as GeoJSON.

## Getting started

1. Open `index.html` in a browser (or serve the repository with your favorite static file server).
2. Use the layer and date controls in the toolbar to update the map imagery.
3. Activate the drawing controls in the lower-left corner to sketch features and export them via the **Export GeoJSON** button.

## Contributing

This project is intentionally simple and we welcome ideas that make it more useful. If you build a new capability—such as additional imagery layers, new export formats, or enhanced map interactions—please open a pull request so others can benefit as well. Bug fixes, documentation improvements, and other suggestions are also appreciated!

## License

This repository is released under the MIT License. See [LICENSE](LICENSE) for details.

---

### Contribuciones
- Corrección menor de documentación.
- Añadido enlace a mi página personal: [Mi sitio](https://giocondafong.github.io/)

  <!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Open EO Viewer — Leaflet + NASA GIBS</title>
  <!-- Leaflet core -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin="anonymous">
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-Dov1AwlA5KfP5l3Zbo4VakiobP6KyHR+Y6wZl2QGItM=" crossorigin="anonymous"></script>

  <!-- Leaflet.draw -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet-draw@1.0.4/dist/leaflet.draw.css" />
  <script src="https://unpkg.com/leaflet-draw@1.0.4/dist/leaflet.draw.js"></script>

  <!-- Leaflet Control Geocoder -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.css" />
  <script src="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.js"></script>

  <!-- leaflet-image for map snapshot (may be limited by CORS policies of tile servers) -->
  <script src="https://unpkg.com/leaflet-image@0.4.0/leaflet-image.js"></script>

  <style>
    :root {
      --bg: #0b1220;
      --panel: #111829;
      --text: #dfe7ff;
      --muted: #94a3b8;
      --accent: #4f46e5;
      --accent-2: #06b6d4;
      --border: #24314a;
    }
    body { margin:0; font-family: system-ui, Segoe UI, Roboto, Helvetica, Arial, sans-serif; background:#0a0f1a; }
    #map { position:absolute; inset:0; }

    /* Top toolbar */
    .toolbar {
      position: fixed; top: 12px; left: 50%; transform: translateX(-50%);
      display: flex; flex-wrap: wrap; gap: 8px; align-items: center; justify-content: center;
      background: rgba(17, 24, 39, 0.85); border: 1px solid var(--border); border-radius: 16px;
      padding: 10px 12px; color: var(--text); backdrop-filter: blur(8px); z-index: 500;
    }
    .toolbar label { font-size: 12px; color: var(--muted); margin-right: 6px; }
    .toolbar select, .toolbar input[type="date"], .toolbar button {
      appearance: none; background: #0f172a; color: var(--text); border: 1px solid var(--border);
      border-radius: 10px; padding: 8px 10px; font-size: 13px;
    }
    .toolbar button { cursor: pointer; }
    .toolbar .split { width: 1px; height: 28px; background: var(--border); margin: 0 4px; }

    /* Bottom-left panel */
    .panel {
      position: fixed; left: 12px; bottom: 12px; z-index: 500; display: grid; gap: 8px;
      background: rgba(17, 24, 39, 0.85); border: 1px solid var(--border); border-radius: 14px;
      color: var(--text); padding: 10px; backdrop-filter: blur(8px);
    }
    .panel .row { display:flex; gap:6px; flex-wrap: wrap; }
    .panel button, .panel select { background:#0f172a; color:var(--text); border:1px solid var(--border); border-radius:10px; padding:8px 10px; font-size:13px; cursor:pointer; }
    .panel small { color: var(--muted); }

    .badge { font-size: 11px; padding: 4px 8px; border-radius: 999px; border: 1px solid var(--border); background:#0f172a; color:var(--muted); }

    .leaflet-control-container .leaflet-top.leaflet-left { margin-top: 80px; }

    @media (max-width: 720px) {
      .toolbar { width: calc(100% - 24px); left: 12px; right: 12px; transform:none; justify-content: flex-start; }
      .leaflet-control-container .leaflet-top.leaflet-left { margin-top: 120px; }
    }
  </style>
</head>
<body>
  <div id="map" role="region" aria-label="Mapa base con imágenes satelitales GIBS"></div>

  <!-- Toolbar superior: idioma, capa, fecha, timelapse -->
  <div class="toolbar" id="toolbar">
    <div>
      <label id="lblLang">Idioma</label>
      <select id="lang">
        <option value="es" selected>ES</option>
        <option value="en">EN</option>
      </select>
    </div>
    <div class="split"></div>
    <div>
      <label id="lblLayer">Capa</label>
      <select id="layerSelect"></select>
    </div>
    <div>
      <label id="lblDate">Fecha</label>
      <input id="date" type="date" value="2025-06-06" />
    </div>
    <div>
      <button id="btnApply">Aplicar</button>
    </div>
    <div class="split"></div>
    <div>
      <button id="btnPlay">▶︎</button>
      <select id="speed">
        <option value="500">2 fps</option>
        <option value="1000" selected>1 fps</option>
        <option value="1500">0.66 fps</option>
        <option value="2000">0.5 fps</option>
      </select>
    </div>
    <span class="badge" id="zoomInfo">zoom</span>
  </div>

  <!-- Panel inferior-izquierdo: dibujo, exportación, utilidades -->
  <div class="panel" id="panel">
    <div class="row"><small id="hintDraw">Usa los controles de dibujo ↖ para delinear áreas.</small></div>
    <div class="row">
      <button id="btnExportGeo">Exportar GeoJSON</button>
      <button id="btnExportKML">Exportar KML</button>
      <button id="btnExportCSV">Exportar CSV</button>
    </div>
    <div class="row">
      <button id="btnSnapshot">Descargar imagen visible</button>
      <button id="btnClear">Limpiar dibujos</button>
    </div>
  </div>

  <script>
    // ====== I18N ======
    const I18N = {
      es: {
        language: 'Idioma', layer: 'Capa', date: 'Fecha', apply: 'Aplicar',
        hintDraw: 'Usa los controles de dibujo ↖ para delinear áreas.',
        exportGeo: 'Exportar GeoJSON', exportKML: 'Exportar KML', exportCSV: 'Exportar CSV',
        snapshot: 'Descargar imagen visible', clear: 'Limpiar dibujos',
        play: 'Reproducir', pause: 'Pausar'
      },
      en: {
        language: 'Language', layer: 'Layer', date: 'Date', apply: 'Apply',
        hintDraw: 'Use the drawing controls ↖ to sketch areas.',
        exportGeo: 'Export GeoJSON', exportKML: 'Export KML', exportCSV: 'Export CSV',
        snapshot: 'Download visible image', clear: 'Clear drawings',
        play: 'Play', pause: 'Pause'
      }
    };

    const ui = {
      lblLang: document.getElementById('lblLang'),
      lblLayer: document.getElementById('lblLayer'),
      lblDate: document.getElementById('lblDate'),
      btnApply: document.getElementById('btnApply'),
      btnPlay: document.getElementById('btnPlay'),
      speed: document.getElementById('speed'),
      hintDraw: document.getElementById('hintDraw'),
      btnExportGeo: document.getElementById('btnExportGeo'),
      btnExportKML: document.getElementById('btnExportKML'),
      btnExportCSV: document.getElementById('btnExportCSV'),
      btnSnapshot: document.getElementById('btnSnapshot'),
      btnClear: document.getElementById('btnClear'),
      lang: document.getElementById('lang'),
      layerSelect: document.getElementById('layerSelect'),
      date: document.getElementById('date'),
      zoomInfo: document.getElementById('zoomInfo'),
    };

    function setLang(lang) {
      const t = I18N[lang] || I18N.es;
      ui.lblLang.textContent = t.language;
      ui.lblLayer.textContent = t.layer;
      ui.lblDate.textContent = t.date;
      ui.btnApply.textContent = t.apply;
      ui.hintDraw.textContent = t.hintDraw;
      ui.btnExportGeo.textContent = t.exportGeo;
      ui.btnExportKML.textContent = t.exportKML;
      ui.btnExportCSV.textContent = t.exportCSV;
      ui.btnSnapshot.textContent = t.snapshot;
      ui.btnClear.textContent = t.clear;
      ui.btnPlay.title = t.play;
    }

    ui.lang.addEventListener('change', e => setLang(e.target.value));
    setLang(ui.lang.value);

    // ====== Map ======
    const map = L.map('map', { zoomControl: true, worldCopyJump: true }).setView([20, 0], 3);
    map.attributionControl.setPrefix('');

    // Geocoder (search by name/coors)
    L.Control.geocoder({ defaultMarkGeocode: true, placeholder: 'Buscar / Search…' }).addTo(map);

    // Show zoom info
    function updateZoomInfo(){ ui.zoomInfo.textContent = `z${map.getZoom()}`; }
    map.on('zoomend', updateZoomInfo); updateZoomInfo();

    // ====== NASA GIBS WMTS config (EPSG:3857) ======
    // Reference: https://gibs.earthdata.nasa.gov/wmts/epsg3857/best/{Layer}/default/{Time}/{TileMatrixSet}/{z}/{y}/{x}.jpg
    // Many true-color layers max out at z=8 or z=9; GoogleMapsCompatible_Level9 is typical.
    const TILEMATRIX = 'GoogleMapsCompatible_Level9';

    const layers = {
      // True color
      'VIIRS NOAA-20 TrueColor': {
        id: 'VIIRS_NOAA20_CorrectedReflectance_TrueColor',
        format: 'jpg', maxZoom: 9
      },
      'VIIRS SNPP TrueColor': {
        id: 'VIIRS_SNPP_CorrectedReflectance_TrueColor',
        format: 'jpg', maxZoom: 9
      },
      'MODIS Terra TrueColor': {
        id: 'MODIS_Terra_CorrectedReflectance_TrueColor',
        format: 'jpg', maxZoom: 9
      },
      'MODIS Aqua TrueColor': {
        id: 'MODIS_Aqua_CorrectedReflectance_TrueColor',
        format: 'jpg', maxZoom: 9
      },
      // Extras (examples):
      'Fires (VIIRS 375m, NRT)': { id: 'FIRMS_VIIRS_SNPP_NRT', format: 'png', maxZoom: 9 },
      'Aerosol Optical Depth (MODIS)': { id: 'MODIS_Terra_Aerosol', format: 'png', maxZoom: 9 },
      'Snow Cover (MODIS)': { id: 'MODIS_Terra_Snow_Cover', format: 'png', maxZoom: 9 }
    };

    // Populate layer dropdown
    for (const k of Object.keys(layers)) {
      const opt = document.createElement('option'); opt.value = k; opt.textContent = k; ui.layerSelect.appendChild(opt);
    }
    ui.layerSelect.value = 'VIIRS NOAA-20 TrueColor';

    let currentTileLayer = null;

    function makeGIBSUrl(layerId, date, fmt, z, y, x) {
      return `https://gibs.earthdata.nasa.gov/wmts/epsg3857/best/${layerId}/default/${date}/${TILEMATRIX}/${z}/${y}/${x}.${fmt}`;
    }

    function setGIBSLayer() {
      const key = ui.layerSelect.value;
      const cfg = layers[key];
      const date = ui.date.value || new Date().toISOString().slice(0,10);

      // Remove previous
      if (currentTileLayer) { map.removeLayer(currentTileLayer); }

      currentTileLayer = L.tileLayer('', {
        minZoom: 1,
        maxZoom: cfg.maxZoom || 9,
        attribution: 'Imagery: NASA GIBS',
        tileSize: 256,
        updateWhenIdle: true,
        crossOrigin: true,
      });

      // Override getTileUrl to inject date + layer id
      currentTileLayer.getTileUrl = function(coords){
        return makeGIBSUrl(cfg.id, date, cfg.format || 'jpg', coords.z, coords.y, coords.x);
      };

      currentTileLayer.addTo(map);
    }

    // Initialize base layer
    setGIBSLayer();

    ui.btnApply.addEventListener('click', setGIBSLayer);

    // ====== Drawing tools ======
    const drawnItems = new L.FeatureGroup();
    map.addLayer(drawnItems);

    const drawControl = new L.Control.Draw({
      position: 'topleft',
      draw: {
        polygon: { allowIntersection: false, showArea: true, shapeOptions: { color: '#06b6d4' } },
        rectangle: { shapeOptions: { color: '#4f46e5' } },
        polyline: { shapeOptions: { color: '#22c55e' } },
        circle: { shapeOptions: { color: '#eab308' } },
        marker: true,
        circlemarker: false
      },
      edit: { featureGroup: drawnItems, remove: true }
    });
    map.addControl(drawControl);

    map.on(L.Draw.Event.CREATED, function (e) {
      drawnItems.addLayer(e.layer);
    });

    // ====== Export helpers ======
    function download(filename, text, mime='application/octet-stream'){
      const blob = new Blob([text], { type: mime });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url; a.download = filename; a.click();
      setTimeout(()=> URL.revokeObjectURL(url), 1000);
    }

    // GeoJSON
    ui.btnExportGeo.addEventListener('click', () => {
      const gj = drawnItems.toGeoJSON();
      download('features.geojson', JSON.stringify(gj, null, 2), 'application/geo+json');
    });

    // KML (simple converter for Point/LineString/Polygon)
    function coordsToKml(coords){ return coords.map(c => c[0] + ',' + c[1] + ',0').join(' '); }
    function gjToKml(gj){
      const feats = (gj.type === 'FeatureCollection') ? gj.features : [gj];
      const kmlParts = [
        `<?xml version="1.0" encoding="UTF-8"?>`,
        `<kml xmlns="http://www.opengis.net/kml/2.2"><Document>`
      ];
      feats.forEach((f,i)=>{
        const g = f.geometry; if(!g) return;
        kmlParts.push(`<Placemark><name>feat_${i+1}</name>`);
        if (g.type === 'Point') {
          const [x,y] = g.coordinates; kmlParts.push(`<Point><coordinates>${x},${y},0</coordinates></Point>`);
        } else if (g.type === 'LineString') {
          kmlParts.push(`<LineString><coordinates>${coordsToKml(g.coordinates)}</coordinates></LineString>`);
        } else if (g.type === 'Polygon') {
          const rings = g.coordinates.map(r => `<outerBoundaryIs><LinearRing><coordinates>${coordsToKml(r)}</coordinates></LinearRing></outerBoundaryIs>`).join('');
          kmlParts.push(`<Polygon>${rings}</Polygon>`);
        } else if (g.type === 'MultiPolygon') {
          g.coordinates.forEach(poly => {
            const rings = poly.map(r => `<outerBoundaryIs><LinearRing><coordinates>${coordsToKml(r)}</coordinates></LinearRing></outerBoundaryIs>`).join('');
            kmlParts.push(`<Polygon>${rings}</Polygon>`);
          });
        }
        kmlParts.push(`</Placemark>`);
      });
      kmlParts.push(`</Document></kml>`);
      return kmlParts.join('');
    }

    ui.btnExportKML.addEventListener('click', () => {
      const gj = drawnItems.toGeoJSON();
      const kml = gjToKml(gj);
      download('features.kml', kml, 'application/vnd.google-earth.kml+xml');
    });

    // CSV (lon,lat pairs for each feature)
    ui.btnExportCSV.addEventListener('click', () => {
      const gj = drawnItems.toGeoJSON();
      let rows = ['feature,index,lon,lat'];
      gj.features.forEach((f,fi) => {
        const g = f.geometry || {}; const t = g.type;
        function pushCoord(arr){ arr.forEach((c,ci)=> rows.push(`${fi},${ci},${c[0]},${c[1]}`)); }
        if (t === 'Point') pushCoord([g.coordinates]);
        else if (t === 'LineString') pushCoord(g.coordinates);
        else if (t === 'Polygon') g.coordinates.forEach(r => pushCoord(r));
        else if (t === 'MultiPolygon') g.coordinates.forEach(poly => poly.forEach(r => pushCoord(r)));
      });
      download('coordinates.csv', rows.join('\n'), 'text/csv');
    });

    // Clear drawings
    ui.btnClear.addEventListener('click', () => { drawnItems.clearLayers(); });

    // ====== Snapshot (map image) ======
    ui.btnSnapshot.addEventListener('click', () => {
      leafletImage(map, function(err, canvas) {
        if (err) { alert('Snapshot failed (CORS?).'); return; }
        canvas.toBlob((blob) => {
          const url = URL.createObjectURL(blob);
          const a = document.createElement('a');
          a.href = url; a.download = `map_${ui.date.value || 'today'}.png`;
          a.click(); setTimeout(()=> URL.revokeObjectURL(url), 1000);
        });
      });
    });

    // ====== Timelapse ======
    let timer = null;
    function stepDate(){
      const d = new Date(ui.date.value);
      d.setUTCDate(d.getUTCDate() + 1); // next day
      ui.date.value = d.toISOString().slice(0,10);
      setGIBSLayer();
    }
    ui.btnPlay.addEventListener('click', () => {
      if (timer) {
        clearInterval(timer); timer = null; ui.btnPlay.textContent = '▶︎'; ui.btnPlay.title = I18N[ui.lang.value].play; return;
      }
      const ms = parseInt(ui.speed.value, 10) || 1000;
      timer = setInterval(stepDate, ms);
      ui.btnPlay.textContent = '⏸'; ui.btnPlay.title = I18N[ui.lang.value].pause;
    });

    // Apply on date change by Enter key
    ui.date.addEventListener('keydown', (e)=>{ if(e.key==='Enter') setGIBSLayer(); });

  </script>
</body>
</html>
