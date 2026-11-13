<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart Farm Advisor & Independent Irrigation Monitor</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f4f7f6; margin: 0; padding: 20px; color: #333; }
        .container { max-width: 700px; background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); margin: auto; }
        h2 { color: #2c3e50; text-align: center; margin-bottom: 5px; }
        p.subtitle { text-align: center; color: #7f8c8d; margin-top: 0; font-size: 14px; }
        .disclaimer-banner { background: #fdfefe; border: 1px dashed #bdc3c7; padding: 8px; font-size: 11px; color: #7f8c8d; text-align: center; margin-bottom: 15px; border-radius: 6px; }
        label { font-weight: bold; display: block; margin-top: 12px; font-size: 14px; color: #34495e; }
        input, select { width: 100%; padding: 10px; margin-top: 5px; border: 1px solid #ccc; border-radius: 6px; box-sizing: border-box; font-size: 14px; }
        .btn-group { display: flex; gap: 10px; margin-top: 15px; }
        button { background: #27ae60; color: white; border: none; padding: 12px; border-radius: 6px; font-size: 16px; cursor: pointer; flex: 1; font-weight: bold; transition: background 0.3s; }
        button:hover { background: #219653; }
        button.secondary { background: #2980b9; }
        button.secondary:hover { background: #1f618d; }
        .card { background: #f8f9fa; border: 1px solid #e9ecef; padding: 20px; margin-top: 25px; border-radius: 8px; display: none; }
        .result-box { border-left: 5px solid #27ae60; background: #e8f8f5; }
        .warning-box { border-left: 5px solid #f39c12; background: #fef9e7; }
        .danger-box { border-left: 5px solid #e74c3c; background: #fadbd8; }
        .gps-presets { display: flex; gap: 5px; margin-top: 5px; font-size: 12px; }
        .gps-presets a { color: #2980b9; cursor: pointer; text-decoration: underline; }
    </style>
</head>
<body>

<div class="container">
    <h2>🌾 Smart Farm Advisor</h2>
    <p class="subtitle">Independent Agricultural Irrigation & Moisture Tracker</p>
    
    <div class="disclaimer-banner">
        ⚠️ <strong>Notice:</strong> This is an independent, unaffiliated analytical tool for educational and demonstration purposes. It does not officially represent external organizations or transmit data to government ministries.
    </div>
    
    <form id="farmForm" onsubmit="processForm(event)">
        <label>1. Farmer's Name:</label>
        <input type="text" id="farmer_name" required placeholder="e.g. Tarek">

        <label>2. Phone / WhatsApp Number:</label>
        <input type="text" id="farmer_phone" placeholder="e.g. +201000000000">

        <label>3. Region / Governorate:</label>
        <input type="text" id="region" placeholder="e.g. Qalyubia, Banha">

        <label>4. Farm GPS Coordinates (Google Maps Location):</label>
        <input type="text" id="gps_coords" required placeholder="e.g. 30.0444, 31.2357">
        <div class="gps-presets">
            Quick fill: 
            <a onclick="setGPS('30.0444, 31.2357')">Cairo</a> | 
            <a onclick="setGPS('30.4660, 31.1848')">Banha</a> | 
            <a onclick="setGPS('31.2001, 29.9187')">Alexandria</a>
        </div>

        <label>5. Crop Type:</label>
        <input type="text" id="target_crop" required placeholder="e.g. Tomatoes, Wheat, Corn">

        <label>6. Weather Condition:</label>
        <select id="weather">
            <option value="hot">Hot (High Evaporation)</option>
            <option value="mild">Mild / Moderate</option>
            <option value="rainy">Rainy</option>
        </select>

        <label>7. Soil Type:</label>
        <select id="soil_type">
            <option value="sandy">Sandy (Fast Drainage)</option>
            <option value="clay">Clay (Retains Water)</option>
            <option value="mixed">Mixed (Balanced)</option>
        </select>

        <div class="btn-group">
            <button type="submit">Generate Seasonal Plan</button>
        </div>
    </form>

    <!-- Seasonal Plan Card -->
    <div id="planCard" class="card result-box">
        <h3>📋 Initial Seasonal Advisory Plan</h3>
        <p id="planDetails"></p>
    </div>

    <!-- Daily Doctor Check-in Section -->
    <div id="checkinSection" class="card" style="display:none; background: #eef6fc; border-left: 5px solid #3498db;">
        <h3>🩺 Daily Farm Status Check-In (Day <span id="dayNum">1</span>)</h3>
        <p id="doctorQuestion"></p>
        <div class="btn-group">
            <button type="button" class="secondary" onclick="dailyCheck(true)">✅ Yes, Irrigated on Time</button>
            <button type="button" style="background: #e74c3c;" onclick="dailyCheck(false)">❌ No, Missed Irrigation</button>
        </div>
    </div>

    <!-- Alert / Status Card -->
    <div id="alertCard" class="card">
        <h3 id="alertTitle">Status Report</h3>
        <p id="alertMessage"></p>
        <p id="mapLinkContainer"></p>
    </div>
</div>

<script>
    let farmData = {};
    let missedDays = 0;
    let dayCounter = 1;

    function setGPS(coords) {
        document.getElementById('gps_coords').value = coords;
    }

    function processForm(event) {
        event.preventDefault();
        
        farmData.name = document.getElementById('farmer_name').value;
        farmData.phone = document.getElementById('farmer_phone').value;
        farmData.region = document.getElementById('region').value;
        farmData.gps = document.getElementById('gps_coords').value;
        farmData.crop = document.getElementById('target_crop').value;
        farmData.weather = document.getElementById('weather').value;
        farmData.soil = document.getElementById('soil_type').value;

        let wateringTime = (farmData.weather === 'hot' || farmData.soil === 'sandy') ? 
            "Early Morning (5:30 AM - 7:30 AM) or Sunset" : "Early Morning (6:00 AM - 8:00 AM)";
        let freq = (farmData.soil === 'sandy') ? "Every 1 to 2 days" : (farmData.soil === 'clay' ? "Every 4 to 5 days" : "Every 2 to 3 days");

        let planText = `<strong>Farmer:</strong> ${farmData.name} (${farmData.phone})<br>` +
                       `<strong>Crop:</strong> ${farmData.crop.toUpperCase()} | <strong>Region:</strong> ${farmData.region}<br>` +
                       `💧 <strong>Best Watering Time:</strong> ${wateringTime}<br>` +
                       `⏱️ <strong>Frequency:</strong> ${freq}<br>` +
                       `📍 <strong>GPS Location:</strong> <a href="https://www.google.com/maps?q=${farmData.gps.replace(/\s/g, '')}" target="_blank">View on Google Maps</a>`;

        document.getElementById('planDetails').innerHTML = planText;
        document.getElementById('planCard').style.display = 'block';

        missedDays = 0;
        dayCounter = 1;
        updateCheckinPrompt();
        document.getElementById('checkinSection').style.display = 'block';
        document.getElementById('alertCard').style.display = 'none';
    }

    function updateCheckinPrompt() {
        document.getElementById('dayNum').innerText = dayCounter;
        document.getElementById('doctorQuestion').innerText = `System check for ${farmData.name}: Did you water your ${farmData.crop} today?`;
    }

    function dailyCheck(didWater) {
        let alertCard = document.getElementById('alertCard');
        let alertTitle = document.getElementById('alertTitle');
        let alertMessage = document.getElementById('alertMessage');
        let mapLinkContainer = document.getElementById('mapLinkContainer');
        
        alertCard.style.display = 'block';
        mapLinkContainer.innerHTML = '';

        if (didWater) {
            missedDays = 0;
            alertCard.className = 'card result-box';
            alertTitle.innerText = "✅ Excellent Work!";
            alertMessage.innerHTML = `Great job ${farmData.name}! Your ${farmData.crop} received water on time today. Soil moisture is optimal.`;
        } else {
            missedDays++;
            if (missedDays === 1) {
                alertCard.className = 'card warning-box';
                alertTitle.innerText = "⚠️ First Warning!";
                alertMessage.innerHTML = `Hello ${farmData.name}, you missed today's irrigation for your ${farmData.crop}! Risk of plant stress. Please irrigate tomorrow morning. (Missed days: 1)`;
            } else {
                let cleanGPS = farmData.gps.replace(/\s/g, '');
                let mapsUrl = `https://www.google.com/maps?q=${cleanGPS}`;
                
                alertCard.className = 'card danger-box';
                alertTitle.innerText = "🚨 CRITICAL DEHYDRATION ALERT!";
                alertMessage.innerHTML = `<strong>DANGER!</strong> ${missedDays} consecutive days without water! Simulated emergency status triggered for local agricultural tracking.<br><br>` +
                                         `<strong>📊 LOGGED MONITORING DATA:</strong><br>` +
                                         `- Farmer: ${farmData.name} (${farmData.phone})<br>` +
                                         `- Region: ${farmData.region}<br>` +
                                         `- Crop: ${farmData.crop.toUpperCase()} (${farmData.soil} soil)<br>` +
                                         `- Status: Critical dehydration risk. Immediate local intervention required.`;
                mapLinkContainer.innerHTML = `🔗 <strong>Saved GPS Link:</strong> <a href="${mapsUrl}" target="_blank">${mapsUrl}</a>`;
            }
        }
        dayCounter++;
        updateCheckinPrompt();
    }
</script>

</body>
</html>