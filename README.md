# weather-risk-map
import React, { useState, useEffect, useRef } from 'react';
import { Upload, Shield, Cloud, Map as MapIcon, AlertTriangle, Calculator, Droplets, Flame } from 'lucide-react';
import * as XLSX from 'xlsx';

const WeatherRiskMap = () => {
  const [locations, setLocations] = useState([]);
  const [weatherData, setWeatherData] = useState({});
  const [selectedLocation, setSelectedLocation] = useState(null);
  const [apiKey, setApiKey] = useState('');
  const mapRef = useRef(null);

  // Sample locations
  const sampleLocations = [
    { id: 1, name: 'Ulaanbaatar Central', lat: 47.9221, lng: 106.9155, coverage: 150000000, type: 'urban' },
    { id: 2, name: 'Erdenet Mining', lat: 49.0347, lng: 104.0736, coverage: 80000000, type: 'industrial' },
    { id: 3, name: 'Darkhan Factory', lat: 49.4869, lng: 105.9644, coverage: 45000000, type: 'industrial' },
    { id: 4, name: 'Choibalsan Agriculture', lat: 48.0717, lng: 114.5408, coverage: 25000000, type: 'rural' }
  ];

  // Fetch weather from OpenWeatherMap
  const fetchWeatherData = async (lat, lon) => {
    if (!apiKey) return null;
    const res = await fetch(https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${apiKey}&units=metric);
    return await res.json();
  };

  // Risk calculation
  const calculateRisk = (loc, weather) => {
    if (!weather) return {};
    const rain = weather.rain?.['1h'] || 0;
    const humidity = weather.main.humidity;
    const temp = weather.main.temp;
    const wind = weather.wind.speed;

    const flood = (rain * 0.4 + humidity * 0.3) / 100;
    const fire = ((40 - humidity) * 0.4 + temp * 0.3 + wind * 0.3) / 100;
    const typeMod = { urban:1.2, industrial:1.5, rural:0.8 };
    const total = (flood + fire) * (typeMod[loc.type] || 1);
    const insuranceLoss = loc.coverage * total * 0.15;

    const cat = total > 0.7 ? 'high' : total > 0.4 ? 'medium' : 'low';
    return {
      floodRisk: Math.min(100, flood*100),
      fireRisk: Math.min(100, fire*100),
      totalRisk: Math.min(100, total*100),
      insuranceLoss,
      category: cat
    };
  };

  // Load locations + weather
  const loadData = async (list) => {
    setLocations(list);
    const wd = {};
    for (const loc of list) {
      const w = await fetchWeatherData(loc.lat, loc.lng);
      wd[loc.id] = w ? calculateRisk(loc, w) : null;
    }
    setWeatherData(wd);
  };

  // Sample loader
  const loadSample = () => loadData(sampleLocations);

  // File upload handler
  const handleFileUpload = async e => {
    const f = e.target.files[0];
    const data = await f.arrayBuffer();
    const wb = XLSX.read(data, {type:'array'});
    const json = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]]);
    const locs = json.map((r,i)=>({
      id:i+1,
      name: r['Location'] || r['Байршил'] || Loc ${i+1},
      lat: parseFloat(r['Latitude'] || r['Өргөрөг']) || 0,
      lng: parseFloat(r['Longitude'] || r['Уртраг']) || 0,
      coverage: +r['Coverage'] || +r['Үнэлгээ'] || 0,
      type: (r['Type'] || r['Төрөл'] || 'urban').toLowerCase()
    }));
    loadData(locs);
  };

  // Render map & markers
  useEffect(() => {
    if (!mapRef.current) return;
    const div = mapRef.current;
    div.innerHTML = '';
    const w = div.offsetWidth, h = div.offsetHeight;
    locations.forEach(loc => {
      const mk = document.createElement('div');
      mk.style.position='absolute';
      mk.style.left = ${Math.random()*(w-60)}px;
      mk.style.top = ${Math.random()*(h-60)}px;
      const r = weatherData[loc.id]?.totalRisk || 0;
      const size = Math.max(20, Math.min(60, loc.coverage/5000000));
      const colors = { low:'#10b981', medium:'#f59e0b', high:'#ef4444' };
      mk.style.width = mk.style.height = ${size}px;
      mk.style.background = colors[weatherData[loc.id]?.category] || '#9ca3af';
      mk.style.border='3px solid white'; mk.style.borderRadius='50%';
      mk.title = loc.name;
      div.appendChild(mk);
    });
  }, [locations, weatherData]);

  return (
    <div className="p-4">
      <h2>Weather Risk Map</h2>
      <div>
        <input type="file" accept=".xlsx,.xls" onChange={handleFileUpload} />
        <button onClick={loadSample}>Load Sample</button>
        <input type="password" value={apiKey} onChange={e=>setApiKey(e.target.value)} placeholder="OpenWeather API key" />
      </div>
      <div ref={mapRef} style={{position:'relative', width:'100%', height:'400px', background:'#333', marginTop:'20px'}} />
      <div>
        {locations.map(loc=>(
          <div key={loc.id}>
            {loc.name}: {weatherData[loc.id]?.totalRisk?.toFixed(1) || 'N/A'}%
          </div>
        ))}
      </div>
    </div>
  );
};

export default WeatherRiskMap;
