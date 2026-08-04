---
layout: page
permalink: /travel-map/
title: Photo Map
---

<style>
/* Keep the map inside the site's normal article column. */
.travel-map-page {
  width: 100%;
  margin-top: 1rem;
}

/* Main map surface without a separate floating-card frame. */
.travel-map-canvas {
  position: relative;
  width: 100%;
  height: clamp(480px, calc(100vh - 10rem), 680px);
  overflow: hidden;
  background: #fbf8ff;
}

/* D3-rendered world map base. */
.travel-map-base {
  position: absolute;
  inset: 0;
  z-index: 0;
  background: linear-gradient(180deg, #fbf8ff 0%, #f3fbff 100%);
}

.travel-map-base svg {
  display: block;
  width: 100%;
  height: 100%;
  cursor: grab;
  touch-action: none;
}

.travel-map-base svg:active {
  cursor: grabbing;
}

/* Quiet label on the map surface. */
.travel-map-watermark {
  position: absolute;
  z-index: 3;
  left: 1rem;
  bottom: 0.75rem;
  color: rgba(77, 45, 101, 0.35);
  font-size: 0.8rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

/* Manual zoom controls for mouse, touchpad, and keyboard users. */
.travel-map-zoom-controls {
  position: absolute;
  z-index: 4;
  top: 1rem;
  left: 1rem;
  display: flex;
  gap: 0.35rem;
  align-items: center;
}

.travel-map-zoom-controls button {
  width: 2rem;
  height: 2rem;
  border: 1px solid #dec9ed;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.9);
  color: #5c3a73;
  cursor: pointer;
  font-size: 1rem;
  line-height: 1;
  box-shadow: 0 8px 18px rgba(31, 21, 41, 0.1);
}

.travel-map-zoom-controls button:hover,
.travel-map-zoom-controls button:focus {
  border-color: DarkViolet;
  color: DarkViolet;
}

.travel-map-zoom-reset {
  width: auto !important;
  padding: 0 0.75rem;
  font-size: 0.82rem !important;
}

/* One city-level location. Exact photo places live inside this city's hover panel. */
.travel-map-location {
  position: absolute;
  left: var(--x);
  top: var(--y);
  z-index: 2;
  transform: translate(-50%, -50%);
}

/* Keep the active location above nearby map points and panels. */
.travel-map-location:hover,
.travel-map-location:focus-within {
  z-index: 20;
}

/* Clickable city marker. */
.travel-map-marker {
  display: block;
  width: 18px;
  height: 18px;
  padding: 0;
  border: 2px solid #fff;
  border-radius: 999px;
  background: DarkViolet;
  box-shadow: 0 0 0 7px rgba(148, 0, 211, 0.15), 0 8px 18px rgba(62, 20, 80, 0.24);
  cursor: pointer;
  transition: transform 180ms ease, box-shadow 180ms ease, background 180ms ease;
}

/* City name tooltip above each marker. */
.travel-map-marker::after {
  content: attr(aria-label);
  position: absolute;
  left: 50%;
  bottom: 170%;
  min-width: max-content;
  padding: 0.25rem 0.55rem;
  border-radius: 999px;
  background: rgba(44, 31, 56, 0.9);
  color: #fff;
  font-size: 0.75rem;
  opacity: 0;
  pointer-events: none;
  transform: translateX(-50%) translateY(4px);
  transition: opacity 160ms ease, transform 160ms ease;
}

/* Hover and focus state for the city marker. */
.travel-map-location:hover .travel-map-marker,
.travel-map-location:focus-within .travel-map-marker {
  background: #ff6f91;
  box-shadow: 0 0 0 9px rgba(255, 111, 145, 0.18), 0 10px 24px rgba(62, 20, 80, 0.28);
  transform: scale(1.18);
}

.travel-map-location:hover .travel-map-marker::after,
.travel-map-location:focus-within .travel-map-marker::after {
  opacity: 1;
  transform: translateX(-50%) translateY(0);
}

/* Hidden city photo panel. It appears only while hovering or focusing a city marker. */
.travel-map-popover {
  position: absolute;
  top: 50%;
  right: auto;
  left: calc(100% + 1rem);
  box-sizing: border-box;
  width: 430px;
  max-width: calc(100vw - 2rem);
  max-height: min(72vh, 680px);
  padding: 1rem;
  border: 1px solid #eadff3;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.96);
  box-shadow: 0 20px 50px rgba(31, 21, 41, 0.18);
  opacity: 0;
  pointer-events: none;
  transform: translateY(-50%) translateX(8px);
  transition: opacity 160ms ease, transform 160ms ease, visibility 160ms ease;
  visibility: hidden;
}

/* Reveal the selected city's photo panel. */
.travel-map-location:hover .travel-map-popover,
.travel-map-location:focus-within .travel-map-popover {
  opacity: 1;
  pointer-events: auto;
  transform: translateY(-50%) translateX(0);
  visibility: visible;
}

/* Invisible bridge so the panel does not disappear while moving from marker to panel. */
.travel-map-popover::before {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: -1rem;
  width: 1rem;
}

/* City heading inside hover and expanded panels. */
.travel-map-popover h2,
.travel-map-dialog h2 {
  margin: 0 0 0.35rem;
  color: #3e2853;
  font-size: 1.25rem;
}

/* City-level date and photo count. */
.travel-map-city-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 0.45rem 0.75rem;
  align-items: center;
  margin-bottom: 0.65rem;
}

.travel-map-date {
  color: DarkViolet;
  font-weight: 700;
}

.travel-map-photo-count {
  color: #777;
  font-size: 0.86rem;
}

/* City-level description before the photo list. */
.travel-map-caption {
  color: #555;
  line-height: 1.55;
  margin: 0 0 0.85rem;
}

/* Button that opens the selected city into the larger detail view. */
.travel-map-open {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 0.9rem;
  border: 1px solid #dec9ed;
  border-radius: 999px;
  padding: 0.35rem 0.8rem;
  background: #fff;
  color: #5c3a73;
  cursor: pointer;
  font-size: 0.88rem;
}

.travel-map-open:hover,
.travel-map-open:focus {
  border-color: DarkViolet;
  color: DarkViolet;
}

/* Scrollable photo list for cities with many images. */
.travel-map-feed {
  max-height: min(48vh, 470px);
  overflow-y: auto;
  padding-right: 0.25rem;
}

/* One photo and its exact shooting place in the hover panel. */
.travel-map-photo {
  display: grid;
  grid-template-columns: 112px minmax(0, 1fr);
  gap: 0.75rem;
  align-items: start;
  padding-bottom: 0.9rem;
  margin-bottom: 0.9rem;
  border-bottom: 1px solid #eee4f4;
}

.travel-map-photo:last-child {
  padding-bottom: 0;
  margin-bottom: 0;
  border-bottom: 0;
}

/* Click target around every image so the original ratio can be viewed larger. */
.travel-map-photo-zoom {
  display: block;
  width: 100%;
  padding: 0;
  border: 0;
  background: transparent;
  cursor: zoom-in;
}

/* Compact thumbnail for long city photo lists. */
.travel-map-photo img {
  display: block;
  width: 112px;
  aspect-ratio: 4 / 3;
  object-fit: cover;
  border-radius: 8px;
  box-shadow: 0 8px 18px rgba(31, 21, 41, 0.12);
}

/* Exact shooting place for each photo. */
.travel-map-photo h3,
.travel-map-dialog-photo h3 {
  margin: 0 0 0.2rem;
  color: #3e2853;
  font-size: 0.98rem;
  line-height: 1.3;
}

.travel-map-photo-meta {
  color: DarkViolet;
  font-size: 0.84rem;
  font-weight: 700;
  margin-bottom: 0.3rem;
}

.travel-map-photo-note {
  color: #555;
  font-size: 0.9rem;
  line-height: 1.45;
  margin: 0;
}

/* Full-screen expanded city view. */
.travel-map-dialog {
  position: fixed;
  inset: 0;
  z-index: 1000;
  display: none;
  padding: clamp(1rem, 3vw, 2.5rem);
  background: rgba(31, 21, 41, 0.58);
}

.travel-map-dialog.is-open {
  display: block;
}

/* Inner sheet for the expanded city photo page. */
.travel-map-dialog-inner {
  position: relative;
  max-width: 1100px;
  max-height: calc(100vh - 5rem);
  margin: 0 auto;
  overflow-y: auto;
  border-radius: 8px;
  background: #fff;
  box-shadow: 0 24px 70px rgba(31, 21, 41, 0.28);
}

/* Sticky dialog header with close control. */
.travel-map-dialog-header {
  position: sticky;
  top: 0;
  z-index: 2;
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  align-items: start;
  padding: 1.25rem 1.25rem 0.9rem;
  border-bottom: 1px solid #eee4f4;
  background: rgba(255, 255, 255, 0.96);
}

/* Close button for expanded views. */
.travel-map-close,
.travel-map-lightbox-close {
  flex: 0 0 auto;
  border: 1px solid #dec9ed;
  border-radius: 999px;
  width: 2rem;
  height: 2rem;
  background: #fff;
  color: #5c3a73;
  cursor: pointer;
  font-size: 1.15rem;
  line-height: 1;
}

.travel-map-close:hover,
.travel-map-close:focus,
.travel-map-lightbox-close:hover,
.travel-map-lightbox-close:focus {
  border-color: DarkViolet;
  color: DarkViolet;
}

/* Expanded photo grid inside the dialog. */
.travel-map-dialog-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 1rem;
  padding: 1.25rem;
}

/* One expanded photo entry. */
.travel-map-dialog-photo {
  border: 1px solid #eee4f4;
  border-radius: 8px;
  overflow: hidden;
  background: #fbf8ff;
}

/* Dialog thumbnails preserve the full composition instead of cropping. */
.travel-map-dialog-photo img {
  display: block;
  width: 100%;
  aspect-ratio: 4 / 3;
  object-fit: contain;
  background: #f8f5fb;
}

.travel-map-dialog-photo div {
  padding: 0.85rem;
}

/* Full-screen original-ratio photo viewer. */
.travel-map-lightbox {
  position: fixed;
  inset: 0;
  z-index: 1100;
  display: none;
  align-items: center;
  justify-content: center;
  padding: 2.5rem 1rem 1.5rem;
  background: rgba(23, 15, 32, 0.82);
}

.travel-map-lightbox.is-open {
  display: flex;
}

.travel-map-lightbox-inner {
  max-width: calc(100vw - 2rem);
  max-height: calc(100vh - 4rem);
  text-align: center;
}

.travel-map-lightbox img {
  display: block;
  max-width: 100%;
  max-height: calc(100vh - 9rem);
  object-fit: contain;
  border-radius: 8px;
  background: #f8f5fb;
  box-shadow: 0 24px 70px rgba(0, 0, 0, 0.35);
}

.travel-map-lightbox-caption {
  margin-top: 0.85rem;
  color: #fff;
}

.travel-map-lightbox-caption h3 {
  margin: 0 0 0.25rem;
  color: #fff;
}

.travel-map-lightbox-close {
  position: fixed;
  top: 1rem;
  right: 1rem;
}

/* Small screens use a bottom sheet because there is no reliable hover space. */
@media (max-width: 760px) {
  .travel-map-page {
    margin-top: 0.5rem;
  }

  .travel-map-canvas {
    width: 100%;
    height: calc(100vh - 6rem);
    min-height: 520px;
  }

  .travel-map-popover {
    position: fixed;
    right: 1rem;
    bottom: 1rem;
    left: 1rem;
    top: auto;
    width: auto;
    max-height: 68vh;
    transform: translateY(10px);
  }

  .travel-map-location:hover .travel-map-popover,
  .travel-map-location:focus-within .travel-map-popover {
    transform: translateY(0);
  }

  .travel-map-feed {
    max-height: 45vh;
  }

  .travel-map-photo {
    grid-template-columns: 92px minmax(0, 1fr);
  }

  .travel-map-photo img {
    width: 92px;
  }

  .travel-map-dialog {
    padding: 0.75rem;
  }

  .travel-map-dialog-inner {
    max-height: calc(100vh - 1.5rem);
  }
}
</style>

<div class="travel-map-page">
  <div class="travel-map-canvas" data-travel-map aria-label="World map with photo locations">
    <div class="travel-map-base" data-map-base></div>
    <div class="travel-map-zoom-controls" aria-label="Map zoom controls">
      <button type="button" data-map-zoom-in aria-label="Zoom in">+</button>
      <button type="button" data-map-zoom-out aria-label="Zoom out">-</button>
      <button class="travel-map-zoom-reset" type="button" data-map-zoom-reset aria-label="Reset map view">Reset</button>
    </div>

    <!-- One generated block equals one city marker plus its hidden hover panel. -->
    {% for place in site.data.travel_map %}
      <div
        class="travel-map-location"
        style="--x: {{ place.x }}%; --y: {{ place.y }}%;"
        data-map-location
        data-lat="{{ place.lat }}"
        data-lon="{{ place.lon }}"
      >
        <button class="travel-map-marker" type="button" aria-label="{{ place.city }}" data-open-map-place="{{ place.id }}"></button>
        <div class="travel-map-popover" aria-label="{{ place.city }} photos">
          <h2>{{ place.city }}</h2>
          <div class="travel-map-city-meta">
            <span class="travel-map-date">{{ place.date }}</span>
            <span class="travel-map-photo-count">{{ place.photos | size }} photos</span>
          </div>
          <p class="travel-map-caption">{{ place.description }}</p>
          <button class="travel-map-open" type="button" data-open-map-place="{{ place.id }}">Open full photo page</button>
          <div class="travel-map-feed">
            {% for photo in place.photos %}
              {% assign photo_title = photo.location | default: place.city %}
              <section class="travel-map-photo">
                <button
                  class="travel-map-photo-zoom"
                  type="button"
                  data-photo-src="{{ photo.src | escape }}"
                  data-photo-title="{{ photo_title | escape }}"
                  data-photo-date="{{ photo.date | escape }}"
                  aria-label="Open photo from {{ photo_title | escape }}"
                >
                  <img src="{{ photo.src }}" alt="{{ photo.alt }}" loading="lazy" decoding="async">
                </button>
                <div>
                  <h3>{{ photo_title }}</h3>
                  {% if photo.date %}
                    <div class="travel-map-photo-meta">{{ photo.date }}</div>
                  {% endif %}
                  {% if photo.note %}
                    <p class="travel-map-photo-note">{{ photo.note }}</p>
                  {% endif %}
                </div>
              </section>
            {% endfor %}
          </div>
        </div>
      </div>
    {% endfor %}

    <span class="travel-map-watermark">Photo Map</span>
  </div>

  <!-- Expanded city pages reuse the same data as the hover panels. -->
  {% for place in site.data.travel_map %}
    <section class="travel-map-dialog" data-map-dialog="{{ place.id }}" aria-hidden="true" role="dialog" aria-label="{{ place.city }} photo page">
      <div class="travel-map-dialog-inner">
        <header class="travel-map-dialog-header">
          <div>
            <h2>{{ place.city }}</h2>
            <div class="travel-map-city-meta">
              <span class="travel-map-date">{{ place.date }}</span>
              <span class="travel-map-photo-count">{{ place.photos | size }} photos</span>
            </div>
            <p class="travel-map-caption">{{ place.description }}</p>
          </div>
          <button class="travel-map-close" type="button" data-close-map-dialog aria-label="Close {{ place.city }} photo page">&times;</button>
        </header>
        <div class="travel-map-dialog-grid">
          {% for photo in place.photos %}
            {% assign photo_title = photo.location | default: place.city %}
            <article class="travel-map-dialog-photo">
              <button
                class="travel-map-photo-zoom"
                type="button"
                data-photo-src="{{ photo.src | escape }}"
                data-photo-title="{{ photo_title | escape }}"
                data-photo-date="{{ photo.date | escape }}"
                aria-label="Open photo from {{ photo_title | escape }}"
              >
                <img src="{{ photo.src }}" alt="{{ photo.alt }}" loading="lazy" decoding="async">
              </button>
              <div>
                <h3>{{ photo_title }}</h3>
                {% if photo.date %}
                  <div class="travel-map-photo-meta">{{ photo.date }}</div>
                {% endif %}
                {% if photo.note %}
                  <p class="travel-map-photo-note">{{ photo.note }}</p>
                {% endif %}
              </div>
            </article>
          {% endfor %}
        </div>
      </div>
    </section>
  {% endfor %}

  <!-- Original-ratio photo viewer for any thumbnail. -->
  <section class="travel-map-lightbox" data-photo-lightbox aria-hidden="true" role="dialog" aria-label="Expanded photo">
    <button class="travel-map-lightbox-close" type="button" data-close-photo-lightbox aria-label="Close expanded photo">&times;</button>
    <div class="travel-map-lightbox-inner">
      <img src="" alt="" data-photo-lightbox-image>
      <div class="travel-map-lightbox-caption">
        <h3 data-photo-lightbox-title></h3>
        <div class="travel-map-photo-meta" data-photo-lightbox-date></div>
      </div>
    </div>
  </section>
</div>

<script src="https://unpkg.com/d3@7"></script>
<script src="https://unpkg.com/topojson-client@3"></script>
<script>
(function () {
  // Collect shared page elements before wiring the map and dialogs.
  var page = document.querySelector('.travel-map-page');
  if (!page) return;

  var mapBase = page.querySelector('[data-map-base]');
  var locations = page.querySelectorAll('[data-map-location]');
  var openControls = page.querySelectorAll('[data-open-map-place]');
  var dialogs = page.querySelectorAll('[data-map-dialog]');
  var closeControls = page.querySelectorAll('[data-close-map-dialog]');
  var photoControls = page.querySelectorAll('[data-photo-src]');
  var lightbox = page.querySelector('[data-photo-lightbox]');
  var lightboxImage = page.querySelector('[data-photo-lightbox-image]');
  var lightboxTitle = page.querySelector('[data-photo-lightbox-title]');
  var lightboxDate = page.querySelector('[data-photo-lightbox-date]');
  var closeLightboxControl = page.querySelector('[data-close-photo-lightbox]');
  var zoomInControl = page.querySelector('[data-map-zoom-in]');
  var zoomOutControl = page.querySelector('[data-map-zoom-out]');
  var zoomResetControl = page.querySelector('[data-map-zoom-reset]');
  var mapProjection = null;
  var mapZoom = null;
  var mapSvg = null;
  var mapInitialTransform = d3.zoomIdentity;
  var mapCurrentTransform = d3.zoomIdentity;

  // Keep body scrolling locked only while an overlay is open.
  function updateBodyLock() {
    var hasOpenDialog = page.querySelector('.travel-map-dialog.is-open');
    var hasOpenLightbox = page.querySelector('.travel-map-lightbox.is-open');
    document.body.style.overflow = hasOpenDialog || hasOpenLightbox ? 'hidden' : '';
  }

  // Position custom city markers from real latitude and longitude on the projected SVG map.
  function positionLocations(projection, transform) {
    locations.forEach(function (location) {
      var lat = Number(location.getAttribute('data-lat'));
      var lon = Number(location.getAttribute('data-lon'));
      if (!Number.isFinite(lat) || !Number.isFinite(lon)) return;

      var point = projection([lon, lat]);
      if (!point) return;
      if (transform) point = transform.apply(point);

      location.style.left = point[0] + 'px';
      location.style.top = point[1] + 'px';
    });
  }

  // Apply the same zoom transform to the map paths and the custom HTML markers.
  function applyMapTransform(transform, mapLayer) {
    mapCurrentTransform = transform;
    mapLayer.attr('transform', transform);
    if (mapProjection) positionLocations(mapProjection, transform);
  }

  // Zoom from the controls without changing the photo interactions.
  function zoomMapBy(scaleFactor) {
    if (!mapSvg || !mapZoom) return;
    mapSvg.transition().duration(180).call(mapZoom.scaleBy, scaleFactor);
  }

  // Reset to the Pacific-centered full-world view.
  function resetMapZoom() {
    if (!mapSvg || !mapZoom) return;
    mapSvg.transition().duration(180).call(mapZoom.transform, mapInitialTransform);
  }

  // Build the purple world map from real Natural Earth country boundaries.
  function initWorldMap() {
    if (!mapBase || !window.d3 || !window.topojson) return;

    fetch('https://unpkg.com/world-atlas@2/countries-110m.json')
      .then(function (response) { return response.json(); })
      .then(function (world) {
        var width = mapBase.clientWidth;
        var height = mapBase.clientHeight;
        var countries = topojson.feature(world, world.objects.countries);
        var coastlines = topojson.mesh(world, world.objects.countries, function (a, b) { return a === b; });
        var borders = topojson.mesh(world, world.objects.countries, function (a, b) { return a !== b; });
        var projection = d3.geoNaturalEarth1()
          .rotate([-150, 0])
          .fitExtent([[28, 28], [width - 28, height - 28]], { type: 'Sphere' });
        var path = d3.geoPath(projection);
        var svg = d3.select(mapBase).html('').append('svg').attr('viewBox', '0 0 ' + width + ' ' + height);
        mapProjection = projection;
        mapSvg = svg;
        mapInitialTransform = d3.zoomIdentity;
        mapCurrentTransform = mapInitialTransform;

        var defs = svg.append('defs');
        var gradient = defs.append('linearGradient')
          .attr('id', 'travel-map-ocean-gradient')
          .attr('x1', '0')
          .attr('x2', '0')
          .attr('y1', '0')
          .attr('y2', '1');

        gradient.append('stop').attr('offset', '0%').attr('stop-color', '#fbf8ff');
        gradient.append('stop').attr('offset', '55%').attr('stop-color', '#f4ecfb');
        gradient.append('stop').attr('offset', '100%').attr('stop-color', '#f3fbff');

        svg.append('path')
          .datum({ type: 'Sphere' })
          .attr('d', path)
          .attr('fill', 'url(#travel-map-ocean-gradient)');

        var mapLayer = svg.append('g');

        mapLayer.append('path')
          .datum(d3.geoGraticule10())
          .attr('d', path)
          .attr('fill', 'none')
          .attr('stroke', 'rgba(100, 76, 120, 0.16)')
          .attr('stroke-width', 0.65);

        mapLayer.append('g')
          .selectAll('path')
          .data(countries.features)
          .join('path')
          .attr('d', path)
          .attr('fill', '#dfd0eb')
          .attr('stroke', 'none')
          .attr('opacity', 0.92);

        mapLayer.append('path')
          .datum(coastlines)
          .attr('d', path)
          .attr('fill', 'none')
          .attr('stroke', '#6f4a86')
          .attr('stroke-width', 1.05)
          .attr('stroke-linejoin', 'round')
          .attr('stroke-linecap', 'round')
          .attr('opacity', 0.86);

        mapLayer.append('path')
          .datum(borders)
          .attr('d', path)
          .attr('fill', 'none')
          .attr('stroke', '#ffffff')
          .attr('stroke-width', 0.78)
          .attr('stroke-linejoin', 'round')
          .attr('stroke-linecap', 'round')
          .attr('opacity', 0.86);

        svg.append('text')
          .attr('x', width - 12)
          .attr('y', height - 10)
          .attr('text-anchor', 'end')
          .attr('fill', 'rgba(77, 45, 101, 0.45)')
          .attr('font-size', 10)
          .text('Natural Earth / world-atlas');

        mapZoom = d3.zoom()
          .scaleExtent([1, 7])
          .translateExtent([[0, 0], [width, height]])
          .extent([[0, 0], [width, height]])
          .on('zoom', function (event) {
            applyMapTransform(event.transform, mapLayer);
          });

        svg.call(mapZoom);
        applyMapTransform(mapInitialTransform, mapLayer);
      })
      .catch(function () {
        locations.forEach(function (location) {
          location.style.left = location.style.getPropertyValue('--x');
          location.style.top = location.style.getPropertyValue('--y');
        });
      });
  }

  // Open the matching city dialog and keep the page behind it still.
  function openDialog(placeId) {
    dialogs.forEach(function (dialog) {
      var isTarget = dialog.getAttribute('data-map-dialog') === placeId;
      dialog.classList.toggle('is-open', isTarget);
      dialog.setAttribute('aria-hidden', isTarget ? 'false' : 'true');
    });
    updateBodyLock();
  }

  // Close every expanded city dialog.
  function closeDialogs() {
    dialogs.forEach(function (dialog) {
      dialog.classList.remove('is-open');
      dialog.setAttribute('aria-hidden', 'true');
    });
    updateBodyLock();
  }

  // Open one photo at its natural ratio in the lightbox.
  function openLightbox(control) {
    var image = control.querySelector('img');
    var src = control.getAttribute('data-photo-src');
    var title = control.getAttribute('data-photo-title') || '';
    var date = control.getAttribute('data-photo-date') || '';

    lightboxImage.setAttribute('src', src);
    lightboxImage.setAttribute('alt', image ? image.getAttribute('alt') || title : title);
    lightboxTitle.textContent = title;
    lightboxDate.textContent = date;
    lightbox.classList.add('is-open');
    lightbox.setAttribute('aria-hidden', 'false');
    updateBodyLock();
  }

  // Close the original-ratio photo viewer.
  function closeLightbox() {
    lightbox.classList.remove('is-open');
    lightbox.setAttribute('aria-hidden', 'true');
    lightboxImage.setAttribute('src', '');
    updateBodyLock();
  }

  // Left-click or right-click on a city opens the expanded page.
  openControls.forEach(function (control) {
    control.addEventListener('click', function () {
      openDialog(control.getAttribute('data-open-map-place'));
    });

    control.addEventListener('contextmenu', function (event) {
      event.preventDefault();
      openDialog(control.getAttribute('data-open-map-place'));
    });
  });

  // Close buttons dismiss the expanded city page.
  closeControls.forEach(function (control) {
    control.addEventListener('click', closeDialogs);
  });

  // Clicking a thumbnail opens the original-ratio photo.
  photoControls.forEach(function (control) {
    control.addEventListener('click', function () {
      openLightbox(control);
    });
  });

  // Lightbox close button dismisses only the enlarged photo.
  if (closeLightboxControl) {
    closeLightboxControl.addEventListener('click', closeLightbox);
  }

  // Manual zoom controls mirror mouse-wheel and trackpad zoom.
  if (zoomInControl) {
    zoomInControl.addEventListener('click', function () {
      zoomMapBy(1.35);
    });
  }

  if (zoomOutControl) {
    zoomOutControl.addEventListener('click', function () {
      zoomMapBy(1 / 1.35);
    });
  }

  if (zoomResetControl) {
    zoomResetControl.addEventListener('click', resetMapZoom);
  }

  // Clicking dark overlays outside their content closes the matching overlay.
  dialogs.forEach(function (dialog) {
    dialog.addEventListener('click', function (event) {
      if (event.target === dialog) closeDialogs();
    });
  });

  if (lightbox) {
    lightbox.addEventListener('click', function (event) {
      if (event.target === lightbox) closeLightbox();
    });
  }

  // Escape closes the topmost overlay first.
  document.addEventListener('keydown', function (event) {
    if (event.key !== 'Escape') return;
    if (lightbox && lightbox.classList.contains('is-open')) {
      closeLightbox();
    } else {
      closeDialogs();
    }
  });

  initWorldMap();
}());
</script>
