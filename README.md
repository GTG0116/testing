<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8"/>
  <title>Ephrata PA – Weather Forecast</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0; padding: 0;
      background: #1e293b;
      color: #f8fafc; /* light text for contrast */
    }
    header {
      background: #0f172a;
      color: white;
      padding: 1rem;
      text-align: center;
    }
    h1 { margin: 0; font-size: 1.5rem; }
    h2 { color: #f1f5f9; }
    .container {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 1rem;
      padding: 1rem;
    }
    .day {
      background: #334155;
      border-radius: 10px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.5);
      padding: 0.75rem;
      width: 160px;
      text-align: center;
      flex: 0 0 auto;
      color: #f8fafc;
    }
    .day img {
      width: 60px;
      height: 60px;
    }
    .day b { display: block; margin: 0.5rem 0; color: #e2e8f0; }
    .hourly {
      margin: 1rem;
      padding: 1rem;
      background: #334155;
      border-radius: 10px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.5);
      color: #f8fafc;
    }
    .hourly h2 {
      margin-top: 0;
    }
    .hour-grid {
      display: flex;
      overflow-x: auto;
      gap: 0.75rem;
    }
    .hour {
      flex: 0 0 auto;
      min-width: 100px;
      text-align: center;
      border-right: 1px solid #475569;
      padding-right: 0.5rem;
    }
    .hour:last-child { border-right: none; }
    .hour img {
      width: 40px;
      height: 40px;
    }
  </style>
</head>
<body>
  <header>
    <h1>Ephrata, PA Weather Forecast</h1>
    <small>Source: National Weather Service (State College, PA office)</small>
  </header>

  <section>
    <h2 style="text-align:center; margin-top:1rem;">5-Day Forecast</h2>
    <div class="container" id="dailyForecast">Loading forecast…</div>
  </section>

  <section class="hourly">
    <h2>Hourly Forecast (Next 24 hrs)</h2>
    <div class="hour-grid" id="hourlyForecast">Loading hourly data…</div>
  </section>

  <script>
    const lat = 40.18, lon = -76.19; // Ephrata, PA

    async function loadForecast() {
      try {
        // Step 1: Get point metadata
        const pointRes = await fetch(`https://api.weather.gov/points/${lat},${lon}`);
        const pointData = await pointRes.json();

        const dailyUrl = pointData.properties.forecast;
        const hourlyUrl = pointData.properties.forecastHourly;

        // Step 2: Fetch 5-day forecast (NWS gives 7-day w/ day+night)
        const dailyRes = await fetch(dailyUrl);
        const dailyData = await dailyRes.json();
        const periods = dailyData.properties.periods;

        // NWS gives ~14 periods (Day/Night). Take first 10 (≈5 days).
        const forecastDiv = document.getElementById("dailyForecast");
        forecastDiv.innerHTML = "";
        periods.slice(0, 10).forEach(p => {
          const div = document.createElement("div");
          div.className = "day";
          div.innerHTML = `
            <b>${p.name}</b>
            <img src="${p.icon}" alt="${p.shortForecast}">
            <span style="font-size:1.2em;">${p.temperature}°${p.temperatureUnit}</span>
            <div>${p.shortForecast}</div>
            <small>Wind: ${p.windSpeed} ${p.windDirection}</small>
          `;
          forecastDiv.appendChild(div);
        });

        // Step 3: Fetch hourly forecast
        const hourlyRes = await fetch(hourlyUrl);
        const hourlyData = await hourlyRes.json();
        const hours = hourlyData.properties.periods.slice(0, 24); // next 24 hours

        const hourlyDiv = document.getElementById("hourlyForecast");
        hourlyDiv.innerHTML = "";
        hours.forEach(h => {
          const div = document.createElement("div");
          div.className = "hour";
          div.innerHTML = `
            <div><b>${new Date(h.startTime).toLocaleTimeString([], {hour: 'numeric'})}</b></div>
            <img src="${h.icon}" alt="${h.shortForecast}">
            <div>${h.temperature}°${h.temperatureUnit}</div>
            <small>${h.shortForecast}</small>
          `;
          hourlyDiv.appendChild(div);
        });

      } catch (err) {
        console.error(err);
        document.getElementById("dailyForecast").innerHTML = "Error loading forecast.";
        document.getElementById("hourlyForecast").innerHTML = "Error loading hourly data.";
      }
    }

    loadForecast();
  </script>
</body>
</html>
