{% comment %}
  Store Locator Section med Leaflet kort
  Features: Kort, geolocation, afstandsberegning, by/postnummer søgning
{% endcomment %}

{%- comment -%} Leaflet CSS {%- endcomment -%}
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" crossorigin="" />
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css" crossorigin="" />
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css" crossorigin="" />

<section class="store-locator" data-section-id="{{ section.id }}">
  <div class="prose text-center">
    <h1>{{ section.settings.heading }}</h1>
    <p class="subheading h6">{{ section.settings.subheading }}</p>
  </div>

  <div class="search-container">
    <div class="search-input-wrapper">
      {%- render 'icon' with 'search', width: 20 -%}
      <input type="text"
             class="search-input"
             id="searchInput-{{ section.id }}"
             placeholder="{{ section.settings.search_placeholder }}">
    </div>
    <select class="country-select" id="countrySelect-{{ section.id }}">
      <option value="">{{ section.settings.country_placeholder }}</option>
    </select>
    <button type="button" class="location-btn" id="locationBtn-{{ section.id }}">
      <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><circle cx="12" cy="12" r="3"/><line x1="12" y1="2" x2="12" y2="4"/><line x1="12" y1="20" x2="12" y2="22"/><line x1="2" y1="12" x2="4" y2="12"/><line x1="20" y1="12" x2="22" y2="12"/></svg>
      <span id="locationBtnText-{{ section.id }}">{{ 'store_locator.use_my_location' | t }}</span>
    </button>
  </div>

  <div class="view-toggle" id="viewToggle-{{ section.id }}">
    <button type="button" class="view-btn active" data-view="map">{{ 'store_locator.show_map' | t | default: 'Kort' }}</button>
    <button type="button" class="view-btn" data-view="list">{{ 'store_locator.show_list' | t | default: 'Liste' }}</button>
  </div>

  <div class="results-info" id="resultsInfo-{{ section.id }}"></div>

  <div class="store-locator-content">
    <div id="storeMap-{{ section.id }}" class="store-map"></div>
    <div id="storesContainer-{{ section.id }}" class="stores-list-container"></div>
  </div>
</section>

<style>
.store-locator {
  max-width: 1600px;
  margin: 0 auto;
  padding: 60px 20px;
  font-family: var(--heading-font-family);
  color: #111;
  line-height: 1.6;
}

.search-container {
  margin: 30px auto;
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  justify-content: center;
  align-items: stretch;
}

.search-input,
.country-select {
  font-family: var(--text-font-family);
  padding: 10px 18px;
  border: 1px solid #ddd;
  border-radius: 0;
  font-size: 15px;
  background: #fff;
  min-width: 300px;
  transition: border-color 0.3s;
}

.country-select {
  -webkit-appearance: none;
  appearance: none;
  padding-right: 40px;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath fill='%23666' d='M6 8L0 0h12z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 15px center;
  background-size: 10px;
  cursor: pointer;
}

.search-input-wrapper {
  position: relative;
  display: inline-block;
  min-width: min(310px, 90vw);
}

.search-input {
  width: 100%;
  min-width: 0;
  padding-left: 40px;
}

.search-input-wrapper .Icon--search-desktop {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
  color: #999;
  width: 20px;
  height: 20px;
}

.search-input:focus,
.country-select:focus {
  border-color: #000;
  outline: none;
}

.location-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 18px;
  background: #111;
  color: #fff;
  border: 1px solid #111;
  font-family: var(--text-font-family);
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s ease;
  white-space: nowrap;
}

.location-btn:hover {
  background: #333;
}

.location-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.location-btn.loading {
  pointer-events: none;
}

.location-btn.loading svg {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.view-toggle {
  display: flex;
  justify-content: center;
  gap: 0;
  margin: 20px 0;
}

.view-btn {
  padding: 10px 24px;
  background: #f5f5f5;
  border: 1px solid #ddd;
  font-family: var(--text-font-family);
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.view-btn:first-child {
  border-right: none;
}

.view-btn.active {
  background: #111;
  color: #fff;
  border-color: #111;
}

.results-info {
  text-align: center;
  margin: 15px 0;
  font-size: 14px;
  color: #666;
}

.store-locator-content {
  position: relative;
}

.store-map {
  width: 100%;
  height: 500px;
  margin-bottom: 30px;
  border: 1px solid #ddd;
  z-index: 1;
}

.store-map.hidden {
  display: none;
}

.stores-list-container.hidden {
  display: none;
}

.stores-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 20px;
}

.store-card {
  border-top: 1px solid #eee;
  padding: 20px;
  background: #f5f5f5;
  display: flex;
  flex-direction: column;
  gap: 10px;
  justify-content: space-between;
  animation: fadeIn 0.3s cubic-bezier(0.4, 0, 0.2, 1) both;
  will-change: opacity, transform;
}

.store-name {
  font-family: var(--heading-font-family);
  font-size: 12px;
  letter-spacing: 0.1px;
  text-transform: uppercase;
  margin-bottom: 10px;
}

.store-distance {
  font-size: 12px;
  color: #666;
  margin-bottom: 8px;
  font-weight: 500;
}

.store-info {
  font-family: var(--text-font-family);
  font-size: 12px;
  color: #444;
}

.store-info div {
  margin-bottom: 6px;
}

.store-info a {
  color: inherit;
  text-decoration: none;
  text-transform: uppercase;
  color: #111;
  padding-bottom: 2px;
  background: linear-gradient(to left, rgba(0, 0, 0, 1)),
    linear-gradient(to left, rgba(255, 250, 250, 1));
  background-size: 100% 1px, 0 1px;
  background-position: 0 100%, 100% 100%;
  background-repeat: no-repeat;
  transition: background-size 400ms;
}

.store-info a:hover {
  background-size: 0 1px, 100% 1px;
}

.error-message {
  background: #ffebee;
  color: #c62828;
  padding: 15px;
  border-radius: 4px;
  margin-bottom: 20px;
  text-align: center;
  font-weight: 500;
}

.no-results {
  text-align: center;
  padding: 50px 20px;
  color: #777;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translate3d(0, 20px, 0);
  }
  to {
    opacity: 1;
    transform: translate3d(0, 0, 0);
  }
}

@media (prefers-reduced-motion: reduce) {
  .store-card {
    animation: none !important;
    opacity: 1;
    transform: none;
  }
}

/* Leaflet popup styling */
.leaflet-popup-content-wrapper {
  border-radius: 0;
  font-family: var(--text-font-family);
}

.leaflet-popup-content {
  margin: 12px 16px;
  font-size: 13px;
  line-height: 1.5;
}

.leaflet-popup-content .popup-title {
  font-family: var(--heading-font-family);
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 8px;
}

.leaflet-popup-content .popup-distance {
  font-size: 11px;
  color: #666;
  margin-bottom: 6px;
}

.leaflet-popup-content a {
  color: #111;
  text-decoration: underline;
}

/* User location marker */
.user-location-marker {
  background: #4285f4;
  border: 3px solid #fff;
  border-radius: 50%;
  box-shadow: 0 2px 6px rgba(0,0,0,0.3);
}

/* Mobil styling */
@media (max-width: 768px) {
  .search-container {
    flex-direction: column;
    align-items: stretch;
  }

  .search-input-wrapper,
  .country-select,
  .location-btn {
    width: 100%;
    min-width: 100%;
  }

  .store-map {
    height: 350px;
  }
}
</style>

<script src="https://cdnjs.cloudflare.com/ajax/libs/fuse.js/7.0.0/fuse.min.js" async></script>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" crossorigin=""></script>
<script src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js" crossorigin=""></script>

<script>
document.addEventListener("DOMContentLoaded", () => {
  const sectionId = "{{ section.id }}";
  const input = document.getElementById("searchInput-" + sectionId);
  const select = document.getElementById("countrySelect-" + sectionId);
  const locationBtn = document.getElementById("locationBtn-" + sectionId);
  const locationBtnText = document.getElementById("locationBtnText-" + sectionId);
  const resultsInfo = document.getElementById("resultsInfo-" + sectionId);
  const container = document.getElementById("storesContainer-" + sectionId);
  const mapContainer = document.getElementById("storeMap-" + sectionId);
  const viewToggle = document.getElementById("viewToggle-" + sectionId);

  // Translations
  const TRANSLATIONS = {
    useMyLocation: "{{ 'store_locator.use_my_location' | t | default: 'Brug min placering' }}",
    searching: "{{ 'store_locator.searching' | t | default: 'Søger...' }}",
    locationDenied: "{{ 'store_locator.location_denied' | t | default: 'Placeringsadgang afvist' }}",
    locationTimeout: "{{ 'store_locator.location_timeout' | t | default: 'Timeout ved hentning af placering' }}",
    geolocationNotSupported: "{{ 'store_locator.geolocation_not_supported' | t | default: 'Geolocation understøttes ikke' }}",
    yourLocation: "{{ 'store_locator.your_location' | t | default: 'Din placering' }}",
    noStoresFound: "{{ 'store_locator.no_stores_found' | t | default: 'Ingen butikker fundet' }}",
    tryDifferentSearch: "{{ 'store_locator.try_different_search' | t | default: 'Prøv at ændre søgning' }}",
    nameNotAvailable: "{{ 'store_locator.name_not_available' | t | default: 'Navn ikke tilgængeligt' }}",
    getDirections: "{{ 'store_locator.get_directions' | t | default: 'Få vejvisning' }}",
    storesFoundOne: "{{ 'store_locator.stores_found.one' | t }}",
    storesFoundOther: "{{ 'store_locator.stores_found.other' | t }}"
  };

  // Vent på at Fuse.js er loaded
  function waitForFuse() {
    return new Promise((resolve) => {
      if (typeof Fuse !== 'undefined') {
        resolve();
      } else {
        const checkInterval = setInterval(() => {
          if (typeof Fuse !== 'undefined') {
            clearInterval(checkInterval);
            resolve();
          }
        }, 50);
      }
    });
  }

  // Vent på at Leaflet er loaded
  function waitForLeaflet() {
    return new Promise((resolve) => {
      if (typeof L !== 'undefined') {
        resolve();
      } else {
        const checkInterval = setInterval(() => {
          if (typeof L !== 'undefined') {
            clearInterval(checkInterval);
            resolve();
          }
        }, 50);
      }
    });
  }

  const CONFIG = {
    STORES_URL: 'https://cdn.jsdelivr.net/gh/NicoBang/sebra-store-locator@main/stores.min.json',
    CACHE_DURATION: 3600000,
    SESSION_CACHE_KEY: 'storeLocatorData',
    LOCATION_CACHE_KEY: 'userLocation',
    IPSTACK_API_KEY: "{{ settings.api_ipstack }}",
    NOMINATIM_COUNTRIES: 'dk,de,se,no,nl,fr,be,gb,at,ch,pl,it,es,pt,fi,is',
    DEFAULT_CENTER: [55.6761, 12.5683], // København
    DEFAULT_ZOOM: 5,
    COUNTRY_NAMES: {
      'af': "{{ 'store_locator.countries.af' | t }}",
      'al': "{{ 'store_locator.countries.al' | t }}",
      'dz': "{{ 'store_locator.countries.dz' | t }}",
      'as': "{{ 'store_locator.countries.as' | t }}",
      'ad': "{{ 'store_locator.countries.ad' | t }}",
      'ao': "{{ 'store_locator.countries.ao' | t }}",
      'ai': "{{ 'store_locator.countries.ai' | t }}",
      'ag': "{{ 'store_locator.countries.ag' | t }}",
      'ar': "{{ 'store_locator.countries.ar' | t }}",
      'am': "{{ 'store_locator.countries.am' | t }}",
      'aw': "{{ 'store_locator.countries.aw' | t }}",
      'au': "{{ 'store_locator.countries.au' | t }}",
      'at': "{{ 'store_locator.countries.at' | t }}",
      'az': "{{ 'store_locator.countries.az' | t }}",
      'bs': "{{ 'store_locator.countries.bs' | t }}",
      'bh': "{{ 'store_locator.countries.bh' | t }}",
      'bd': "{{ 'store_locator.countries.bd' | t }}",
      'bb': "{{ 'store_locator.countries.bb' | t }}",
      'by': "{{ 'store_locator.countries.by' | t }}",
      'be': "{{ 'store_locator.countries.be' | t }}",
      'bz': "{{ 'store_locator.countries.bz' | t }}",
      'bj': "{{ 'store_locator.countries.bj' | t }}",
      'bm': "{{ 'store_locator.countries.bm' | t }}",
      'bt': "{{ 'store_locator.countries.bt' | t }}",
      'bo': "{{ 'store_locator.countries.bo' | t }}",
      'bq': "{{ 'store_locator.countries.bq' | t }}",
      'ba': "{{ 'store_locator.countries.ba' | t }}",
      'bw': "{{ 'store_locator.countries.bw' | t }}",
      'bv': "{{ 'store_locator.countries.bv' | t }}",
      'br': "{{ 'store_locator.countries.br' | t }}",
      'io': "{{ 'store_locator.countries.io' | t }}",
      'vg': "{{ 'store_locator.countries.vg' | t }}",
      'bn': "{{ 'store_locator.countries.bn' | t }}",
      'bg': "{{ 'store_locator.countries.bg' | t }}",
      'bf': "{{ 'store_locator.countries.bf' | t }}",
      'bi': "{{ 'store_locator.countries.bi' | t }}",
      'cv': "{{ 'store_locator.countries.cv' | t }}",
      'kh': "{{ 'store_locator.countries.kh' | t }}",
      'cm': "{{ 'store_locator.countries.cm' | t }}",
      'ca': "{{ 'store_locator.countries.ca' | t }}",
      'ky': "{{ 'store_locator.countries.ky' | t }}",
      'cf': "{{ 'store_locator.countries.cf' | t }}",
      'td': "{{ 'store_locator.countries.td' | t }}",
      'cl': "{{ 'store_locator.countries.cl' | t }}",
      'cn': "{{ 'store_locator.countries.cn' | t }}",
      'cx': "{{ 'store_locator.countries.cx' | t }}",
      'cc': "{{ 'store_locator.countries.cc' | t }}",
      'co': "{{ 'store_locator.countries.co' | t }}",
      'km': "{{ 'store_locator.countries.km' | t }}",
      'cg': "{{ 'store_locator.countries.cg' | t }}",
      'cd': "{{ 'store_locator.countries.cd' | t }}",
      'ck': "{{ 'store_locator.countries.ck' | t }}",
      'cr': "{{ 'store_locator.countries.cr' | t }}",
      'ci': "{{ 'store_locator.countries.ci' | t }}",
      'hr': "{{ 'store_locator.countries.hr' | t }}",
      'cu': "{{ 'store_locator.countries.cu' | t }}",
      'cw': "{{ 'store_locator.countries.cw' | t }}",
      'cy': "{{ 'store_locator.countries.cy' | t }}",
      'cz': "{{ 'store_locator.countries.cz' | t }}",
      'dk': "{{ 'store_locator.countries.dk' | t }}",
      'dj': "{{ 'store_locator.countries.dj' | t }}",
      'dm': "{{ 'store_locator.countries.dm' | t }}",
      'do': "{{ 'store_locator.countries.do' | t }}",
      'ec': "{{ 'store_locator.countries.ec' | t }}",
      'eg': "{{ 'store_locator.countries.eg' | t }}",
      'sv': "{{ 'store_locator.countries.sv' | t }}",
      'gq': "{{ 'store_locator.countries.gq' | t }}",
      'er': "{{ 'store_locator.countries.er' | t }}",
      'ee': "{{ 'store_locator.countries.ee' | t }}",
      'sz': "{{ 'store_locator.countries.sz' | t }}",
      'et': "{{ 'store_locator.countries.et' | t }}",
      'fk': "{{ 'store_locator.countries.fk' | t }}",
      'fo': "{{ 'store_locator.countries.fo' | t }}",
      'fj': "{{ 'store_locator.countries.fj' | t }}",
      'fi': "{{ 'store_locator.countries.fi' | t }}",
      'fr': "{{ 'store_locator.countries.fr' | t }}",
      'gf': "{{ 'store_locator.countries.gf' | t }}",
      'pf': "{{ 'store_locator.countries.pf' | t }}",
      'tf': "{{ 'store_locator.countries.tf' | t }}",
      'ga': "{{ 'store_locator.countries.ga' | t }}",
      'gm': "{{ 'store_locator.countries.gm' | t }}",
      'ge': "{{ 'store_locator.countries.ge' | t }}",
      'de': "{{ 'store_locator.countries.de' | t }}",
      'gh': "{{ 'store_locator.countries.gh' | t }}",
      'gi': "{{ 'store_locator.countries.gi' | t }}",
      'gr': "{{ 'store_locator.countries.gr' | t }}",
      'gl': "{{ 'store_locator.countries.gl' | t }}",
      'gd': "{{ 'store_locator.countries.gd' | t }}",
      'gp': "{{ 'store_locator.countries.gp' | t }}",
      'gu': "{{ 'store_locator.countries.gu' | t }}",
      'gt': "{{ 'store_locator.countries.gt' | t }}",
      'gg': "{{ 'store_locator.countries.gg' | t }}",
      'gn': "{{ 'store_locator.countries.gn' | t }}",
      'gw': "{{ 'store_locator.countries.gw' | t }}",
      'gy': "{{ 'store_locator.countries.gy' | t }}",
      'ht': "{{ 'store_locator.countries.ht' | t }}",
      'hm': "{{ 'store_locator.countries.hm' | t }}",
      'va': "{{ 'store_locator.countries.va' | t }}",
      'hn': "{{ 'store_locator.countries.hn' | t }}",
      'hk': "{{ 'store_locator.countries.hk' | t }}",
      'hu': "{{ 'store_locator.countries.hu' | t }}",
      'is': "{{ 'store_locator.countries.is' | t }}",
      'in': "{{ 'store_locator.countries.in' | t }}",
      'id': "{{ 'store_locator.countries.id' | t }}",
      'ir': "{{ 'store_locator.countries.ir' | t }}",
      'iq': "{{ 'store_locator.countries.iq' | t }}",
      'ie': "{{ 'store_locator.countries.ie' | t }}",
      'im': "{{ 'store_locator.countries.im' | t }}",
      'il': "{{ 'store_locator.countries.il' | t }}",
      'it': "{{ 'store_locator.countries.it' | t }}",
      'jm': "{{ 'store_locator.countries.jm' | t }}",
      'jp': "{{ 'store_locator.countries.jp' | t }}",
      'je': "{{ 'store_locator.countries.je' | t }}",
      'jo': "{{ 'store_locator.countries.jo' | t }}",
      'kz': "{{ 'store_locator.countries.kz' | t }}",
      'ke': "{{ 'store_locator.countries.ke' | t }}",
      'ki': "{{ 'store_locator.countries.ki' | t }}",
      'kp': "{{ 'store_locator.countries.kp' | t }}",
      'kr': "{{ 'store_locator.countries.kr' | t }}",
      'kw': "{{ 'store_locator.countries.kw' | t }}",
      'kg': "{{ 'store_locator.countries.kg' | t }}",
      'la': "{{ 'store_locator.countries.la' | t }}",
      'lv': "{{ 'store_locator.countries.lv' | t }}",
      'lb': "{{ 'store_locator.countries.lb' | t }}",
      'ls': "{{ 'store_locator.countries.ls' | t }}",
      'lr': "{{ 'store_locator.countries.lr' | t }}",
      'ly': "{{ 'store_locator.countries.ly' | t }}",
      'li': "{{ 'store_locator.countries.li' | t }}",
      'lt': "{{ 'store_locator.countries.lt' | t }}",
      'lu': "{{ 'store_locator.countries.lu' | t }}",
      'mo': "{{ 'store_locator.countries.mo' | t }}",
      'mg': "{{ 'store_locator.countries.mg' | t }}",
      'mw': "{{ 'store_locator.countries.mw' | t }}",
      'my': "{{ 'store_locator.countries.my' | t }}",
      'mv': "{{ 'store_locator.countries.mv' | t }}",
      'ml': "{{ 'store_locator.countries.ml' | t }}",
      'mt': "{{ 'store_locator.countries.mt' | t }}",
      'mh': "{{ 'store_locator.countries.mh' | t }}",
      'mq': "{{ 'store_locator.countries.mq' | t }}",
      'mr': "{{ 'store_locator.countries.mr' | t }}",
      'mu': "{{ 'store_locator.countries.mu' | t }}",
      'yt': "{{ 'store_locator.countries.yt' | t }}",
      'mx': "{{ 'store_locator.countries.mx' | t }}",
      'fm': "{{ 'store_locator.countries.fm' | t }}",
      'md': "{{ 'store_locator.countries.md' | t }}",
      'mc': "{{ 'store_locator.countries.mc' | t }}",
      'mn': "{{ 'store_locator.countries.mn' | t }}",
      'me': "{{ 'store_locator.countries.me' | t }}",
      'ms': "{{ 'store_locator.countries.ms' | t }}",
      'ma': "{{ 'store_locator.countries.ma' | t }}",
      'mz': "{{ 'store_locator.countries.mz' | t }}",
      'mm': "{{ 'store_locator.countries.mm' | t }}",
      'na': "{{ 'store_locator.countries.na' | t }}",
      'nr': "{{ 'store_locator.countries.nr' | t }}",
      'np': "{{ 'store_locator.countries.np' | t }}",
      'nl': "{{ 'store_locator.countries.nl' | t }}",
      'nc': "{{ 'store_locator.countries.nc' | t }}",
      'nz': "{{ 'store_locator.countries.nz' | t }}",
      'ni': "{{ 'store_locator.countries.ni' | t }}",
      'ne': "{{ 'store_locator.countries.ne' | t }}",
      'ng': "{{ 'store_locator.countries.ng' | t }}",
      'nu': "{{ 'store_locator.countries.nu' | t }}",
      'nf': "{{ 'store_locator.countries.nf' | t }}",
      'mk': "{{ 'store_locator.countries.mk' | t }}",
      'mp': "{{ 'store_locator.countries.mp' | t }}",
      'no': "{{ 'store_locator.countries.no' | t }}",
      'om': "{{ 'store_locator.countries.om' | t }}",
      'pk': "{{ 'store_locator.countries.pk' | t }}",
      'pw': "{{ 'store_locator.countries.pw' | t }}",
      'ps': "{{ 'store_locator.countries.ps' | t }}",
      'pa': "{{ 'store_locator.countries.pa' | t }}",
      'pg': "{{ 'store_locator.countries.pg' | t }}",
      'py': "{{ 'store_locator.countries.py' | t }}",
      'pe': "{{ 'store_locator.countries.pe' | t }}",
      'ph': "{{ 'store_locator.countries.ph' | t }}",
      'pn': "{{ 'store_locator.countries.pn' | t }}",
      'pl': "{{ 'store_locator.countries.pl' | t }}",
      'pt': "{{ 'store_locator.countries.pt' | t }}",
      'pr': "{{ 'store_locator.countries.pr' | t }}",
      'qa': "{{ 'store_locator.countries.qa' | t }}",
      're': "{{ 'store_locator.countries.re' | t }}",
      'ro': "{{ 'store_locator.countries.ro' | t }}",
      'ru': "{{ 'store_locator.countries.ru' | t }}",
      'rw': "{{ 'store_locator.countries.rw' | t }}",
      'bl': "{{ 'store_locator.countries.bl' | t }}",
      'sh': "{{ 'store_locator.countries.sh' | t }}",
      'kn': "{{ 'store_locator.countries.kn' | t }}",
      'lc': "{{ 'store_locator.countries.lc' | t }}",
      'mf': "{{ 'store_locator.countries.mf' | t }}",
      'pm': "{{ 'store_locator.countries.pm' | t }}",
      'vc': "{{ 'store_locator.countries.vc' | t }}",
      'ws': "{{ 'store_locator.countries.ws' | t }}",
      'sm': "{{ 'store_locator.countries.sm' | t }}",
      'st': "{{ 'store_locator.countries.st' | t }}",
      'sa': "{{ 'store_locator.countries.sa' | t }}",
      'sn': "{{ 'store_locator.countries.sn' | t }}",
      'rs': "{{ 'store_locator.countries.rs' | t }}",
      'sc': "{{ 'store_locator.countries.sc' | t }}",
      'sl': "{{ 'store_locator.countries.sl' | t }}",
      'sg': "{{ 'store_locator.countries.sg' | t }}",
      'sx': "{{ 'store_locator.countries.sx' | t }}",
      'sk': "{{ 'store_locator.countries.sk' | t }}",
      'si': "{{ 'store_locator.countries.si' | t }}",
      'sb': "{{ 'store_locator.countries.sb' | t }}",
      'so': "{{ 'store_locator.countries.so' | t }}",
      'za': "{{ 'store_locator.countries.za' | t }}",
      'gs': "{{ 'store_locator.countries.gs' | t }}",
      'ss': "{{ 'store_locator.countries.ss' | t }}",
      'es': "{{ 'store_locator.countries.es' | t }}",
      'lk': "{{ 'store_locator.countries.lk' | t }}",
      'sd': "{{ 'store_locator.countries.sd' | t }}",
      'sr': "{{ 'store_locator.countries.sr' | t }}",
      'sj': "{{ 'store_locator.countries.sj' | t }}",
      'se': "{{ 'store_locator.countries.se' | t }}",
      'ch': "{{ 'store_locator.countries.ch' | t }}",
      'sy': "{{ 'store_locator.countries.sy' | t }}",
      'tw': "{{ 'store_locator.countries.tw' | t }}",
      'tj': "{{ 'store_locator.countries.tj' | t }}",
      'tz': "{{ 'store_locator.countries.tz' | t }}",
      'th': "{{ 'store_locator.countries.th' | t }}",
      'tl': "{{ 'store_locator.countries.tl' | t }}",
      'tg': "{{ 'store_locator.countries.tg' | t }}",
      'tk': "{{ 'store_locator.countries.tk' | t }}",
      'to': "{{ 'store_locator.countries.to' | t }}",
      'tt': "{{ 'store_locator.countries.tt' | t }}",
      'tn': "{{ 'store_locator.countries.tn' | t }}",
      'tr': "{{ 'store_locator.countries.tr' | t }}",
      'tm': "{{ 'store_locator.countries.tm' | t }}",
      'tc': "{{ 'store_locator.countries.tc' | t }}",
      'tv': "{{ 'store_locator.countries.tv' | t }}",
      'ug': "{{ 'store_locator.countries.ug' | t }}",
      'ua': "{{ 'store_locator.countries.ua' | t }}",
      'ae': "{{ 'store_locator.countries.ae' | t }}",
      'gb': "{{ 'store_locator.countries.gb' | t }}",
      'us': "{{ 'store_locator.countries.us' | t }}",
      'um': "{{ 'store_locator.countries.um' | t }}",
      'uy': "{{ 'store_locator.countries.uy' | t }}",
      'uz': "{{ 'store_locator.countries.uz' | t }}",
      'vu': "{{ 'store_locator.countries.vu' | t }}",
      've': "{{ 'store_locator.countries.ve' | t }}",
      'vn': "{{ 'store_locator.countries.vn' | t }}",
      'wf': "{{ 'store_locator.countries.wf' | t }}",
      'eh': "{{ 'store_locator.countries.eh' | t }}",
      'ye': "{{ 'store_locator.countries.ye' | t }}",
      'zm': "{{ 'store_locator.countries.zm' | t }}",
      'zw': "{{ 'store_locator.countries.zw' | t }}"
    }
  };

  let allStores = [];
  let filteredStores = [];
  let currentFilter = "all";
  let currentView = "map";
  let fuse;
  let map;
  let markerClusterGroup;
  let userMarker;
  let userPosition = null;

  initializeStoreLocator();

  async function initializeStoreLocator() {
    await Promise.all([waitForFuse(), waitForLeaflet()]);
    await loadStores();
    initializeMap();
    initializeSearch();
    initializeViewToggle();
    initializeLocationButton();
    setSearchInputWidth();
    const countryCode = await setUserCountry();
    if (countryCode) {
      searchStores();
    }
  }

  function initializeMap() {
    map = L.map(mapContainer).setView(CONFIG.DEFAULT_CENTER, CONFIG.DEFAULT_ZOOM);

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>',
      maxZoom: 19
    }).addTo(map);

    markerClusterGroup = L.markerClusterGroup({
      maxClusterRadius: 50,
      spiderfyOnMaxZoom: true,
      showCoverageOnHover: false,
      zoomToBoundsOnClick: true
    });
    map.addLayer(markerClusterGroup);
  }

  function updateMapMarkers(stores) {
    markerClusterGroup.clearLayers();

    const storesWithCoords = stores.filter(s => s.Latitude && s.Longitude);

    storesWithCoords.forEach(store => {
      const lat = parseFloat(store.Latitude);
      const lng = parseFloat(store.Longitude);

      if (isNaN(lat) || isNaN(lng)) return;

      const distanceHtml = store.distance !== undefined
        ? `<div class="popup-distance">${formatDistance(store.distance)}</div>`
        : '';

      const popupContent = `
        <div class="popup-title">${store.Company || TRANSLATIONS.nameNotAvailable}</div>
        ${distanceHtml}
        <div>${store.Address || ''}</div>
        <div>${store["Postal code"] || store["Postal Code"] || ''} ${store.City || ''}</div>
        ${store.Phone ? `<div>${store.Phone}</div>` : ''}
        <div style="margin-top: 8px;">
          <a href="https://www.google.com/maps/dir/?api=1&destination=${encodeURIComponent((store.Address || '') + ', ' + (store["Postal code"] || store["Postal Code"] || '') + ' ' + (store.City || ''))}" target="_blank">${TRANSLATIONS.getDirections}</a>
        </div>
      `;

      const marker = L.marker([lat, lng])
        .bindPopup(popupContent);

      markerClusterGroup.addLayer(marker);
    });

    // Fit bounds hvis der er markers
    if (storesWithCoords.length > 0) {
      const bounds = markerClusterGroup.getBounds();
      if (bounds.isValid()) {
        map.fitBounds(bounds, { padding: [50, 50], maxZoom: 12 });
      }
    }
  }

  function initializeViewToggle() {
    const buttons = viewToggle.querySelectorAll('.view-btn');
    buttons.forEach(btn => {
      btn.addEventListener('click', () => {
        buttons.forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        currentView = btn.dataset.view;

        if (currentView === 'map') {
          mapContainer.classList.remove('hidden');
          container.classList.add('hidden');
          map.invalidateSize();
        } else {
          mapContainer.classList.add('hidden');
          container.classList.remove('hidden');
        }
      });
    });
  }

  function initializeLocationButton() {
    locationBtn.addEventListener('click', getUserLocation);
  }

  function getUserLocation() {
    if (!navigator.geolocation) {
      locationBtnText.textContent = TRANSLATIONS.geolocationNotSupported;
      return;
    }

    locationBtn.classList.add('loading');
    locationBtnText.textContent = TRANSLATIONS.searching;

    navigator.geolocation.getCurrentPosition(
      (position) => {
        const { latitude, longitude } = position.coords;
        userPosition = { lat: latitude, lng: longitude };

        // Tilføj/opdater bruger marker
        if (userMarker) {
          userMarker.setLatLng([latitude, longitude]);
        } else {
          const userIcon = L.divIcon({
            className: 'user-location-marker',
            iconSize: [16, 16],
            iconAnchor: [8, 8]
          });
          userMarker = L.marker([latitude, longitude], { icon: userIcon })
            .addTo(map)
            .bindPopup(TRANSLATIONS.yourLocation);
        }

        // Beregn afstande og sorter
        sortStoresByDistance(latitude, longitude);

        // Center kort på bruger
        map.setView([latitude, longitude], 10);

        locationBtn.classList.remove('loading');
        locationBtnText.textContent = TRANSLATIONS.useMyLocation;
      },
      (error) => {
        locationBtn.classList.remove('loading');

        switch (error.code) {
          case error.PERMISSION_DENIED:
            locationBtnText.textContent = TRANSLATIONS.locationDenied;
            break;
          case error.TIMEOUT:
            locationBtnText.textContent = TRANSLATIONS.locationTimeout;
            break;
          default:
            locationBtnText.textContent = TRANSLATIONS.useMyLocation;
        }

        setTimeout(() => {
          locationBtnText.textContent = TRANSLATIONS.useMyLocation;
        }, 3000);
      },
      {
        enableHighAccuracy: true,
        timeout: 10000,
        maximumAge: 300000
      }
    );
  }

  // Haversine formel til afstandsberegning
  function calculateDistance(lat1, lon1, lat2, lon2) {
    const R = 6371; // Jordens radius i km
    const dLat = toRad(lat2 - lat1);
    const dLon = toRad(lon2 - lon1);
    const a = Math.sin(dLat / 2) ** 2 +
              Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) *
              Math.sin(dLon / 2) ** 2;
    return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  }

  function toRad(deg) {
    return deg * Math.PI / 180;
  }

  function formatDistance(km) {
    if (km < 1) {
      return Math.round(km * 1000) + ' m';
    }
    return new Intl.NumberFormat('da-DK', {
      minimumFractionDigits: 1,
      maximumFractionDigits: 1
    }).format(km) + ' km';
  }

  function sortStoresByDistance(userLat, userLng) {
    // Beregn afstand for alle butikker
    allStores.forEach(store => {
      if (store.Latitude && store.Longitude) {
        store.distance = calculateDistance(
          userLat, userLng,
          parseFloat(store.Latitude), parseFloat(store.Longitude)
        );
      } else {
        store.distance = Infinity;
      }
    });

    // Nulstil filtre og søg igen
    select.value = '';
    input.value = '';
    searchStores();
  }

  function setSearchInputWidth() {
    const placeholder = input.placeholder;
    const fontSize = parseFloat(getComputedStyle(input).fontSize);
    const estimatedWidth = (placeholder.length * fontSize * 0.48) + 50;
    const minWidth = Math.max(300, Math.min(estimatedWidth, window.innerWidth * 0.9));
    input.closest('.search-input-wrapper').style.minWidth = minWidth + 'px';
  }

  async function setUserCountry() {
    try {
      const geodata = localStorage.getItem('geodata');
      if (geodata) {
        const location = JSON.parse(geodata);
        if (location.country_code) {
          select.value = location.country_code;
          return location.country_code;
        }
      }

      if (CONFIG.IPSTACK_API_KEY) {
        const url = `https://api.ipstack.com/check?access_key=${CONFIG.IPSTACK_API_KEY}`;
        const response = await fetch(url);
        const data = await response.json();

        if (data.country_code) {
          localStorage.setItem('geodata', JSON.stringify(data));
          window.ipstack = data;
          select.value = data.country_code;
          return data.country_code;
        }
      }
    } catch (error) {
      console.log('Could not detect user country:', error);
    }
    return null;
  }

  async function loadStores() {
    const response = await fetch(CONFIG.STORES_URL);
    const data = await response.json();
    allStores = Array.isArray(data) ? data : (data.stores || []);
    allStores = allStores.map(store => ({
      ...store,
      City: store.City ? store.City.trim() : "",
      Company: store.Company ? store.Company.trim() : "",
      Country: store.Country ? store.Country.trim() : ""
    }));
  }

  function initializeSearch() {
    fuse = new Fuse(allStores, {
      keys: ["City", "Company", "Postal Code", "Postal code", "Address"],
      threshold: 0.4,
      distance: 100,
      minMatchCharLength: 2
    });

    input.addEventListener("input", debounce(handleSearchInput, 300));
    select.addEventListener("change", searchStores);
    populateCountryDropdown();
  }

  function handleSearchInput() {
    const term = input.value.trim();

    // Hvis bruger søger efter by/postnummer, prøv Nominatim
    if (term.length >= 3 && !select.value) {
      searchLocationByQuery(term);
    } else {
      searchStores();
    }
  }

  // Nominatim geocoding for by/postnummer søgning
  async function searchLocationByQuery(query) {
    try {
      const url = `https://nominatim.openstreetmap.org/search?` +
        `q=${encodeURIComponent(query)}&format=json&limit=1&countrycodes=${CONFIG.NOMINATIM_COUNTRIES}`;

      const response = await fetch(url, {
        headers: { 'User-Agent': 'StoreLocator/1.0' }
      });

      const data = await response.json();

      if (data && data[0]) {
        const lat = parseFloat(data[0].lat);
        const lon = parseFloat(data[0].lon);

        // Gem position og sorter
        userPosition = { lat, lng: lon };

        // Beregn afstande
        allStores.forEach(store => {
          if (store.Latitude && store.Longitude) {
            store.distance = calculateDistance(
              lat, lon,
              parseFloat(store.Latitude), parseFloat(store.Longitude)
            );
          } else {
            store.distance = Infinity;
          }
        });

        // Sorter og vis
        filteredStores = [...allStores].sort((a, b) => (a.distance || Infinity) - (b.distance || Infinity));
        displayStores(filteredStores);
        updateMapMarkers(filteredStores);

        // Center kort
        map.setView([lat, lon], 10);

        return;
      }
    } catch (error) {
      console.log('Nominatim search error:', error);
    }

    // Fallback til normal søgning
    searchStores();
  }

  function populateCountryDropdown() {
    const countries = {};
    allStores.forEach(s => {
      if (s.Country) countries[s.Country] = (countries[s.Country] || 0) + 1;
    });
    const sorted = Object.entries(countries).sort((a, b) => b[1] - a[1]);
    select.innerHTML = '<option value="">{{ section.settings.country_placeholder }}</option>';
    sorted.forEach(([code, count]) => {
      const name = CONFIG.COUNTRY_NAMES[code.toLowerCase()] || code;
      select.innerHTML += `<option value="${code}">${name} (${count})</option>`;
    });
  }

  function searchStores() {
    const term = input.value.trim();
    const country = select.value;
    let results = [...allStores];

    // Filtrer efter land først
    if (country) {
      results = results.filter(s => s.Country === country);
    }

    // Søg derefter
    if (term) {
      const isPostalCode = /^\d+$/.test(term);

      if (isPostalCode) {
        results = results.filter(s =>
          (s["Postal Code"] && s["Postal Code"].toString().startsWith(term)) ||
          (s["Postal code"] && s["Postal code"].toString().startsWith(term))
        );
      } else {
        const tempFuse = new Fuse(results, {
          keys: ["City", "Company", "Postal Code", "Postal code", "Address"],
          threshold: 0.4,
          distance: 100,
          minMatchCharLength: 2
        });
        results = tempFuse.search(term).map(r => r.item);
      }
    }

    // Sorter efter afstand hvis vi har beregnet det
    if (results.some(s => s.distance !== undefined)) {
      results.sort((a, b) => (a.distance || Infinity) - (b.distance || Infinity));
    }

    filteredStores = results;
    displayStores(results);
    updateMapMarkers(results);
  }

  function displayStores(stores) {
    // Opdater results info
    const storesFoundOneText = TRANSLATIONS.storesFoundOne || '{count} butik fundet';
    const storesFoundOtherText = TRANSLATIONS.storesFoundOther || '{count} butikker fundet';
    let storesFoundText = stores.length === 1
      ? storesFoundOneText.replace('{count}', stores.length)
      : storesFoundOtherText.replace('{count}', stores.length);

    resultsInfo.textContent = stores.length === 0 ? TRANSLATIONS.noStoresFound : storesFoundText;

    if (!stores.length) {
      container.innerHTML = `<div class='no-results'><h3>${TRANSLATIONS.noStoresFound}</h3><p>${TRANSLATIONS.tryDifferentSearch}</p></div>`;
      return;
    }

    container.innerHTML = "<div class='stores-grid'>" + stores.map((s, i) => {
      const distanceHtml = s.distance !== undefined && s.distance !== Infinity
        ? `<div class="store-distance">${formatDistance(s.distance)}</div>`
        : '';

      const postalCode = s["Postal code"] || s["Postal Code"] || '';
      const directionsUrl = `https://www.google.com/maps/dir/?api=1&destination=${encodeURIComponent((s.Address || '') + ', ' + postalCode + ' ' + (s.City || ''))}`;

      return `
        <div class="store-card" style="animation-delay:${i * 0.05}s">
          <div class="part-one-wrapper">
            <div class="store-name">${s.Company || TRANSLATIONS.nameNotAvailable}</div>
            ${distanceHtml}
            <div class="store-info">
              ${s.Address ? `<div>${s.Address}</div>` : ""}
              ${s.City || postalCode ? `<div>${postalCode} ${s.City || ""}</div>` : ""}
              ${s.Phone ? `<div>${s.Phone}</div>` : ""}
              ${s.Address ? `<div><a href="${directionsUrl}" target="_blank">${TRANSLATIONS.getDirections}</a></div>` : ""}
            </div>
          </div>
        </div>
      `;
    }).join("") + "</div>";

    // Fjern will-change efter animation
    container.querySelectorAll('.store-card').forEach(card => {
      card.addEventListener('animationend', () => {
        card.style.willChange = 'auto';
      }, { once: true });
    });
  }

  function debounce(func, delay) {
    let t;
    return function(...a) {
      clearTimeout(t);
      t = setTimeout(() => func.apply(this, a), delay);
    };
  }
});
</script>

{% schema %}
{
  "name": "Store Locator",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "Overskrift",
      "default": "Find Forhandler"
    },
    {
      "type": "text",
      "id": "subheading",
      "label": "Undertekst",
      "default": "Find nærmeste forhandler eller søg efter by"
    },
    {
      "type": "text",
      "id": "search_placeholder",
      "label": "Søgefelt placeholder",
      "default": "Søg efter by, postnummer eller butiksnavn..."
    },
    {
      "type": "text",
      "id": "country_placeholder",
      "label": "Land-dropdown placeholder",
      "default": "Alle lande"
    }
  ],
  "presets": [
    {
      "name": "Store Locator"
    }
  ]
}
{% endschema %}
