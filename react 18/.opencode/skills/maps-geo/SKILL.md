---
name: maps-geo
description: Maps and geodata with leaflet 1.9, react-leaflet 4.2, leaflet-draw, react-leaflet-cluster, mapbox-gl 2.13, maplibre-gl 2.4, react-map-gl 7, mapbox-gl-draw, and @turf helpers. Use when building map views, markers, clusters, draw/edit tools, geofences, or spatial checks.
---

# Maps + Geo (pinned: leaflet@1.9, react-leaflet@4.2, cluster@2.1, mapbox-gl@2.13, maplibre@2.4, react-map-gl@7, turf@6.5)

Offline-first. Two stacks coexist: Leaflet (default) and Mapbox/MapLibre (rich vector). Do not mix both on one screen.

## 1. Leaflet + react-leaflet v4 essentials

```tsx
import 'leaflet/dist/leaflet.css'
import { MapContainer, TileLayer, Marker, Popup } from 'react-leaflet'

;<MapContainer
  center={[51.16, 71.43]}
  zoom={12}
  style={{ height: 600 }}
  scrollWheelZoom
>
  <TileLayer
    url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
    attribution="© OpenStreetMap"
  />
  <Marker position={[51.16, 71.43]}>
    <Popup>Astana</Popup>
  </Marker>
</MapContainer>
```

Marker icon fix (Leaflet + bundlers): default icon URLs break — configure once:

```ts
import L from 'leaflet'
import markerUrl from 'leaflet/dist/images/marker-icon.png'
import shadowUrl from 'leaflet/dist/images/marker-shadow.png'

L.Icon.Default.mergeOptions({ iconUrl: markerUrl, shadowUrl: shadowUrl })
```

`MapContainer` props are immutable after mount — to move the view use `useMap().setView()` or `<Recenter>` child, never expect `center=` prop updates to fly.

## 2. Clustering + draw

- Cluster: `react-leaflet-cluster` around `<Marker>` lists >200 points.
- Draw/edit: `leaflet-draw` (+ `@types/leaflet-draw`) via `L.Control.Draw` in a `useEffect` on the map instance; persist results to store/API on `draw:created/edited/deleted`. Cleanup control on unmount.
- Mapbox draw: `@mapbox/mapbox-gl-draw` (+ `@types/mapbox__mapbox-gl-draw`) only with mapbox stack.

## 3. Mapbox / MapLibre / react-map-gl v7

```tsx
import Map from 'react-map-gl/maplibre'
import 'maplibre-gl/dist/maplibre-gl.css'

;<Map
  initialViewState={{ longitude: 71.43, latitude: 51.16, zoom: 11 }}
  style={{ height: 600 }}
  mapStyle="https://demotiles.maplibre.org/style.json"
/>
```

Import path `react-map-gl/maplibre` for MapLibre, `react-map-gl` for Mapbox token stack. Never hardcode tokens — env only.

## 4. Turf v6 spatial ops (all @turf/* are 6.5)

```ts
import bbox from '@turf/bbox'
import booleanPointInPolygon from '@turf/boolean-point-in-polygon'
import intersect from '@turf/intersect'
import mask from '@turf/mask'
import transformScale from '@turf/transform-scale'

const box = bbox(polygon) // fitBounds input
const inside = booleanPointInPolygon(point, polygon)
const clipped = intersect(polyA, polyB)
```

Heavy ops (mask/intersect on large polys) go to `useMemo` or worker — never per-render. Validate GeoJSON before turf calls (a null ring crashes intersect).

## 5. Perf + lifecycle rules

1. Markers >500 → cluster (Leaflet) or `cluster` source prop (MapLibre). Never render 5k raw markers.
2. Tile layer + attribution always (license compliance).
3. Map event listeners (`moveend`, `draw:created`) added in effects with `map.off` cleanup — StrictMode-safe.
4. Selection state in zustand (client-state skill), map only renders it; URL holds view (`?z=&lat=&lng=`) for shareable links.
5. Resize: call `map.invalidateSize()` after drawer/modal animations.

## Checklist

- [ ] Icon fix applied (Leaflet).
- [ ] View changes via map instance, not MapContainer props.
- [ ] Clustering over thresholds.
- [ ] Draw controls cleaned up; turf inputs validated.
