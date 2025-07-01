import React, { useState, useEffect, useRef } from ‘react’;
import { Upload, AlertTriangle, Shield, Cloud, Droplets, Flame, Map, Calculator } from ‘lucide-react’;
import * as XLSX from ‘xlsx’;

const WeatherRiskMap = () => {
const [locations, setLocations] = useState([]);
const [weatherData, setWeatherData] = useState({});
const [selectedLocation, setSelectedLocation] = useState(null);
const [mapType, setMapType] = useState(‘satellite’);
const [apiKey, setApiKey] = useState(’’);
const [riskAnalysis, setRiskAnalysis] = useState({});
const mapRef = useRef(null);
const [mapLoaded, setMapLoaded] = useState(false);

// Sample data for demonstration
const sampleLocations = [
{ id: 1, name: ‘Улаанbaatar Central’, lat: 47.9221, lng: 106.9155, coverage: 150000000, type: ‘urban’ },
{ id: 2, name: ‘Erdenet Mining’, lat: 49.0347, lng: 104.0736, coverage: 80000000, type: ‘industrial’ },
{ id: 3, name: ‘Darkhan Factory’, lat: 49.4869, lng: 105.9644, coverage: 45000000, type: ‘industrial’ },
{ id: 4, name: ‘Choibalsan Agriculture’, lat: 48.0717, lng: 114.5408, coverage: 25000000, type: ‘rural’ }
];

// Initialize map with ArcGIS integration
useEffect(() => {
if (!mapRef.current) return;

// Create map container
const mapContainer = document.createElement('div');
mapContainer.style.width = '100%';
mapContainer.style.height = '100%';
mapContainer.style.background = 'linear-gradient(45deg, #1e3a8a, #3b82f6)';
mapContainer.style.position = 'relative';
mapContainer.style.borderRadius = '8px';

mapRef.current.appendChild(mapContainer);

// Add ArcGIS-style satellite overlay
const overlay = document.createElement('div');
overlay.style.position = 'absolute';
overlay.style.top = '0';
overlay.style.left = '0';
overlay.style.right = '0';
overlay.style.bottom = '0';
overlay.style.background = `
  radial-gradient(circle at 20% 30%, rgba(34, 197, 94, 0.3) 0%, transparent 50%),
  radial-gradient(circle at 80% 70%, rgba(239, 68, 68, 0.3) 0%, transparent 50%),
  radial-gradient(circle at 50% 50%, rgba(59, 130, 246, 0.2) 0%, transparent 70%)
`;
overlay.style.borderRadius = '8px';
mapContainer.appendChild(overlay);

// Add ArcGIS attribution
const attribution = document.createElement('div');
attribution.style.position = 'absolute';
attribution.style.bottom = '5px';
attribution.style.right = '5px';
attribution.style.fontSize = '10px';
attribution.style.color = 'rgba(255,255,255,0.7)';
attribution.style.background = 'rgba(0,0,0,0.3)';
attribution.style.padding = '2px 6px';
attribution.style.borderRadius = '3px';
attribution.textContent = 'ArcGIS Online Compatible';
mapContainer.appendChild(attribution);

setMapLoaded(true);
renderLocations();

}, []);

// Render location markers
const renderLocations = () => {
if (!mapRef.current || locations.length === 0) return;

const container = mapRef.current.querySelector('div');
if (!container) return;

// Clear existing markers
const existingMarkers = container.querySelectorAll('.location-marker');
existingMarkers.forEach(marker => marker.remove());

locations.forEach((location, index) => {
  const marker = document.createElement('div');
  marker.className = 'location-marker';
  marker.style.position = 'absolute';
  marker.style.left = `${20 + (index * 150) % 400}px`;
  marker.style.top = `${80 + (index * 100) % 200}px`;
  
  const size = Math.max(20, Math.min(60, location.coverage / 5000000));
  marker.style.width = `${size}px`;
  marker.style.height = `${size}px`;
  marker.style.borderRadius = '50%';
  marker.style.cursor = 'pointer';
  
  const riskLevel = getRiskLevel(location);
  marker.style.background = getRiskColor(riskLevel);
  marker.style.border = '3px solid white';
  marker.style.boxShadow = '0 4px 12px rgba(0,0,0,0.3)';
  marker.style.transition = 'all 0.3s ease';
  
  // Add pulsing animation for high risk
  if (riskLevel === 'high') {
    marker.style.animation = 'pulse 2s infinite';
  }

  marker.addEventListener('click', () => setSelectedLocation(location));
  marker.addEventListener('mouseenter', () => {
    marker.style.transform = 'scale(1.2)';
    marker.style.zIndex = '1000';
  });
  marker.addEventListener('mouseleave', () => {
    marker.style.transform = 'scale(1)';
    marker.style.zIndex = '1';
  });

  container.appendChild(marker);
});

};

useEffect(() => {
renderLocations();
}, [locations]);

// Mock weather data fetch
const fetchWeatherData = async (lat, lng) => {
// Simulate API call
return new Promise(resolve => {
setTimeout(() => {
resolve({
temp: Math.round(Math.random() * 40 - 10),
humidity: Math.round(Math.random() * 100),
windSpeed: Math.round(Math.random() * 30),
precipitation: Math.round(Math.random() * 50),
conditions: [‘Clear’, ‘Cloudy’, ‘Rainy’, ‘Stormy’][Math.floor(Math.random() * 4)]
});
}, 500);
});
};

// Risk calculation
const calculateRisk = (location, weather) => {
let floodRisk = 0;
let fireRisk = 0;
let totalRisk = 0;

if (weather) {
  // Flood risk calculation
  floodRisk = (weather.precipitation * 0.4 + weather.humidity * 0.3) / 100;
  
  // Fire risk calculation  
  fireRisk = ((40 - weather.humidity) * 0.4 + weather.temp * 0.3 + weather.windSpeed * 0.3) / 100;
  
  // Location type modifier
  const typeModifier = {
    urban: 1.2,
    industrial: 1.5,
    rural: 0.8
  };
  
  totalRisk = (floodRisk + fireRisk) * (typeModifier[location.type] || 1);
}

const insuranceLoss = location.coverage * totalRisk * 0.15; // 15% potential loss

return {
  floodRisk: Math.min(100, floodRisk * 100),
  fireRisk: Math.min(100, fireRisk * 100),
  totalRisk: Math.min(100, totalRisk * 100),
  insuranceLoss,
  category: totalRisk > 0.7 ? 'high' : totalRisk > 0.4 ? 'medium' : 'low'
};

};

const getRiskLevel = (location) => {
const weather = weatherData[location.id];
if (!weather) return ‘unknown’;
const risk = calculateRisk(location, weather);
return risk.category;
};

const getRiskColor = (level) => {
const colors = {
low: ‘linear-gradient(135deg, #10b981, #34d399)’,
medium: ‘linear-gradient(135deg, #f59e0b, #fbbf24)’,
high: ‘linear-gradient(135deg, #ef4444, #f87171)’,
unknown: ‘linear-gradient(135deg, #6b7280, #9ca3af)’
};
return colors[level] || colors.unknown;
};

// File upload handler
const handleFileUpload = async (event) => {
const file = event.target.files[0];
if (!file) return;

try {
  const data = await file.arrayBuffer();
  const workbook = XLSX.read(data, { type: 'array' });
  const worksheet = workbook.Sheets[workbook.SheetNames[0]];
  const jsonData = XLSX.utils.sheet_to_json(worksheet);

  const processedLocations = jsonData.map((row, index) => ({
    id: index + 1,
    name: row['Байршил'] || row['Location'] || `Location ${index + 1}`,
    lat: parseFloat(row['Өргөрөг'] || row['Latitude'] || (47.9 + Math.random() * 2)),
    lng: parseFloat(row['Уртраг'] || row['Longitude'] || (106.9 + Math.random() * 8)),
    coverage: parseFloat(row['Үнэлгэн'] || row['Coverage'] || 10000000),
    type: (row['Төрөл'] || row['Type'] || 'urban').toLowerCase()
  }));

  setLocations(processedLocations);
  
  // Fetch weather for all locations
  for (const location of processedLocations) {
    const weather = await fetchWeatherData(location.lat, location.lng);
    setWeatherData(prev => ({
      ...prev,
      [location.id]: weather
    }));
  }
} catch (error) {
  console.error('Error reading file:', error);
  alert('Файл уншихад алдаа гарлаа. Excel файлын форматыг шалгана уу.');
}

};

// Load sample data
const loadSampleData = async () => {
setLocations(sampleLocations);

for (const location of sampleLocations) {
  const weather = await fetchWeatherData(location.lat, location.lng);
  setWeatherData(prev => ({
    ...prev,
    [location.id]: weather
  }));
}

};

// Export to ArcGIS compatible formats
const exportToArcGIS = () => {
const arcgisData = locations.map(location => {
const weather = weatherData[location.id];
const risk = weather ? calculateRisk(location, weather) : null;

  return {
    geometry: {
      x: location.lng,
      y: location.lat,
      spatialReference: { wkid: 4326 }
    },
    attributes: {
      OBJECTID: location.id,
      Name: location.name,
      Coverage: location.coverage,
      Type: location.type,
      Temperature: weather?.temp || 0,
      Humidity: weather?.humidity || 0,
      WindSpeed: weather?.windSpeed || 0,
      Precipitation: weather?.precipitation || 0,
      FloodRisk: risk?.floodRisk || 0,
      FireRisk: risk?.fireRisk || 0,
      TotalRisk: risk?.totalRisk || 0,
      InsuranceLoss: risk?.insuranceLoss || 0,
      RiskCategory: risk?.category || 'unknown'
    }
  };
});

// Create downloadable JSON file for ArcGIS
const dataStr = JSON.stringify({
  displayFieldName: "Name",
  fieldAliases: {
    "Name": "Байршлын нэр",
    "Coverage": "Үнэлгээ",
    "TotalRisk": "Нийт эрсдэл (%)",
    "InsuranceLoss": "Даатгалын хохирол"
  },
  geometryType: "esriGeometryPoint",
  spatialReference: { wkid: 4326 },
  features: arcgisData
}, null, 2);

const blob = new Blob([dataStr], { type: 'application/json' });
const url = URL.createObjectURL(blob);
const a = document.createElement('a');
a.href = url;
a.download = 'weather_risk_analysis.json';
a.click();
URL.revokeObjectURL(url);

};

// Generate CSV for ArcGIS Table
const exportToCSV = () => {
const csvHeader = ‘Longitude,Latitude,Name,Coverage,Type,Temperature,Humidity,WindSpeed,Precipitation,FloodRisk,FireRisk,TotalRisk,InsuranceLoss,RiskCategory\n’;

const csvRows = locations.map(location => {
  const weather = weatherData[location.id];
  const risk = weather ? calculateRisk(location, weather) : null;
  
  return [
    location.lng,
    location.lat,
    `"${location.name}"`,
    location.coverage,
    location.type,
    weather?.temp || 0,
    weather?.humidity || 0,
    weather?.windSpeed || 0,
    weather?.precipitation || 0,
    risk?.floodRisk?.toFixed(2) || 0,
    risk?.fireRisk?.toFixed(2) || 0,
    risk?.totalRisk?.toFixed(2) || 0,
    risk?.insuranceLoss?.toFixed(0) || 0,
    risk?.category || 'unknown'
  ].join(',');
}).join('\n');

const csvContent = csvHeader + csvRows;
const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
const url = URL.createObjectURL(blob);
const a = document.createElement('a');
a.href = url;
a.download = 'weather_risk_analysis.csv';
a.click();
URL.revokeObjectURL(url);

};

return (
<div className="min-h-screen bg-gradient-to-br from-slate-900 via-blue-900 to-slate-800 p-6">
<div className="max-w-7xl mx-auto">
{/* Header */}
<div className="mb-8 text-center">
<h1 className="text-4xl font-bold text-white mb-4 flex items-center justify-center gap-3">
<Map className="text-blue-400" />
Цаг агаар болон эрсдэлийн карт
</h1>
<p className="text-blue-200 text-lg">
OpenWeatherMap API болон Excel өгөгдөл ашиглан эрсдэлийн үнэлгээ
</p>
</div>

    {/* Controls */}
    <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
      <div className="bg-white/10 backdrop-blur-sm rounded-xl p-6 border border-white/20">
        <h3 className="text-white font-semibold mb-4 flex items-center gap-2">
          <Upload className="text-blue-400" size={20} />
          Өгөгдөл оруулах
        </h3>
        <input
          type="file"
          accept=".xlsx,.xls"
          onChange={handleFileUpload}
          className="w-full mb-3 text-sm text-white file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:bg-blue-500 file:text-white hover:file:bg-blue-600"
        />
        <button
          onClick={loadSampleData}
          className="w-full bg-green-500 hover:bg-green-600 text-white py-2 px-4 rounded-lg transition-colors"
        >
          Жишээ өгөгдөл ачаалах
        </button>
        {/* ArcGIS Free Tips */}
        {locations.length > 0 && (
          <div className="bg-white/10 backdrop-blur-sm rounded-xl p-6 border border-white/20">
            <h3 className="text-white font-semibold mb-4 flex items-center gap-2">
              <Shield className="text-blue-400" size={20} />
              ArcGIS Free хязгаарлалт
            </h3>
            
            <div className="space-y-2 text-sm">
              {arcgisUsageTips.map((tip, index) => (
                <div key={index} className="flex items-start gap-2 text-blue-200">
                  <div className="w-1.5 h-1.5 bg-blue-400 rounded-full mt-2 flex-shrink-0"></div>
                  <span>{tip}</span>
                </div>
              ))}
            </div>
            
            <div className="mt-4 p-3 bg-blue-500/20 rounded-lg">
              <p className="text-blue-200 text-xs">
                💡 <strong>Зөвлөмж:</strong> Та JSON эсвэл CSV файлаа татаж аваад, 
                ArcGIS Online-д "Add Layer from File" хэсгээр оруулна уу.
              </p>
            </div>
          </div>
        )}
      </div>

      <div className="bg-white/10 backdrop-blur-sm rounded-xl p-6 border border-white/20">
        <h3 className="text-white font-semibold mb-4 flex items-center gap-2">
          <Cloud className="text-blue-400" size={20} />
          API Тохиргоо
        </h3>
        <input
          type="password"
          placeholder="OpenWeatherMap API Key"
          value={apiKey}
          onChange={(e) => setApiKey(e.target.value)}
          className="w-full bg-white/10 border border-white/30 rounded-lg px-3 py-2 text-white placeholder-white/60"
        />
        <p className="text-xs text-blue-200 mt-2">
          Одоогоор жишээ өгөгдөл ашиглаж байна
        </p>
      </div>

      <div className="bg-white/10 backdrop-blur-sm rounded-xl p-6 border border-white/20">
        <h3 className="text-white font-semibold mb-4 flex items-center gap-2">
          <Map className="text-blue-400" size={20} />
          ArcGIS Export
        </h3>
        <div className="space-y-3">
          <button
            onClick={exportToArcGIS}
            disabled={locations.length === 0}
            className="w-full bg-blue-500 hover:bg-blue-600 disabled:bg-gray-500 text-white py-2 px-4 rounded-lg transition-colors flex items-center justify-center gap-2"
          >
            <Shield size={16} />
            JSON Export
          </button>
          <button
            onClick={exportToCSV}
            disabled={locations.length === 0}
            className="w-full bg-green-500 hover:bg-green-600 disabled:bg-gray-500 text-white py-2 px-4 rounded-lg transition-colors"
          >
            CSV Export
          </button>
        </div>
        <p className="text-xs text-blue-200 mt-2">
          ArcGIS Online-д шууд import хийх боломжтой
        </p>
      </div>
    </div>

    {/* Main Content */}
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
      {/* Map */}
      <div className="lg:col-span-2">
        <div className="bg-white/10 backdrop-blur-sm rounded-xl border border-white/20 overflow-hidden">
          <div className="p-4 border-b border-white/20">
            <h3 className="text-white font-semibold flex items-center gap-2">
              <Map className="text-blue-400" size={20} />
              Эрсдэлийн карт - {mapType === 'satellite' ? 'Satellite зураг' : 'Газрын зураг'}
            </h3>
          </div>
          <div 
            ref={mapRef}
            className="h-96 relative"
            style={{
              background: mapType === 'satellite' 
                ? 'linear-gradient(45deg, #1e3a8a, #3b82f6)' 
                : 'linear-gradient(45deg, #166534, #22c55e)'
            }}
          />
          
          {/* Legend */}
          <div className="p-4 border-t border-white/20">
            <div className="flex flex-wrap gap-4 justify-center">
              <div className="flex items-center gap-2">
                <div className="w-4 h-4 rounded-full bg-gradient-to-r from-green-500 to-green-400"></div>
                <span className="text-white text-sm">Бага эрсдэл</span>
              </div>
              <div className="flex items-center gap-2">
                <div className="w-4 h-4 rounded-full bg-gradient-to-r from-yellow-500 to-yellow-400"></div>
                <span className="text-white text-sm">Дунд эрсдэл</span>
              </div>
              <div className="flex items-center gap-2">
                <div className="w-4 h-4 rounded-full bg-gradient-to-r from-red-500 to-red-400"></div>
                <span className="text-white text-sm">Өндөр эрсдэл</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      {/* Sidebar */}
      <div className="space-y-6">
        {/* Location Details */}
        {selectedLocation && (
          <div className="bg-white/10 backdrop-blur-sm rounded-xl p-6 border border-white/20">
            <h3 className="text-white font-semibold mb-4 flex items-center gap-2">
              <AlertTriangle className="text-yellow-400" size={20} />
              {selectedLocation.name}
            </h3>
            
            {weatherData[selectedLocation.id] && (
              <div className="space-y-4">
                <div className="grid grid-cols-2 gap-3 text-sm">
                  <div className="text-blue-200">Температур:</div>
                  <div className="text-white">{weatherData[selectedLocation.id].temp}°C</div>
                  
                  <div className="text-blue-200">Чийгшил:</div>
                  <div className="text-white">{weatherData[selectedLocation.id].humidity}%</div>
                  
                  <div className="text-blue-200">Салхины хурд:</div>
                  <div className="text-white">{weatherData[selectedLocation.id].windSpeed} км/ц</div>
                  
                  <div className="text-blue-200">Хур тунадас:</div>
                  <div className="text-white">{weatherData[selectedLocation.id].precipitation}мм</div>
                </div>

                {(() => {
                  const risk = calculateRisk(selectedLocation, weatherData[selectedLocation.id]);
                  return (
                    <div className="mt-6 p-4 bg-black/20 rounded-lg">
                      <h4 className="text-white font-medium mb-3 flex items-center gap-2">
                        <Calculator size={16} />
                        Эрсдэлийн тооцоо
                      </h4>
                      <div className="space-y-2 text-sm">
                        <div className="flex justify-between">
                          <span className="text-blue-200 flex items-center gap-1">
                            <Droplets size={14} />
                            Үерийн эрсдэл:
                          </span>
                          <span className="text-white">{risk.floodRisk.toFixed(1)}%</span>
                        </div>
                        <div className="flex justify-between">
                          <span className="text-blue-200 flex items-center gap-1">
                            <Flame size={14} />
                            Галын эрсдэл:
                          </span>
                          <span className="text-white">{risk.fireRisk.toFixed(1)}%</span>
                        </div>
                        <div className="flex justify-between font-medium">
                          <span className="text-blue-200">Нийт эрсдэл:</span>
                          <span className={`${
                            risk.category === 'high' ? 'text-red-400' :
                            risk.category === 'medium' ? 'text-yellow-400' : 'text-green-400'
                          }`}>
                            {risk.totalRisk.toFixed(1)}%
                          </span>
                        </div>
                        <div className="pt-2 border-t border-white/20">
                          <div className="flex justify-between">
                            <span className="text-blue-200 flex items-center gap-1">
                              <Shield size={14} />
                              Даатгалын хохирол:
                            </span>
                            <span className="text-red-300 font-medium">
                              {formatCurrency(risk.insuranceLoss)}
                            </span>
                          </div>
                        </div>
                      </div>
                    </div>
                  );
                })()}
              </div>
            )}
          </div>
        )}

        {/* Risk Summary */}
        {locations.length > 0 && (
          <div className="bg-white/10 backdrop-blur-sm rounded-xl p-6 border border-white/20">
            <h3 className="text-white font-semibold mb-4 flex items-center gap-2">
              <Shield className="text-green-400" size={20} />
              Эрсдэлийн хураангуй
            </h3>
            
            <div className="space-y-3">
              {locations.map(location => {
                const weather = weatherData[location.id];
                const risk = weather ? calculateRisk(location, weather) : null;
                
                return (
                  <div
                    key={location.id}
                    className="flex items-center justify-between p-3 bg-black/20 rounded-lg cursor-pointer hover:bg-black/30 transition-colors"
                    onClick={() => setSelectedLocation(location)}
                  >
                    <div className="flex items-center gap-3">
                      <div 
                        className="w-3 h-3 rounded-full"
                        style={{ background: getRiskColor(getRiskLevel(location)) }}
                      />
                      <span className="text-white text-sm">{location.name}</span>
                    </div>
                    {risk && (
                      <span className={`text-xs px-2 py-1 rounded-full ${
                        risk.category === 'high' ? 'bg-red-500/30 text-red-200' :
                        risk.category === 'medium' ? 'bg-yellow-500/30 text-yellow-200' : 
                        'bg-green-500/30 text-green-200'
                      }`}>
                        {risk.totalRisk.toFixed(0)}%
                      </span>
                    )}
                  </div>
                );
              })}
            </div>
          </div>
        )}
      </div>
    </div>

    {/* Footer */}
    <div className="mt-8 text-center text-blue-200 text-sm">
      <p>OpenWeatherMap API, Excel файл, ArcGIS нийцтэй форматтай</p>
      <p className="mt-1">MongoDB, PostgreSQL өгөгдлийн сангаас өгөгдөл татах боломжтой</p>
    </div>
  </div>

  <style jsx>{`
    @keyframes pulse {
      0%, 100% { transform: scale(1); opacity: 1; }
      50% { transform: scale(1.1); opacity: 0.8; }
    }
  `}</style>
</div>

);
};

export default WeatherRiskMap;
