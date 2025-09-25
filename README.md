<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Insight Security Dashboard</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
  <link rel="stylesheet" href="https://unpkg.com/leaflet-geosearch/dist/geosearch.css" />
  <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster/dist/MarkerCluster.css" />
  <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster/dist/MarkerCluster.Default.css" />
  <style>
    :root {
      --primary-color: #007acc;
      --danger-color: #ff4444;
      --warning-color: #ffaa00;
      --success-color: #00c851;
      --dark-bg: #0d1b2a;
      --panel-bg: rgba(13, 27, 42, 0.95);
      --text-light: #e0e1dd;
    }
    
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: var(--dark-bg);
      color: var(--text-light);
      overflow: hidden;
      height: 100vh;
    }
    
    #map {
      height: 100vh;
      width: 100vw;
      z-index: 1;
    }
    
    .dashboard-header {
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      z-index: 1000;
      background: linear-gradient(to right, rgba(13, 27, 42, 0.95), rgba(25, 55, 85, 0.9));
      padding: 12px 20px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.5);
      border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    }
    
    .logo-container {
      display: flex;
      align-items: center;
      gap: 15px;
    }
    
    .company-logo {
      height: 45px;
      width: auto;
      border-radius: 4px;
    }
    
    .dashboard-title {
      font-size: 22px;
      font-weight: 600;
      color: white;
      letter-spacing: 0.5px;
    }
    
    .dashboard-subtitle {
      font-size: 14px;
      opacity: 0.8;
      margin-top: 2px;
    }
    
    .header-controls {
      display: flex;
      gap: 15px;
      align-items: center;
    }
    
    .control-panel {
      position: absolute;
      top: 80px;
      left: 20px;
      z-index: 1000;
      background: var(--panel-bg);
      color: white;
      padding: 20px;
      border-radius: 12px;
      width: 360px;
      backdrop-filter: blur(10px);
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
      border: 1px solid rgba(255, 255, 255, 0.1);
      max-height: calc(100vh - 100px);
      overflow-y: auto;
    }
    
    .control-section {
      margin-bottom: 20px;
      padding-bottom: 15px;
      border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    }
    
    .control-section:last-child {
      border-bottom: none;
    }
    
    .section-title {
      font-size: 16px;
      font-weight: 600;
      margin-bottom: 12px;
      color: var(--primary-color);
      display: flex;
      align-items: center;
      gap: 8px;
    }
    
    select, input, button {
      width: 100%;
      padding: 10px 12px;
      margin: 5px 0;
      background: rgba(255, 255, 255, 0.1);
      color: white;
      border: 1px solid rgba(255, 255, 255, 0.2);
      border-radius: 6px;
      font-size: 14px;
      transition: all 0.3s;
    }
    
    select:focus, input:focus {
      outline: none;
      border-color: var(--primary-color);
      box-shadow: 0 0 0 2px rgba(0, 122, 204, 0.2);
    }
    
    select option {
      background: var(--dark-bg);
      color: white;
    }
    
    button {
      background: var(--primary-color);
      cursor: pointer;
      font-weight: 600;
      border: none;
    }
    
    button:hover {
      background: #005a9e;
      transform: translateY(-1px);
    }
    
    button.danger {
      background: var(--danger-color);
    }
    
    button.danger:hover {
      background: #cc0000;
    }
    
    .threat-level {
      padding: 4px 10px;
      border-radius: 4px;
      font-size: 12px;
      font-weight: bold;
      display: inline-block;
    }
    
    .critical { background: var(--danger-color); }
    .high { background: var(--warning-color); color: #333; }
    .medium { background: #ffcc00; color: #333; }
    .low { background: var(--success-color); color: #333; }
    
    .stats-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-top: 15px;
    }
    
    .stat-card {
      background: rgba(255, 255, 255, 0.05);
      padding: 12px;
      border-radius: 6px;
      text-align: center;
    }
    
    .stat-value {
      font-size: 24px;
      font-weight: bold;
      margin: 5px 0;
    }
    
    .critical-count { color: var(--danger-color); }
    .ops-count { color: var(--primary-color); }
    .countries-count { color: var(--success-color); }
    
    .tab-container {
      display: flex;
      background: rgba(255, 255, 255, 0.05);
      border-radius: 6px;
      padding: 4px;
      margin-bottom: 15px;
    }
    
    .tab {
      flex: 1;
      text-align: center;
      padding: 8px;
      cursor: pointer;
      border-radius: 4px;
      transition: all 0.3s;
    }
    
    .tab.active {
      background: var(--primary-color);
    }
    
    .tab-content {
      display: none;
    }
    
    .tab-content.active {
      display: block;
    }
    
    .country-info {
      background: rgba(255, 255, 255, 0.05);
      padding: 12px;
      border-radius: 6px;
      margin-top: 10px;
      font-size: 14px;
    }
    
    .region-list {
      max-height: 200px;
      overflow-y: auto;
      margin-top: 10px;
      background: rgba(255, 255, 255, 0.05);
      border-radius: 6px;
      padding: 5px;
    }
    
    .region-item {
      padding: 8px 10px;
      margin: 3px 0;
      background: rgba(255, 255, 255, 0.05);
      border-radius: 4px;
      cursor: pointer;
      transition: all 0.2s;
      font-size: 14px;
    }
    
    .region-item:hover {
      background: rgba(255, 255, 255, 0.1);
    }
    
    .region-item.active {
      background: var(--primary-color);
    }
    
    .legend {
      position: absolute;
      bottom: 20px;
      right: 20px;
      z-index: 1000;
      background: var(--panel-bg);
      color: white;
      padding: 15px;
      border-radius: 8px;
      width: 200px;
      backdrop-filter: blur(10px);
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
      border: 1px solid rgba(255, 255, 255, 0.1);
    }
    
    .legend-title {
      font-size: 14px;
      font-weight: 600;
      margin-bottom: 10px;
      color: var(--primary-color);
    }
    
    .legend-item {
      display: flex;
      align-items: center;
      margin: 8px 0;
      font-size: 13px;
    }
    
    .legend-color {
      width: 16px;
      height: 16px;
      border-radius: 50%;
      margin-right: 10px;
      border: 2px solid rgba(255, 255, 255, 0.3);
    }
    
    .status-indicator {
      position: absolute;
      bottom: 20px;
      left: 20px;
      z-index: 1000;
      background: var(--panel-bg);
      padding: 10px 15px;
      border-radius: 8px;
      font-size: 12px;
      backdrop-filter: blur(10px);
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
      border: 1px solid rgba(255, 255, 255, 0.1);
      display: flex;
      align-items: center;
      gap: 8px;
    }
    
    .status-dot {
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: var(--success-color);
      animation: pulse 2s infinite;
    }
    
    @keyframes pulse {
      0% { opacity: 1; }
      50% { opacity: 0.5; }
      100% { opacity: 1; }
    }
    
    .map-controls {
      position: absolute;
      top: 80px;
      right: 20px;
      z-index: 1000;
      display: flex;
      flex-direction: column;
      gap: 10px;
    }
    
    .map-control-btn {
      width: 40px;
      height: 40px;
      border-radius: 6px;
      background: var(--panel-bg);
      border: 1px solid rgba(255, 255, 255, 0.1);
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      transition: all 0.3s;
      backdrop-filter: blur(10px);
    }
    
    .map-control-btn:hover {
      background: var(--primary-color);
      transform: scale(1.05);
    }
    
    /* Custom scrollbar */
    ::-webkit-scrollbar {
      width: 6px;
    }
    
    ::-webkit-scrollbar-track {
      background: rgba(255, 255, 255, 0.05);
      border-radius: 3px;
    }
    
    ::-webkit-scrollbar-thumb {
      background: var(--primary-color);
      border-radius: 3px;
    }
    
    /* Responsive adjustments */
    @media (max-width: 768px) {
      .control-panel {
        width: 300px;
        top: 70px;
        left: 10px;
      }
      
      .dashboard-title {
        font-size: 18px;
      }
      
      .dashboard-header {
        padding: 10px 15px;
      }
    }
    
    /* Loading animation for GADM data */
    .loading-overlay {
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: rgba(0, 0, 0, 0.7);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 2000;
      flex-direction: column;
      gap: 15px;
    }
    
    .spinner {
      width: 50px;
      height: 50px;
      border: 5px solid rgba(255, 255, 255, 0.3);
      border-radius: 50%;
      border-top-color: var(--primary-color);
      animation: spin 1s ease-in-out infinite;
    }
    
    @keyframes spin {
      to { transform: rotate(360deg); }
    }
  </style>
</head>
<body>
  <div class="dashboard-header">
    <div class="logo-container">
      <img class="company-logo" src="https://raw.githubusercontent.com/laansales999/MappingSystem/main/LOGO.png" alt="Insight Security Logo">
      <div>
        <div class="dashboard-title">Insight Security Dashboard</div>
        <div class="dashboard-subtitle">African Regional Threat Monitoring</div>
      </div>
    </div>
    <div class="header-controls">
      <div id="current-time">--:--:-- UTC</div>
    </div>
  </div>

  <div class="control-panel">
    <div class="control-section">
      <div class="section-title">🌍 Region Selection</div>
      
      <div class="tab-container">
        <div class="tab active" data-tab="countries">Countries</div>
        <div class="tab" data-tab="regions">Regions</div>
      </div>
      
      <div class="tab-content active" id="countries-tab">
        <select id="country-select">
          <option value="">Select African Country</option>
          <option value="CD">Congo (Democratic Republic)</option>
          <option value="KE">Kenya</option>
          <option value="NG">Nigeria</option>
          <option value="ZA">South Africa</option>
          <option value="ET">Ethiopia</option>
          <option value="EG">Egypt</option>
        </select>
        
        <div class="country-info" id="country-info" style="display: none;">
          <div><strong>Capital:</strong> <span id="country-capital">-</span></div>
          <div><strong>Population:</strong> <span id="country-population">-</span></div>
          <div><strong>Security Level:</strong> <span id="country-security">-</span></div>
        </div>
      </div>
      
      <div class="tab-content" id="regions-tab">
        <select id="region-country-select">
          <option value="">Select Country First</option>
          <option value="CD">Congo (DRC)</option>
          <option value="KE">Kenya</option>
        </select>
        
        <select id="region-select" disabled>
          <option value="">Select Region</option>
        </select>
        
        <div class="region-list" id="region-list" style="display: none;">
          <!-- Regions will be populated by JavaScript -->
        </div>
      </div>
    </div>
    
    <div class="control-section">
      <div class="section-title">📍 Location Marking</div>
      
      <input type="text" id="location-search" placeholder="Search city/region/coordinates">
      
      <select id="threat-level">
        <option value="critical">🔴 Critical Threat</option>
        <option value="high">🟡 High Alert</option>
        <option value="medium">🟠 Medium Risk</option>
        <option value="low">🟢 Low Risk</option>
      </select>
      
      <input type="text" id="incident-code" placeholder="Incident Code (e.g., ALPHA-7B)">
      
      <button id="mark-location">Mark Location</button>
    </div>
    
    <div class="control-section">
      <div class="section-title">📊 Operations Dashboard</div>
      
      <div class="stats-grid">
        <div class="stat-card">
          <div>Active Ops</div>
          <div class="stat-value ops-count" id="op-count">0</div>
        </div>
        <div class="stat-card">
          <div>Critical</div>
          <div class="stat-value critical-count" id="critical-count">0</div>
        </div>
        <div class="stat-card">
          <div>Countries</div>
          <div class="stat-value countries-count" id="countries-covered">2</div>
        </div>
        <div class="stat-card">
          <div>Regions</div>
          <div class="stat-value" id="regions-covered">26</div>
        </div>
      </div>
      
      <div style="display: flex; gap: 10px; margin-top: 15px;">
        <button id="satellite-toggle">Toggle Satellite</button>
        <button id="clear-all" class="danger">Clear All</button>
      </div>
    </div>
    
    <div class="control-section">
      <div class="section-title">🔐 System Status</div>
      
      <div style="font-size: 13px; opacity: 0.9;">
        <div><strong>INTEL LEVEL:</strong> CLASSIFIED</div>
        <div><strong>Last Update:</strong> <span id="update-time">--:--:--</span></div>
        <div><strong>Data Source:</strong> GADM v4.1 | AFRICOM</div>
        <div><strong>Coverage:</strong> Congo (DRC), Kenya</div>
      </div>
    </div>
  </div>

  <div class="map-controls">
    <div class="map-control-btn" id="zoom-in" title="Zoom In">+</div>
    <div class="map-control-btn" id="zoom-out" title="Zoom Out">−</div>
    <div class="map-control-btn" id="reset-view" title="Reset View">↻</div>
  </div>

  <div class="legend">
    <div class="legend-title">Threat Level Legend</div>
    <div class="legend-item">
      <div class="legend-color" style="background: #ff4444;"></div>
      <div>Critical Threat</div>
    </div>
    <div class="legend-item">
      <div class="legend-color" style="background: #ffaa00;"></div>
      <div>High Alert</div>
    </div>
    <div class="legend-item">
      <div class="legend-color" style="background: #ffcc00;"></div>
      <div>Medium Risk</div>
    </div>
    <div class="legend-item">
      <div class="legend-color" style="background: #00c851;"></div>
      <div>Low Risk</div>
    </div>
  </div>

  <div class="status-indicator">
    <div class="status-dot"></div>
    <div>SYSTEM ONLINE</div>
  </div>

  <div id="map"></div>

  <!-- Loading overlay for GADM data -->
  <div class="loading-overlay" id="loading-overlay" style="display: none;">
    <div class="spinner"></div>
    <div>Loading GADM Geographic Data...</div>
  </div>

  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script src="https://unpkg.com/leaflet-geosearch/dist/geosearch.umd.js"></script>
  <script src="https://unpkg.com/leaflet.markercluster/dist/leaflet.markercluster.js"></script>
  
  <script>
    // Enhanced African countries data with GADM integration
    const africanCountries = {
      "CD": {
        name: "Congo (Democratic Republic)",
        capital: "Kinshasa",
        population: "101 million",
        securityLevel: "High Risk",
        gadmFile: "https://raw.githubusercontent.com/laansales999/MappingSystem/main/data/gadm41_COD.geojson",
        regions: [
          "Kinshasa", "Kongo Central", "Kwango", "Kwilu", "Mai-Ndombe", "Equateur", 
          "Mongala", "Nord-Ubangi", "Sud-Ubangi", "Tshuapa", "Tshopo", "Bas-Uele", 
          "Haut-Uele", "Ituri", "Maniema", "Nord-Kivu", "Sankuru", "Sud-Kivu", "Tanganyika"
        ]
      },
      "KE": {
        name: "Kenya",
        capital: "Nairobi",
        population: "53 million",
        securityLevel: "Medium Risk",
        gadmFile: "https://raw.githubusercontent.com/laansales999/MappingSystem/main/data/gadm41_KEN.geojson",
        regions: [
          "Nairobi", "Central", "Coast", "Eastern", "North Eastern", "Nyanza", 
          "Rift Valley", "Western"
        ]
      },
      "NG": {
        name: "Nigeria",
        capital: "Abuja",
        population: "206 million",
        securityLevel: "High Risk",
        regions: [
          "Abia", "Adamawa", "Akwa Ibom", "Anambra", "Bauchi", "Bayelsa", "Benue", "Borno"
        ]
      },
      "ZA": {
        name: "South Africa",
        capital: "Pretoria",
        population: "59 million",
        securityLevel: "Medium Risk",
        regions: [
          "Eastern Cape", "Free State", "Gauteng", "KwaZulu-Natal", "Limpopo"
        ]
      },
      "ET": {
        name: "Ethiopia",
        capital: "Addis Ababa",
        population: "118 million",
        securityLevel: "High Risk",
        regions: [
          "Addis Ababa", "Afar", "Amhara", "Benishangul-Gumuz", "Dire Dawa"
        ]
      },
      "EG": {
        name: "Egypt",
        capital: "Cairo",
        population: "104 million",
        securityLevel: "Medium Risk",
        regions: [
          "Alexandria", "Aswan", "Asyut", "Beheira", "Beni Suef", "Cairo"
        ]
      }
    };

    // Initialize the map with a professional theme
    const map = L.map('map', {
      center: [-4, 25],
      zoom: 4,
      zoomControl: false,
      attributionControl: false
    });

    // Add multiple tile layers for different views
    const osmLayer = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© OpenStreetMap contributors'
    }).addTo(map);

    const satelliteLayer = L.tileLayer('https://{s}.google.com/vt/lyrs=s&x={x}&y={y}&z={z}', {
      maxZoom: 20,
      subdomains: ['mt0', 'mt1', 'mt2', 'mt3'],
      attribution: '© Google Maps'
    });

    const darkLayer = L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
      attribution: '© CartoDB'
    });

    // Initialize marker clusters
    const markersCluster = L.markerClusterGroup({
      chunkedLoading: true,
      maxClusterRadius: 50,
      spiderfyOnMaxZoom: true,
      showCoverageOnHover: false
    });
    map.addLayer(markersCluster);

    // Store operations and boundaries
    let operations = [];
    let countryBoundaries = {};
    let currentCountryLayer = null;
    let currentRegionLayer = null;
    let activeCountry = null;
    let activeRegion = null;

    // Initialize geocoder
    const geocoder = new GeoSearch.GeoSearchControl({
      provider: new GeoSearch.OpenStreetMapProvider(),
      style: 'bar',
      searchLabel: 'Enter location',
      showMarker: false,
      autoClose: true
    });
    map.addControl(geocoder);

    // Populate country selects
    const countrySelect = document.getElementById('country-select');
    const regionCountrySelect = document.getElementById('region-country-select');
    
    Object.keys(africanCountries).forEach(code => {
      const option1 = document.createElement('option');
      option1.value = code;
      option1.textContent = africanCountries[code].name;
      countrySelect.appendChild(option1);
      
      if (code === 'CD' || code === 'KE') {
        const option2 = document.createElement('option');
        option2.value = code;
        option2.textContent = africanCountries[code].name;
        regionCountrySelect.appendChild(option2);
      }
    });

    // Update timestamps
    function updateTimestamps() {
      const now = new Date();
      document.getElementById('update-time').textContent = now.toLocaleTimeString();
      document.getElementById('current-time').textContent = now.toUTCString().split(' ')[4] + ' UTC';
    }
    setInterval(updateTimestamps, 1000);
    updateTimestamps();

    // Show/hide loading overlay
    function showLoading() {
      document.getElementById('loading-overlay').style.display = 'flex';
    }
    
    function hideLoading() {
      document.getElementById('loading-overlay').style.display = 'none';
    }

    // Load GADM country boundaries
    async function loadCountryBoundaries(countryCode) {
      try {
        showLoading();
        
        // Remove previous highlights
        if (currentCountryLayer) {
          map.removeLayer(currentCountryLayer);
        }
        if (currentRegionLayer) {
          map.removeLayer(currentRegionLayer);
        }
        
        const countryData = africanCountries[countryCode];
        if (!countryData || !countryData.gadmFile) {
          // Fallback to simplified boundaries for non-GADM countries
          await loadSimplifiedCountry(countryCode);
          hideLoading();
          return;
        }
        
        // Load GADM GeoJSON
        const response = await fetch(countryData.gadmFile);
        const geoData = await response.json();
        
        // Style for the country
        const countryStyle = {
          fillColor: '#ff4444',
          fillOpacity: 0.1,
          color: '#ff4444',
          weight: 3,
          opacity: 0.8
        };
        
        currentCountryLayer = L.geoJSON(geoData, {
          style: countryStyle,
          onEachFeature: function(feature, layer) {
            // Add tooltip with region name
            if (feature.properties.NAME_1) {
              layer.bindTooltip(feature.properties.NAME_1, { 
                permanent: false, 
                direction: 'center',
                className: 'region-tooltip'
              });
            }
          }
        }).addTo(map);
        
        // Zoom to country bounds
        map.fitBounds(currentCountryLayer.getBounds());
        
        // Update country info
        document.getElementById('country-info').style.display = 'block';
        document.getElementById('country-capital').textContent = countryData.capital;
        document.getElementById('country-population').textContent = countryData.population;
        document.getElementById('country-security').textContent = countryData.securityLevel;
        
        activeCountry = countryCode;
        updateStats();
        
        hideLoading();
      } catch (error) {
        console.warn('Could not load GADM boundaries:', error);
        // Fallback to simplified boundaries
        await loadSimplifiedCountry(countryCode);
        hideLoading();
      }
    }

    // Fallback for countries without GADM files
    async function loadSimplifiedCountry(countryCode) {
      try {
        // Use a simplified world boundaries dataset as fallback
        const response = await fetch('https://raw.githubusercontent.com/johan/world.geo.json/master/countries.geo.json');
        const worldData = await response.json();
        
        const country = worldData.features.find(f => 
          f.properties.iso3 === countryCode || f.properties.name === africanCountries[countryCode].name
        );
        
        if (country) {
          currentCountryLayer = L.geoJSON(country, {
            style: {
              fillColor: '#ff4444',
              fillOpacity: 0.1,
              color: '#ff4444',
              weight: 3,
              opacity: 0.8
            }
          }).addTo(map);
          
          map.fitBounds(currentCountryLayer.getBounds());
          
          document.getElementById('country-info').style.display = 'block';
          document.getElementById('country-capital').textContent = africanCountries[countryCode].capital;
          document.getElementById('country-population').textContent = africanCountries[countryCode].population;
          document.getElementById('country-security').textContent = africanCountries[countryCode].securityLevel;
          
          activeCountry = countryCode;
          updateStats();
        }
      } catch (error) {
        console.warn('Could not load simplified boundaries:', error);
      }
    }

    // Load region data from GADM
    async function loadRegionData(countryCode, regionName) {
      try {
        showLoading();
        
        if (currentRegionLayer) {
          map.removeLayer(currentRegionLayer);
        }
        
        const countryData = africanCountries[countryCode];
        if (!countryData || !countryData.gadmFile) {
          // Fallback for non-GADM countries
          loadSimplifiedRegion(countryCode, regionName);
          hideLoading();
          return;
        }
        
        const response = await fetch(countryData.gadmFile);
        const geoData = await response.json();
        
        // Find the specific region
        const regionFeature = geoData.features.find(f => 
          f.properties.NAME_1 === regionName
        );
        
        if (regionFeature) {
          currentRegionLayer = L.geoJSON(regionFeature, {
            style: {
              fillColor: '#007acc',
              fillOpacity: 0.2,
              color: '#007acc',
              weight: 3,
              opacity: 0.8
            }
          }).addTo(map);
          
          // Zoom to region
          map.fitBounds(currentRegionLayer.getBounds());
          
          activeRegion = regionName;
          updateStats();
        }
        
        hideLoading();
      } catch (error) {
        console.warn('Could not load region data:', error);
        loadSimplifiedRegion(countryCode, regionName);
        hideLoading();
      }
    }

    // Fallback for regions without GADM data
    function loadSimplifiedRegion(countryCode, regionName) {
      // Create a simulated region boundary
      const countryCenter = getCountryCenter(countryCode);
      const regionCenter = [
        countryCenter[0] + (Math.random() - 0.5) * 2,
        countryCenter[1] + (Math.random() - 0.5) * 2
      ];
      
      currentRegionLayer = L.circle(regionCenter, {
        color: '#007acc',
        fillColor: '#007acc',
        fillOpacity: 0.1,
        radius: 50000
      }).addTo(map);
      
      L.marker(regionCenter)
        .bindTooltip(regionName, { permanent: true, direction: 'center' })
        .addTo(map);
      
      map.setView(regionCenter, 8);
      activeRegion = regionName;
      updateStats();
    }

    // Helper function to get country centers
    function getCountryCenter(countryCode) {
      const centers = {
        "CD": [-4, 23], // Congo
        "KE": [0, 38],  // Kenya
        "NG": [9, 8],   // Nigeria
        "ZA": [-30, 25], // South Africa
        "ET": [9, 40],  // Ethiopia
        "EG": [27, 30]  // Egypt
      };
      return centers[countryCode] || [0, 0];
    }

    // Mark location function
    function markLocation(lat, lng, locationName, threatLevel, incidentCode) {
      const threatColors = {
        critical: '#ff4444',
        high: '#ffaa00',
        medium: '#ffcc00',
        low: '#00c851'
      };
      
      const threatIcons = {
        critical: '🔴',
        high: '🟡',
        medium: '🟠',
        low: '🟢'
      };

      const customIcon = L.divIcon({
        html: `<div style="background: ${threatColors[threatLevel]}; 
                          width: 20px; 
                          height: 20px; 
                          border-radius: 50%; 
                          border: 3px solid white;
                          box-shadow: 0 0 10px ${threatColors[threatLevel]};"></div>`,
        className: 'threat-marker',
        iconSize: [26, 26]
      });

      const marker = L.marker([lat, lng], { icon: customIcon });
      
      marker.bindPopup(`
        <div style="min-width: 200px;">
          <h4>${threatIcons[threatLevel]} SECURITY ALERT</h4>
          <p><strong>Location:</strong> ${locationName}</p>
          <p><strong>Coordinates:</strong> ${lat.toFixed(4)}, ${lng.toFixed(4)}</p>
          <p><strong>Threat Level:</strong> <span class="threat-level ${threatLevel}">${threatLevel.toUpperCase()}</span></p>
          <p><strong>Incident Code:</strong> ${incidentCode}</p>
          <p><strong>Timestamp:</strong> ${new Date().toLocaleString()}</p>
          <hr>
          <button onclick="zoomToOperation(${operations.length})" style="width:100%; padding:5px;">
            Zoom to Location
          </button>
        </div>
      `);

      markersCluster.addLayer(marker);
      
      const operation = {
        id: operations.length + 1,
        lat: lat,
        lng: lng,
        name: locationName,
        threat: threatLevel,
        code: incidentCode,
        timestamp: new Date(),
        marker: marker
      };
      
      operations.push(operation);
      updateOperationsCount();
      
      if (threatLevel === 'critical') {
        map.setView([lat, lng], 12);
      }
      
      return operation;
    }

    // Update operations counter
    function updateOperationsCount() {
      const opCount = operations.length;
      const criticalCount = operations.filter(op => op.threat === 'critical').length;
      
      document.getElementById('op-count').textContent = opCount;
      document.getElementById('critical-count').textContent = criticalCount;
    }

    // Update overall stats
    function updateStats() {
      // Update regions covered count
      if (activeCountry && africanCountries[activeCountry]) {
        document.getElementById('regions-covered').textContent = 
          africanCountries[activeCountry].regions.length;
      }
    }

    // Global functions for UI interactions
    window.zoomToOperation = function(index) {
      if (operations[index]) {
        const op = operations[index];
        map.setView([op.lat, op.lng], 14);
        op.marker.openPopup();
      }
    };

    // Tab functionality
    document.querySelectorAll('.tab').forEach(tab => {
      tab.addEventListener('click', function() {
        document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
        this.classList.add('active');
        
        document.querySelectorAll('.tab-content').forEach(content => {
          content.classList.remove('active');
        });
        
        const tabId = this.getAttribute('data-tab');
        document.getElementById(`${tabId}-tab`).classList.add('active');
      });
    });

    // Event listeners
    document.getElementById('country-select').addEventListener('change', function() {
      if (this.value) {
        loadCountryBoundaries(this.value);
      } else {
        document.getElementById('country-info').style.display = 'none';
        if (currentCountryLayer) {
          map.removeLayer(currentCountryLayer);
          currentCountryLayer = null;
        }
        activeCountry = null;
        updateStats();
      }
    });

    document.getElementById('region-country-select').addEventListener('change', function() {
      const regionSelect = document.getElementById('region-select');
      const regionList = document.getElementById('region-list');
      
      if (this.value) {
        regionSelect.disabled = false;
        regionSelect.innerHTML = '<option value="">Select Region</option>';
        regionList.innerHTML = '';
        
        const countryData = africanCountries[this.value];
        countryData.regions.forEach(region => {
          const option = document.createElement('option');
          option.value = region;
          option.textContent = region;
          regionSelect.appendChild(option);
          
          const regionItem = document.createElement('div');
          regionItem.className = 'region-item';
          regionItem.textContent = region;
          regionItem.addEventListener('click', function() {
            document.querySelectorAll('.region-item').forEach(item => {
              item.classList.remove('active');
            });
            this.classList.add('active');
            loadRegionData(this.value, region);
          });
          regionList.appendChild(regionItem);
        });
        
        regionList.style.display = 'block';
      } else {
        regionSelect.disabled = true;
        regionSelect.innerHTML = '<option value="">Select Country First</option>';
        regionList.style.display = 'none';
        if (currentRegionLayer) {
          map.removeLayer(currentRegionLayer);
          currentRegionLayer = null;
        }
        activeRegion = null;
        updateStats();
      }
    });

    document.getElementById('region-select').addEventListener('change', function() {
      if (this.value) {
        const countryCode = document.getElementById('region-country-select').value;
        loadRegionData(countryCode, this.value);
      } else {
        if (currentRegionLayer) {
          map.removeLayer(currentRegionLayer);
          currentRegionLayer = null;
        }
        activeRegion = null;
        updateStats();
      }
    });

    document.getElementById('mark-location').addEventListener('click', function() {
      const searchTerm = document.getElementById('location-search').value;
      const threatLevel = document.getElementById('threat-level').value;
      const incidentCode = document.getElementById('incident-code').value || `OP-${Date.now()}`;
      
      if (!searchTerm) {
        alert('Please enter a location');
        return;
      }

      fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(searchTerm)}&limit=1`)
        .then(response => response.json())
        .then(data => {
          if (data && data.length > 0) {
            const lat = parseFloat(data[0].lat);
            const lng = parseFloat(data[0].lon);
            
            markLocation(lat, lng, data[0].display_name, threatLevel, incidentCode);
            document.getElementById('location-search').value = '';
          } else {
            alert('Location not found. Try a more specific search.');
          }
        })
        .catch(error => {
          alert('Geocoding service error. Please try again.');
          console.error(error);
        });
    });

    document.getElementById('satellite-toggle').addEventListener('click', function() {
      if (map.hasLayer(osmLayer)) {
        map.removeLayer(osmLayer);
        satelliteLayer.addTo(map);
        this.textContent = 'Toggle Map View';
      } else {
        map.removeLayer(satelliteLayer);
        osmLayer.addTo(map);
        this.textContent = 'Toggle Satellite View';
      }
    });

    document.getElementById('clear-all').addEventListener('click', function() {
      markersCluster.clearLayers();
      operations = [];
      updateOperationsCount();
      
      if (currentCountryLayer) {
        map.removeLayer(currentCountryLayer);
        currentCountryLayer = null;
      }
      
      if (currentRegionLayer) {
        map.removeLayer(currentRegionLayer);
        currentRegionLayer = null;
      }
      
      activeCountry = null;
      activeRegion = null;
      
      map.setView([-4, 25], 4);
    });

    // Map control buttons
    document.getElementById('zoom-in').addEventListener('click', function() {
      map.zoomIn();
    });

    document.getElementById('zoom-out').addEventListener('click', function() {
      map.zoomOut();
    });

    document.getElementById('reset-view').addEventListener('click', function() {
      map.setView([-4, 25], 4);
    });

    // Right-click to mark location
    map.on('contextmenu', function(e) {
      const threatLevel = document.getElementById('threat-level').value;
      const incidentCode = document.getElementById('incident-code').value || `MAP-${Date.now()}`;
      
      fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=${e.latlng.lat}&lon=${e.latlng.lng}&zoom=10`)
        .then(response => response.json())
        .then(data => {
          const locationName = data.display_name || 'Unknown Location';
          markLocation(e.latlng.lat, e.latlng.lng, locationName, threatLevel, incidentCode);
        });
    });

    // Keyboard shortcuts
    document.addEventListener('keydown', function(e) {
      if (e.ctrlKey && e.key === 'k') {
        e.preventDefault();
        document.getElementById('location-search').focus();
      }
      if (e.key === 'Escape') {
        map.closePopup();
      }
    });

    // Initialize with sample data
    setTimeout(() => {
      markLocation(-4.4419, 15.2663, "Kinshasa, Congo", "critical", "COD-ALPHA-01");
      markLocation(-1.2921, 36.8219, "Nairobi, Kenya", "high", "KEN-BRAVO-02");
      markLocation(0.3136, 32.5811, "Kampala, Uganda", "medium", "UGA-CHARLIE-03");
      
      console.log('🛡️ Insight Security Dashboard Initialized');
      console.log('🌍 GADM Data Integration: Congo & Kenya');
    }, 1000);
  </script>
</body>
</html>
