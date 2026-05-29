---
layout: page
permalink: /travel-map/
title: Photo Map
---

<style>
.travel-map-page {
  margin-top: 1.5rem;
}

.travel-map-intro {
  color: #555;
  font-size: 1.05rem;
  line-height: 1.7;
  margin-bottom: 1.5rem;
}

.travel-map-shell {
  display: grid;
  grid-template-columns: minmax(0, 1.35fr) minmax(260px, 0.65fr);
  gap: 1.5rem;
  align-items: start;
}

.travel-map-canvas {
  position: relative;
  border: 1px solid #e9e1f3;
  border-radius: 24px;
  overflow: hidden;
  background: linear-gradient(180deg, #fbf8ff 0%, #f3fbff 100%);
  box-shadow: 0 18px 45px rgba(67, 31, 95, 0.08);
}

.travel-map-canvas svg {
  display: block;
  width: 100%;
  height: auto;
}

.travel-map-watermark {
  position: absolute;
  left: 1rem;
  bottom: 0.75rem;
  color: rgba(77, 45, 101, 0.35);
  font-size: 0.8rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.travel-map-marker {
  position: absolute;
  left: var(--x);
  top: var(--y);
  width: 18px;
  height: 18px;
  padding: 0;
  border: 2px solid #fff;
  border-radius: 999px;
  background: DarkViolet;
  box-shadow: 0 0 0 7px rgba(148, 0, 211, 0.15), 0 8px 18px rgba(62, 20, 80, 0.24);
  cursor: pointer;
  transform: translate(-50%, -50%);
  transition: transform 180ms ease, box-shadow 180ms ease, background 180ms ease;
}

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

.travel-map-marker:hover,
.travel-map-marker:focus,
.travel-map-marker.is-active {
  background: #ff6f91;
  box-shadow: 0 0 0 9px rgba(255, 111, 145, 0.18), 0 10px 24px rgba(62, 20, 80, 0.28);
  transform: translate(-50%, -50%) scale(1.18);
}

.travel-map-marker:hover::after,
.travel-map-marker:focus::after,
.travel-map-marker.is-active::after {
  opacity: 1;
  transform: translateX(-50%) translateY(0);
}

.travel-map-panel {
  border: 1px solid #eadff3;
  border-radius: 24px;
  padding: 1.25rem;
  background: rgba(255, 255, 255, 0.88);
  box-shadow: 0 18px 45px rgba(67, 31, 95, 0.07);
}

.travel-map-panel h2 {
  margin-top: 0;
  margin-bottom: 0.35rem;
  color: #3e2853;
}

.travel-map-date {
  color: DarkViolet;
  font-weight: 700;
  margin-bottom: 0.75rem;
}

.travel-map-caption {
  color: #555;
  line-height: 1.65;
  margin-bottom: 1rem;
}

.travel-map-gallery {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 0.65rem;
}

.travel-map-gallery img {
  display: block;
  width: 100%;
  aspect-ratio: 1 / 1;
  object-fit: cover;
  border-radius: 14px;
  box-shadow: 0 8px 18px rgba(31, 21, 41, 0.12);
}

.travel-map-card {
  display: none;
}

.travel-map-card.is-active {
  display: block;
}

.travel-map-list {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-top: 1rem;
}

.travel-map-list button {
  border: 1px solid #dec9ed;
  border-radius: 999px;
  padding: 0.35rem 0.75rem;
  background: #fff;
  color: #5c3a73;
  cursor: pointer;
}

.travel-map-list button.is-active {
  background: DarkViolet;
  color: #fff;
  border-color: DarkViolet;
}

.world-map-land {
  fill: #dfd0eb;
  stroke: #ffffff;
  stroke-width: 1.2;
}

.world-map-grid {
  fill: none;
  stroke: rgba(100, 76, 120, 0.16);
  stroke-width: 0.7;
}

@media (max-width: 900px) {
  .travel-map-shell {
    grid-template-columns: 1fr;
  }
}
</style>

<div class="travel-map-page">
  <p class="travel-map-intro">
    A small visual diary built from places, dates, and photos. Click a dot on the world map to revisit that moment.
  </p>

  <div class="travel-map-shell" data-travel-map>
    <div class="travel-map-canvas" aria-label="World map with photo locations">
      <svg viewBox="0 0 1000 520" role="img" aria-labelledby="world-map-title world-map-desc">
        <title id="world-map-title">World map base</title>
        <desc id="world-map-desc">A decorative world map used as the base layer for clickable travel photo locations.</desc>
        <path class="world-map-grid" d="M80 80H920M80 160H920M80 240H920M80 320H920M80 400H920M200 50V470M350 50V470M500 50V470M650 50V470M800 50V470" />
        <path class="world-map-land" d="M109 148l50-31 82 5 49 39-5 61 43 33-18 54-57 5-36 47-88-20-26-61 20-50-32-42z" />
        <path class="world-map-land" d="M246 315l45 36 32 75-37 62-45-15-20-69-31-44z" />
        <path class="world-map-land" d="M456 146l43-26 81 10 32 31 60-24 103 9 95 56-37 45 30 49-48 32-101-18-55 34-75-14-71 28-76-32 24-59-38-47z" />
        <path class="world-map-land" d="M493 284l72 12 39 49-14 88-61 30-55-47-21-77z" />
        <path class="world-map-land" d="M720 320l62-18 69 38 42 63-34 49-82-12-55-52z" />
        <path class="world-map-land" d="M392 112l33-20 44 11-3 36-49 12z" />
        <path class="world-map-land" d="M841 103l45-15 39 18 1 30-48 14-39-17z" />
      </svg>

      {% for place in site.data.travel_map %}
        <button
          class="travel-map-marker{% if forloop.first %} is-active{% endif %}"
          type="button"
          style="--x: {{ place.x }}%; --y: {{ place.y }}%;"
          data-place="{{ place.id }}"
          aria-label="{{ place.city }}"
        ></button>
      {% endfor %}
      <span class="travel-map-watermark">Photo Map</span>
    </div>

    <aside class="travel-map-panel" aria-live="polite">
      {% for place in site.data.travel_map %}
        <section class="travel-map-card{% if forloop.first %} is-active{% endif %}" data-card="{{ place.id }}">
          <h2>{{ place.city }}</h2>
          <div class="travel-map-date">{{ place.date }}</div>
          <p class="travel-map-caption">{{ place.description }}</p>
          <div class="travel-map-gallery">
            {% for photo in place.photos %}
              <img src="{{ photo.src }}" alt="{{ photo.alt }}">
            {% endfor %}
          </div>
        </section>
      {% endfor %}

      <div class="travel-map-list" aria-label="Travel location list">
        {% for place in site.data.travel_map %}
          <button class="{% if forloop.first %}is-active{% endif %}" type="button" data-place="{{ place.id }}">{{ place.city }}</button>
        {% endfor %}
      </div>
    </aside>
  </div>
</div>

<script>
(function () {
  var root = document.querySelector('[data-travel-map]');
  if (!root) return;

  var controls = root.querySelectorAll('[data-place]');
  var cards = root.querySelectorAll('[data-card]');

  function selectPlace(placeId) {
    controls.forEach(function (control) {
      control.classList.toggle('is-active', control.getAttribute('data-place') === placeId);
    });

    cards.forEach(function (card) {
      card.classList.toggle('is-active', card.getAttribute('data-card') === placeId);
    });
  }

  controls.forEach(function (control) {
    control.addEventListener('click', function () {
      selectPlace(control.getAttribute('data-place'));
    });
  });
}());
</script>
