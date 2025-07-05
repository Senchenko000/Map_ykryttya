<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no"/>
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="theme-color" content="#007bff">
  <title>Укриття по Україні (OSM)</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <style>
    body, html { 
      margin: 0; 
      padding: 0; 
      height: 100%; 
      font-family: Arial, sans-serif;
    }
    
    #map { 
      width: 100%; 
      height: 100vh; 
    }
    
    .sidebar {
      position: absolute;
      top: 10px;
      bottom: 10px;
      width: 280px;
      background: white;
      border-radius: 8px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.3);
      padding: 15px;
      overflow-y: auto;
      z-index: 1000;
    }
    
    /* Мобільні стилі */
    @media (max-width: 768px) {
      .sidebar {
        width: 90%;
        max-width: 320px;
        top: 5px;
        bottom: 5px;
        padding: 10px;
        font-size: 14px;
      }
      
      .sidebar-left {
        left: 5px;
      }
      
      .sidebar-right {
        right: 5px;
        left: 5px;
        width: calc(100% - 10px);
        max-width: none;
        top: auto;
        bottom: 5px;
        height: 40%;
      }
      
      .btn {
        padding: 12px;
        font-size: 16px;
        margin: 8px 0;
      }
      
      .legend-item {
        font-size: 14px;
      }
      
      .legend-icon {
        width: 24px;
        height: 24px;
      }
    }
    
    @media (max-width: 480px) {
      .sidebar {
        width: calc(100% - 10px);
        padding: 8px;
      }
      
      .sidebar h3 {
        font-size: 16px;
      }
    }
    
    .sidebar-left {
      left: 10px;
    }
    
    .sidebar-right {
      right: 10px;
      display: none;
    }
    
    .sidebar h3 {
      margin-top: 0;
      margin-bottom: 15px;
      color: #333;
      border-bottom: 2px solid #007bff;
      padding-bottom: 8px;
    }
    
    .btn {
      width: 100%;
      padding: 10px;
      margin: 5px 0;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-size: 14px;
      transition: background-color 0.3s;
    }
    
    .btn-primary {
      background-color: #007bff;
      color: white;
    }
    
    .btn-primary:hover {
      background-color: #0056b3;
    }
    
    .btn-success {
      background-color: #28a745;
      color: white;
    }
    
    .btn-success:hover {
      background-color: #1e7e34;
    }
    
    .btn-danger {
      background-color: #dc3545;
      color: white;
    }
    
    .btn-danger:hover {
      background-color: #c82333;
    }
    
    .slider-container {
      margin: 15px 0;
    }
    
    .slider-container label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }
    
    .slider {
      width: 100%;
      margin: 10px 0;
    }
    
    .slider-value {
      text-align: center;
      font-weight: bold;
      color: #007bff;
    }
    
    .info-section {
      margin: 15px 0;
      padding: 10px;
      background: #f8f9fa;
      border-radius: 5px;
      border-left: 4px solid #007bff;
    }
    
    .info-section h4 {
      margin-top: 0;
      color: #007bff;
    }
    
    .loading {
      text-align: center;
      padding: 20px;
      background: #fff3cd;
      border: 1px solid #ffeaa7;
      border-radius: 5px;
      margin: 10px 0;
    }
    
    .error {
      background: #f8d7da;
      border: 1px solid #f5c6cb;
      color: #721c24;
      padding: 10px;
      border-radius: 5px;
      margin: 10px 0;
    }
    
    .success {
      background: #d4edda;
      border: 1px solid #c3e6cb;
      color: #155724;
      padding: 10px;
      border-radius: 5px;
      margin: 10px 0;
    }
    
    .route-btn {
      margin-top: 10px;
      padding: 8px 16px;
      background-color: #17a2b8;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      font-size: 12px;
    }
    
    .route-btn:hover {
      background-color: #138496;
    }
    
    .stats {
      background: #e9ecef;
      padding: 10px;
      border-radius: 5px;
      margin: 10px 0;
      text-align: center;
    }
    
    .legend {
      background: white;
      padding: 10px;
      border-radius: 5px;
      margin: 10px 0;
      border: 1px solid #ddd;
    }
    
    .legend-item {
      display: flex;
      align-items: center;
      margin: 5px 0;
      font-size: 12px;
    }
    
    .legend-icon {
      width: 20px;
      height: 20px;
      margin-right: 8px;
    }
  </style>
</head>
<body>
  <div id="map"></div>
  
  <!-- Ліве меню -->
  <div class="sidebar sidebar-left">
    <h3>🛡️ Карта укриттів</h3>
    
    <button class="btn btn-primary" id="findLocation">
      📍 Моє місце розташування
    </button>
    
    <button class="btn btn-success" id="loadEmergencyServices">
      🚨 Підрозділи ДСНС
    </button>
    
    <div class="slider-container">
      <label for="radiusSlider">Радіус пошуку:</label>
      <input type="range" id="radiusSlider" class="slider" min="500" max="5000" value="1500" step="100">
      <div class="slider-value" id="radiusValue">1.5 км</div>
    </div>
    
    <button class="btn btn-primary" id="loadShelters">
      🏠 Завантажити укриття
    </button>
    
    <div class="stats" id="stats" style="display: none;">
      <div>Знайдено укриттів: <span id="shelterCount">0</span></div>
      <div>Підрозділів ДСНС: <span id="emergencyCount">0</span></div>
    </div>
    
    <div class="legend">
      <h4>Легенда:</h4>
      <div class="legend-item">
        <img src="https://cdn-icons-png.flaticon.com/512/684/684908.png" class="legend-icon" alt="Укриття">
        Підземні паркінги
      </div>
      <div class="legend-item">
        <img src="https://cdn-icons-png.flaticon.com/512/2972/2972185.png" class="legend-icon" alt="Метро">
        Станції метро
      </div>
      <div class="legend-item">
        <img src="https://cdn-icons-png.flaticon.com/512/1670/1670074.png" class="legend-icon" alt="Тунель">
        Підземні переходи
      </div>
      <div class="legend-item">
        <img src="https://cdn-icons-png.flaticon.com/512/2913/2913465.png" class="legend-icon" alt="ДСНС">
        Підрозділи ДСНС
      </div>
    </div>
    
    <div id="messages"></div>
  </div>
  
  <!-- Праве меню -->
  <div class="sidebar sidebar-right" id="rightSidebar">
    <h3>ℹ️ Інформація</h3>
    <div id="selectedInfo">
      <p>Оберіть укриття або підрозділ на карті для отримання детальної інформації</p>
    </div>
  </div>

  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    // Створюємо карту
    const map = L.map('map').setView([49.0, 31.0], 6);

    // Підключаємо базову карту OSM
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; OpenStreetMap contributors',
      maxZoom: 18,
    }).addTo(map);

    // Стилі для маркерів
    const icons = {
      underground_parking: L.icon({
        iconUrl: 'https://cdn-icons-png.flaticon.com/512/684/684908.png',
        iconSize: [24, 24],
        iconAnchor: [12, 12],
        popupAnchor: [0, -10]
      }),
      subway: L.icon({
        iconUrl: 'https://cdn-icons-png.flaticon.com/512/2972/2972185.png',
        iconSize: [24, 24],
        iconAnchor: [12, 12],
        popupAnchor: [0, -10]
      }),
      tunnel: L.icon({
        iconUrl: 'https://cdn-icons-png.flaticon.com/512/1670/1670074.png',
        iconSize: [24, 24],
        iconAnchor: [12, 12],
        popupAnchor: [0, -10]
      }),
      emergency: L.icon({
        iconUrl: 'https://cdn-icons-png.flaticon.com/512/2913/2913465.png',
        iconSize: [30, 30],
        iconAnchor: [15, 15],
        popupAnchor: [0, -10]
      }),
      user: L.icon({
        iconUrl: 'https://cdn-icons-png.flaticon.com/512/1077/1077114.png',
        iconSize: [20, 20],
        iconAnchor: [10, 10],
        popupAnchor: [0, -10]
      })
    };

    // Групи маркерів
    const shelterGroup = L.layerGroup().addTo(map);
    const emergencyGroup = L.layerGroup().addTo(map);
    let userLocationMarker = null;
    let searchRadiusCircle = null;
    let userLocation = null;

    // Елементи інтерфейсу
    const messagesDiv = document.getElementById('messages');
    const rightSidebar = document.getElementById('rightSidebar');
    const selectedInfo = document.getElementById('selectedInfo');
    const radiusSlider = document.getElementById('radiusSlider');
    const radiusValue = document.getElementById('radiusValue');
    const shelterCount = document.getElementById('shelterCount');
    const emergencyCount = document.getElementById('emergencyCount');
    const stats = document.getElementById('stats');

    // Функції для повідомлень
    function showMessage(text, type = 'info') {
      const messageDiv = document.createElement('div');
      messageDiv.className = type;
      messageDiv.textContent = text;
      messagesDiv.appendChild(messageDiv);
      
      setTimeout(() => {
        messageDiv.remove();
      }, 5000);
    }

    // Оновлення значення радіуса
    radiusSlider.addEventListener('input', function() {
      const value = this.value;
      radiusValue.textContent = value < 1000 ? `${value} м` : `${(value/1000).toFixed(1)} км`;
      
      if (userLocation && searchRadiusCircle) {
        map.removeLayer(searchRadiusCircle);
        searchRadiusCircle = L.circle(userLocation, {
          radius: parseInt(value),
          color: '#007bff',
          fillColor: '#007bff',
          fillOpacity: 0.1,
          weight: 2
        }).addTo(map);
      }
    });

    // Мобільна оптимізація
    map.on('locationfound', function(e) {
      // Автоматично ставим користувача в центр
      userLocation = [e.latlng.lat, e.latlng.lng];
      
      if (userLocationMarker) {
        map.removeLayer(userLocationMarker);
      }
      
      userLocationMarker = L.marker(userLocation, { icon: icons.user })
        .addTo(map)
        .bindPopup('📍 Ваше місце розташування');
      
      const radius = parseInt(radiusSlider.value);
      if (searchRadiusCircle) {
        map.removeLayer(searchRadiusCircle);
      }
      
      searchRadiusCircle = L.circle(userLocation, {
        radius: radius,
        color: '#007bff',
        fillColor: '#007bff',
        fillOpacity: 0.1,
        weight: 2
      }).addTo(map);
      
      map.setView(userLocation, 14);
      showMessage('Місце розташування визначено!', 'success');
    });

    map.on('locationerror', function(e) {
      showMessage('Помилка геолокації: ' + e.message, 'error');
    });

    // Знаходження місця розташування користувача
    document.getElementById('findLocation').addEventListener('click', function() {
      showMessage('Визначаємо ваше місце розташування...', 'loading');
      
      // Використовуємо Leaflet API для кращої сумісності з мобільними
      map.locate({
        setView: false,
        maxZoom: 16,
        timeout: 10000,
        enableHighAccuracy: true
      });
    });

    // Завантаження укриттів
    document.getElementById('loadShelters').addEventListener('click', function() {
      if (!userLocation) {
        showMessage('Спочатку визначте ваше місце розташування', 'error');
        return;
      }
      
      loadSheltersNearLocation();
    });

    // Завантаження підрозділів ДСНС
    document.getElementById('loadEmergencyServices').addEventListener('click', function() {
      if (!userLocation) {
        showMessage('Спочатку визначте ваше місце розташування', 'error');
        return;
      }
      
      loadEmergencyServices();
    });

    function loadSheltersNearLocation() {
      showMessage('Завантажуємо укриття...', 'loading');
      shelterGroup.clearLayers();
      
      const radius = parseInt(radiusSlider.value);
      const radiusInDegrees = radius / 111000; // Приблизне перетворення метрів в градуси
      
      const bbox = [
        userLocation[0] - radiusInDegrees,
        userLocation[1] - radiusInDegrees,
        userLocation[0] + radiusInDegrees,
        userLocation[1] + radiusInDegrees
      ];

      const query = `
        [out:json][timeout:30];
        (
          node["building"="underground_parking"](${bbox[0]},${bbox[1]},${bbox[2]},${bbox[3]});
          node["railway"="subway_entrance"](${bbox[0]},${bbox[1]},${bbox[2]},${bbox[3]});
          node["highway"="footway"]["tunnel"="yes"](${bbox[0]},${bbox[1]},${bbox[2]},${bbox[3]});
          way["building"="underground_parking"](${bbox[0]},${bbox[1]},${bbox[2]},${bbox[3]});
          way["highway"="footway"]["tunnel"="yes"](${bbox[0]},${bbox[1]},${bbox[2]},${bbox[3]});
        );
        out center;
      `;

      fetch("https://overpass-api.de/api/interpreter", {
        method: "POST",
        body: query
      })
        .then(res => res.json())
        .then(data => {
          let count = 0;
          
          if (data.elements) {
            data.elements.forEach(el => {
              let lat, lon;
              
              if (el.type === 'node') {
                lat = el.lat;
                lon = el.lon;
              } else if (el.type === 'way' && el.center) {
                lat = el.center.lat;
                lon = el.center.lon;
              }
              
              if (lat && lon) {
                // Перевіряємо чи в радіусі
                const distance = calculateDistance(userLocation[0], userLocation[1], lat, lon);
                if (distance <= radius) {
                  const tags = el.tags || {};
                  const name = tags.name || tags['name:uk'] || 'Укриття';
                  let type = 'Укриття';
                  let icon = icons.underground_parking;
                  
                  if (tags.building === 'underground_parking') {
                    type = 'Підземний паркінг';
                    icon = icons.underground_parking;
                  } else if (tags.railway === 'subway_entrance') {
                    type = 'Вхід в метро';
                    icon = icons.subway;
                  } else if (tags.highway === 'footway' && tags.tunnel === 'yes') {
                    type = 'Підземний перехід';
                    icon = icons.tunnel;
                  }
                  
                  const marker = L.marker([lat, lon], { icon: icon })
                    .addTo(shelterGroup);
                  
                  // Додаємо обробник кліку
                  marker.on('click', function() {
                    showShelterInfo(name, type, lat, lon, tags);
                  });
                  
                  count++;
                }
              }
            });
          }
          
          shelterCount.textContent = count;
          stats.style.display = 'block';
          
          if (count > 0) {
            showMessage(`Знайдено ${count} укриттів`, 'success');
          } else {
            showMessage('Укриття не знайдено в заданому радіусі', 'error');
          }
        })
        .catch(err => {
          console.error(err);
          showMessage('Помилка завантаження укриттів', 'error');
        });
    }

    function loadEmergencyServices() {
      showMessage('Завантажуємо підрозділи ДСНС...', 'loading');
      emergencyGroup.clearLayers();
      
      const radius = parseInt(radiusSlider.value);
      const radiusInDegrees = radius / 111000;
      
      const bbox = [
        userLocation[0] - radiusInDegrees,
        userLocation[1] - radiusInDegrees,
        userLocation[0] + radiusInDegrees,
        userLocation[1] + radiusInDegrees
      ];

      const query = `
        [out:json][timeout:30];
        (
          node["amenity"="fire_station"](${bbox[0]},${bbox[1]},${bbox[2]},${bbox[3]});
          node["emergency"="fire_station"](${bbox[0]},${bbox[1]},${bbox[2]},${bbox[3]});
          way["amenity"="fire_station"](${bbox[0]},${bbox[1]},${bbox[2]},${bbox[3]});
          way["emergency"="fire_station"](${bbox[0]},${bbox[1]},${bbox[2]},${bbox[3]});
        );
        out center;
      `;

      fetch("https://overpass-api.de/api/interpreter", {
        method: "POST",
        body: query
      })
        .then(res => res.json())
        .then(data => {
          let count = 0;
          
          if (data.elements) {
            data.elements.forEach(el => {
              let lat, lon;
              
              if (el.type === 'node') {
                lat = el.lat;
                lon = el.lon;
              } else if (el.type === 'way' && el.center) {
                lat = el.center.lat;
                lon = el.center.lon;
              }
              
              if (lat && lon) {
                const distance = calculateDistance(userLocation[0], userLocation[1], lat, lon);
                if (distance <= radius) {
                  const tags = el.tags || {};
                  const name = tags.name || tags['name:uk'] || 'Підрозділ ДСНС';
                  
                  const marker = L.marker([lat, lon], { icon: icons.emergency })
                    .addTo(emergencyGroup);
                  
                  marker.on('click', function() {
                    showEmergencyInfo(name, lat, lon, tags);
                  });
                  
                  count++;
                }
              }
            });
          }
          
          emergencyCount.textContent = count;
          
          if (count > 0) {
            showMessage(`Знайдено ${count} підрозділів ДСНС`, 'success');
          } else {
            showMessage('Підрозділи ДСНС не знайдено в заданому радіусі', 'error');
          }
        })
        .catch(err => {
          console.error(err);
          showMessage('Помилка завантаження підрозділів ДСНС', 'error');
        });
    }

    function showShelterInfo(name, type, lat, lon, tags) {
      rightSidebar.style.display = 'block';
      
      const address = tags['addr:full'] || tags['addr:street'] || 'Адреса не вказана';
      const capacity = tags.capacity || 'Не вказано';
      const distance = userLocation ? calculateDistance(userLocation[0], userLocation[1], lat, lon) : 0;
      
      selectedInfo.innerHTML = `
        <div class="info-section">
          <h4>🏠 ${name}</h4>
          <p><strong>Тип:</strong> ${type}</p>
          <p><strong>Адреса:</strong> ${address}</p>
          <p><strong>Місткість:</strong> ${capacity}</p>
          <p><strong>Відстань:</strong> ${distance > 0 ? Math.round(distance) + ' м' : 'Не визначено'}</p>
          <p><strong>Координати:</strong> ${lat.toFixed(6)}, ${lon.toFixed(6)}</p>
          <button class="route-btn" onclick="showRoute(${lat}, ${lon}, '${name}')">
            🗺️ Прокласти маршрут
          </button>
        </div>
      `;
    }

    function showEmergencyInfo(name, lat, lon, tags) {
      rightSidebar.style.display = 'block';
      
      const address = tags['addr:full'] || tags['addr:street'] || 'Адреса не вказана';
      const phone = tags.phone || 'Телефон не вказаний';
      const distance = userLocation ? calculateDistance(userLocation[0], userLocation[1], lat, lon) : 0;
      
      selectedInfo.innerHTML = `
        <div class="info-section">
          <h4>🚨 ${name}</h4>
          <p><strong>Тип:</strong> Підрозділ ДСНС</p>
          <p><strong>Адреса:</strong> ${address}</p>
          <p><strong>Телефон:</strong> ${phone}</p>
          <p><strong>Відстань:</strong> ${distance > 0 ? Math.round(distance) + ' м' : 'Не визначено'}</p>
          <p><strong>Координати:</strong> ${lat.toFixed(6)}, ${lon.toFixed(6)}</p>
          <button class="route-btn" onclick="showRoute(${lat}, ${lon}, '${name}')">
            🗺️ Прокласти маршрут
          </button>
        </div>
      `;
    }

    function showRoute(lat, lon, name) {
      if (!userLocation) {
        showMessage('Спочатку визначте ваше місце розташування', 'error');
        return;
      }
      
      const url = `https://www.google.com/maps/dir/${userLocation[0]},${userLocation[1]}/${lat},${lon}`;
      window.open(url, '_blank');
    }

    function calculateDistance(lat1, lon1, lat2, lon2) {
      const R = 6371e3; // Радіус Землі в метрах
      const φ1 = lat1 * Math.PI/180;
      const φ2 = lat2 * Math.PI/180;
      const Δφ = (lat2-lat1) * Math.PI/180;
      const Δλ = (lon2-lon1) * Math.PI/180;

      const a = Math.sin(Δφ/2) * Math.sin(Δφ/2) +
                Math.cos(φ1) * Math.cos(φ2) *
                Math.sin(Δλ/2) * Math.sin(Δλ/2);
      const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));

      return R * c;
    }

    // Закриття правого сайдбара при кліку по карті
    map.on('click', function() {
      rightSidebar.style.display = 'none';
    });
  </script>
</body>
</html>
