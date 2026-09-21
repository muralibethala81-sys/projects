from flask import Flask, render_template, request, jsonify
import os

app = Flask(__name__)

# Route to serve the frontend visual webpage
@app.route('/')
def home():
    return render_template('index.html')

# API Route to handle calculation requests from frontend
@app.route('/api/calculate', methods=['POST'])
def calculate():
    data = request.get_json()
    
    # Extract values safely, defaulting to 0 if empty
    transport_km = float(data.get('transportKm', 0) or 0)
    electricity_kwh = float(data.get('electricityKwh', 0) or 0)

    # Environmental carbon factors (kg of CO2 per unit)
    transport_factor = 0.21   # Average vehicle emissions per km
    electricity_factor = 0.85 # Grid power emissions per kWh

    # Perform calculations
    transport_emissions = transport_km * transport_factor
    energy_emissions = electricity_kwh * electricity_factor
    total_emissions = transport_emissions + energy_emissions

    # Social feedback logic based on emission levels
    advice = "మీరు చాలా పర్యావరణ అనుకూలంగా ఉన్నారు! గ్రీన్ లైఫ్! 🌱"
    if total_emissions > 25:
        advice = "హెచ్చరిక: మీ కార్బన్ ఉద్గారాలు ఎక్కువగా ఉన్నాయి. దయచేసి బైక్/కారు వాడకం తగ్గించి, కరెంట్ ఆదా చేయండి! ⚠️"

    # Return response formatted to 2 decimal places
    return jsonify({
        "total": f"{total_emissions:.2f}",
        "transport": f"{transport_emissions:.2f}",
        "energy": f"{energy_emissions:.2f}",
        "advice": advice
    })

if __name__ == '__main__':
    # Dynamic port configuration makes it ready for cloud deployment (e.g., Render)
    port = int(os.environ.get("PORT", 5000))
    app.run(host='0.0.0.0', port=port, debug=True)
Use code with caution.2. 🎨 templates/index.html (The Graphical UI)Inside your main folder, create a new folder named templates. Inside that folder, create a file named index.html and paste the exact same HTML code we designed earlier.The only small update is the API link inside the script tag, which is pre-configured here:html<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EcoTrack - Carbon Footprint Dashboard</title>
    <script src="https://jsdelivr.net"></script>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f4f9f4; margin: 0; padding: 20px; display: flex; justify-content: center; align-items: center; min-height: 100vh; box-sizing: border-box; }
        .card { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.08); width: 100%; max-width: 450px; text-align: center; }
        h2 { color: #2e7d32; margin-top: 0; font-size: 26px; }
        .intro-text { color: #555; font-size: 14px; margin-bottom: 25px; line-height: 1.4; }
        label { display: block; margin: 15px 0 6px; text-align: left; font-weight: 600; color: #333; font-size: 14px; }
        input { width: 100%; padding: 11px; border: 1px solid #ccc; border-radius: 6px; box-sizing: border-box; font-size: 15px; outline: none; }
        button { background-color: #2e7d32; color: white; border: none; width: 100%; padding: 13px; border-radius: 6px; font-size: 16px; font-weight: bold; margin-top: 22px; cursor: pointer; transition: background 0.2s; }
        button:hover { background-color: #1b5e20; }
        .result { margin-top: 25px; padding: 18px; background: #e8f5e9; border-radius: 8px; display: none; text-align: left; border-left: 5px solid #2e7d32; }
        .result p { margin: 8px 0; font-size: 15px; color: #333; }
        .chart-container { width: 100%; max-width: 280px; margin: 25px auto 0 auto; display: none; }
    </style>
</head>
<body>

<div class="card">
    <h2>🌱 EcoTrack కాలిక్యులేటర్</h2>
    <p class="intro-text">మీ రోజువారీ పనుల వల్ల ఎంత కాలుష్యం (CO2) పుడుతుందో లెక్కించి నిమిషాల్లో విజువలైజ్ చేసుకోండి.</p>
    
    <label>ఈరోజు బైక్/కారు ఎంత దూరం నడిపారు? (కిలోమీటర్లలో)</label>
    <input type="number" id="km" placeholder="ఉదాహరణకు: 15" min="0">

    <label>ఈరోజు మీ ఇంట్లో వాడిన కరెంట్? (యూనిట్లలో/kWh)</label>
    <input type="number" id="kwh" placeholder="ఉదాహరణకు: 5" min="0">

    <button onclick="calculateFootprint()">లెక్కించు (Calculate)</button>

    <div class="result" id="resultBox">
        <strong style="color: #2e7d32; font-size: 17px; display: block; margin-bottom: 10px;">📊 విశ్లేషణ ఫలితాలు:</strong>
        <p>🚗 వాహన కాలుష్యం: <strong><span id="transportCo2">0</span> kg</strong> CO₂</p>
        <p>💡 విద్యుత్ కాలుష్యం: <strong><span id="energyCo2">0</span> kg</strong> CO₂</p>
        <hr style="border: 0; border-top: 1px solid #c8e6c9; margin: 12px 0;">
        <p style="font-size: 16px; font-weight: bold; color: #1b5e20;">🌳 మొత్తం ఉద్గారం: <span id="totalCo2">0</span> kg</p>
        <p style="margin-top: 10px; line-height: 1.4;"><strong>💡 సలహా:</strong> <span id="adviceText"></span></p>
    </div>

    <div class="chart-container" id="chartContainer">
        <canvas id="emissionChart"></canvas>
    </div>
</div>
