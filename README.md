# Weather-Forecast
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Stunning Weather App</title>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap');

  * { box-sizing: border-box; }

  body {
    margin: 0;
    font-family: 'Poppins', sans-serif;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 30px 15px 60px;
    color: #00feba;
    background: linear-gradient(to bottom, #0f2027, #203a43, #2c5364);
    background-size: cover;
    background-position: center;
    transition: background-image 1s ease-in-out;
  }

  body::before {
    content: "";
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.45);
    z-index: -1;
  }

  header { margin-bottom: 25px; text-align: center; }

  header h1 {
    font-size: 2.8rem;
    color: #00feba;
    text-shadow: 0 0 12px #00febaaa;
  }

  form.search {
    display: flex;
    max-width: 470px;
    width: 100%;
    margin-bottom: 20px;
  }

  form.search input {
    flex: 1;
    padding: 14px 20px;
    border: none;
    border-radius: 50px 0 0 50px;
    font-size: 18px;
    background: rgba(255 255 255 / 0.15);
    color: #00feba;
  }

  form.search button {
    background: #00feba;
    border: none;
    border-radius: 0 50px 50px 0;
    width: 60px;
    cursor: pointer;
  }

  #cardsContainer {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 25px;
    width: 100%;
    max-width: 1100px;
  }

  .weather-card {
    background: rgba(255 255 255 / 0.1);
    backdrop-filter: blur(25px);
    border-radius: 25px;
    padding: 25px 30px;
    width: 320px;
    text-align: center;
    box-shadow: 0 0 25px #00feba88;
    transition: 0.3s;
    color: #00feba;
  }

  .card-header { font-size: 1.8rem; }

  .weather-icon { width: 120px; }

  .temp { font-size: 48px; }

  .city-name { font-size: 24px; color: white; }

  .details {
    display: flex;
    justify-content: space-around;
  }

  .error-msg { color: red; }

  .forecast { margin-top: 15px; }
  .forecast-container {
    display: flex;
    gap: 10px;
    overflow-x: auto;
  }
  .forecast-card {
    min-width: 90px;
    background: rgba(255 255 255 / 0.1);
    border-radius: 12px;
    padding: 10px;
    font-size: 13px;
    color: #9ef7f1;
  }
</style>
</head>

<body>

<header>
  <h1>Stunning Weather App</h1>
</header>

<form class="search" onsubmit="return false;">
  <input type="text" placeholder="Enter city name"/>
  <button>🔍</button>
</form>

<div id="cardsContainer"></div>
<div class="error-msg" id="errorMsg"></div>

<script>
const apiKey = "03707dfc0b181dca5661b3662b8798e6";

const iconMap = {
  Clear: "https://img.icons8.com/ios-filled/100/00feba/sun--v1.png",
  Clouds: " https://img.icons8.com/ios-filled/100/00feba/cloud.png ",
  Rain: " https://img.icons8.com/ios-filled/100/00feba/rain.png ",
  Drizzle: " https://img.icons8.com/ios-filled/100/00feba/light-rain.png ",
  Mist: " https://img.icons8.com/ios-filled/100/00feba/fog-day.png ",
  Snow: "  https://img.icons8.com/ios-filled/100/00feba/snow.png ",
  Thunderstorm: "  https://img.icons8.com/ios-filled/100/00feba/storm.png "
};

const moodMap = {
  Clear: "☀️ It's a bright and sunny day! Perfect for outdoor fun.",
  Clouds: "☁️ Relax and enjoy the calm weather.",
  Rain: "🌧️ Don't forget your umbrella, rainy vibes today!",
  Drizzle: "🌦️ Light rain outside, take a jacket just in case.",
  Mist: "🌫️ Misty weather, drive safe and stay cozy.",
  Snow: "❄️ Snowfall alert! Time for some winter fun.",
  Thunderstorm: "⛈️ Thunderstorms nearby, better stay indoors."
};

const cardsContainer = document.getElementById("cardsContainer");
const errorMsg = document.getElementById("errorMsg");
const form = document.querySelector("form.search");
const input = form.querySelector("input");

const bgImages = {
  Clear: "url('  https://images.unsplash.com/photo-1506748686214-e9df14d4d9d0?auto=format&fit=crop&w=1470&q=80' )",
  Clouds: "url(' https://images.unsplash.com/photo-1499346030926-9a72daac6c63?auto=format&fit=crop&w=1470&q=80' )",
  Rain: "url(' https://images.unsplash.com/photo-1527766833261-b09c3163a791?auto=format&fit=crop&w=1470&q=80 ')",
  Drizzle: "url(' https://images.unsplash.com/photo-1506744038136-46273834b3fb?auto=format&fit=crop&w=1470&q=80 ')",
  Mist: "url(' https://images.unsplash.com/photo-1502082553048-f009c37129b9?auto=format&fit=crop&w=1470&q=80 ')",
  Snow: "url(' https://images.unsplash.com/photo-1601327350982-1826f9a6b8b7?auto=format&fit=crop&w=1470&q=80 ')",
  Thunderstorm: "url(' https://images.unsplash.com/photo-1504384308090-c894fdcc538d?auto=format&fit=crop&w=1470&q=80' )"
};

function createWeatherCard(data, title = "Current Location") {
  const weatherType = data.weather[0].main;
  const mood = moodMap[weatherType] || "Enjoy the weather!";
  document.body.style.backgroundImage = bgImages[weatherType] || bgImages.Clear;

  const card = document.createElement("article");
  card.className = "weather-card";

  if(title === "Current Location"){
    card.setAttribute("data-current-location", "true");
  }

  card.innerHTML = `
    <div class="card-header">${title}</div>
    <img class="weather-icon" src="${iconMap[weatherType]}" />
    <div class="temp">${Math.round(data.main.temp)}°C</div>
    <div class="weather-mood">${mood}</div>
    <div class="city-name">${data.name}</div>
    <div class="details">
      <div>${data.main.humidity}%</div>
      <div>${(data.wind.speed*3.6).toFixed(1)} km/h</div>
    </div>
  `;
  return card;
}

async function fetchForecast(city, card) {
  const resp = await fetch(
    `https://api.openweathermap.org/data/2.5/forecast?q=${city}&units=metric&appid=${apiKey}`
  );
  const data = await resp.json();

  let html = `<div class="forecast"><div class="forecast-container">`;

  for (let i = 0; i < data.list.length; i += 8) {
    const d = data.list[i];
    html += `
      <div class="forecast-card">
        <div>${new Date(d.dt_txt).toDateString()}</div>
        <div>${Math.round(d.main.temp)}°C</div>
        <div>${d.weather[0].main}</div>
      </div>
    `;
  }

  html += `</div></div>`;
  card.insertAdjacentHTML("beforeend", html);
}

async function fetchWeatherCity(city, title = "Searched City") {
  try {
    const resp = await fetch(
      `https://api.openweathermap.org/data/2.5/weather?q=${city}&units=metric&appid=${apiKey}`
    );
    const data = await resp.json();

    const card = createWeatherCard(data, title);
    fetchForecast(data.name, card);

    cardsContainer.appendChild(card);

    // ✅ ONLY CHANGE
    const searched = [...cardsContainer.querySelectorAll(".weather-card:not([data-current-location])")];
    if (searched.length > 2) {
      searched[0].remove();
    }

  } catch {
    errorMsg.textContent = "City not found";
  }
}

async function fetchWeatherCoords(lat, lon) {
  const resp = await fetch(
    ` https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&units=metric&appid=${apiKey} `
  );
  const data = await resp.json();

  const card = createWeatherCard(data, "Current Location");
  fetchForecast(data.name, card);

  cardsContainer.prepend(card);
}

async function fetchLocationByIP() {
  const resp = await fetch("https://ipapi.co/json/");
  const data = await resp.json();
  if (data.city) fetchWeatherCity(data.city, "Current Location");
}

window.addEventListener("load", () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      pos => fetchWeatherCoords(pos.coords.latitude, pos.coords.longitude),
      () => fetchLocationByIP()
    );
  } else {
    fetchLocationByIP();
  }
});

form.addEventListener("submit", () => {
  const city = input.value.trim();
  if (!city) return;
  fetchWeatherCity(city);
  input.value = "";
});
</script>

</body>
</html>



