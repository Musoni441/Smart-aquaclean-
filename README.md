<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart AquaClean Advisor</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f0fdf4; margin: 0; padding: 15px; text-align: center; color: #333; }
        .container { max-width: 460px; margin: 0 auto; background: white; padding: 20px; border-radius: 14px; box-shadow: 0 4px 12px rgba(0,0,0,0.08); text-align: left; }
        .header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
        h1 { color: #065f46; font-size: 20px; margin: 0; }
        .lang-btn { background: #e2e8f0; border: none; padding: 6px 12px; border-radius: 6px; cursor: pointer; font-weight: bold; }
        
        .form-group { margin-bottom: 14px; }
        label { display: block; font-size: 13px; font-weight: bold; margin-bottom: 5px; color: #374151; }
        input, select { width: 100%; padding: 11px; border: 1px solid #cbd5e1; border-radius: 6px; box-sizing: border-box; font-size: 15px; background: white; }
        
        .btn-analyze { background-color: #059669; color: white; border: none; padding: 14px; font-size: 16px; font-weight: bold; border-radius: 6px; cursor: pointer; width: 100%; margin-top: 10px; }
        .btn-analyze:hover { background-color: #047857; }
        
        #advisorResult { margin-top: 20px; padding: 15px; border-radius: 8px; display: none; }
        .status-title { font-weight: bold; font-size: 18px; margin-bottom: 6px; }
        .guide-text { font-size: 14px; margin: 4px 0; line-height: 1.4; }
        
        .countdown-box { margin-top: 10px; padding: 10px; background: rgba(0,0,0,0.05); border-left: 4px solid #b91c1c; border-radius: 4px; font-weight: bold; font-size: 14px; }
        
        /* Styled WhatsApp broadcast button layout */
        .btn-whatsapp { background-color: #25d366; color: white; border: none; padding: 10px; font-size: 14px; font-weight: bold; border-radius: 6px; cursor: pointer; width: 100%; margin-top: 15px; display: flex; align-items: center; justify-content: center; gap: 8px; }
        .btn-whatsapp:hover { background-color: #128c7e; }

        .history-box { margin-top: 25px; background: #f8fafc; padding: 15px; border-radius: 10px; border: 1px solid #e2e8f0; }
        .history-title { font-weight: bold; color: #475569; font-size: 15px; border-bottom: 1px solid #cbd5e1; padding-bottom: 5px; margin-bottom: 10px; }
        .history-item { font-size: 13px; padding: 8px; border-bottom: 1px dashed #e2e8f0; color: #334155; display: flex; justify-content: space-between; }
        .clear-btn { background: #fee2e2; color: #b91c1c; border: none; padding: 4px 8px; border-radius: 4px; cursor: pointer; font-size: 11px; font-weight: bold; }
    </style>
</head>
<body>

    <div class="container">
        <div class="header">
            <h1 id="lblTitle">Smart AquaClean Dashboard</h1>
            <button class="lang-btn" onclick="switchLanguage()" id="langBtn">Kinyarwanda</button>
        </div>
        
        <div class="form-group">
            <label id="lblPond">Select Target Pond:</label>
            <select id="valPond">
                <option value="Pond A">Pond A (Tilapia)</option>
                <option value="Pond B">Pond B (Catfish)</option>
                <option value="Pond C">Pond C (Nursery)</option>
            </select>
        </div>

        <div class="form-group">
            <label id="lblVolume">Pond Water Volume (Cubic Meters - m³):</label>
            <input type="number" id="valVolume" step="1" placeholder="e.g. 150" value="100">
        </div>

        <div class="form-group">
            <label id="lblPh">Water pH Level (Ideal: 6.5 - 8.5):</label>
            <input type="number" id="valPh" step="0.1" placeholder="e.g. 7.2">
        </div>

        <div class="form-group">
            <label id="lblTemp">Water Temperature (°C):</label>
            <input type="number" id="valTemp" step="0.5" placeholder="e.g. 26">
        </div>

        <div class="form-group">
            <label id="lblAmmonia">Ammonia Level (mg/L):</label>
            <input type="number" id="valAmmonia" step="0.01" placeholder="e.g. 0.02">
        </div>

        <button class="btn-analyze" onclick="runWaterDiagnostic()" id="btnText">Analyze Water Quality</button>

        <div id="advisorResult">
            <div class="status-title" id="resStatus">-</div>
            <div class="guide-text" id="resAlert"></div>
            <div id="timeToCrisisContainer"></div> 
            <div class="guide-text" style="margin-top:10px;"><strong>Action Required:</strong> <span id="resAction">-</span></div>
            
            <button class="btn-whatsapp" onclick="sendWhatsAppBroadcast()" id="btnWa">📲 Share Emergency Alert to Cooperative</button>
        </div>

        <div class="history-box">
            <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #cbd5e1; padding-bottom: 5px; margin-bottom: 10px;">
                <span class="history-title" id="lblHist" style="border:none; margin:0; padding:0;">Pond Scan Log History</span>
                <button class="clear-btn" onclick="clearLogs()" id="lblClear">Clear All</button>
            </div>
            <div id="historyLogsList"></div>
        </div>
    </div>

    <script>
        let currentLang = "en";
        // Global variables to capture text states for the WhatsApp payload array
        let lastReportedPond = "";
        let lastReportedMetrics = "";
        let lastReportedAction = "";

        const translations = {
            en: {
                title: "Smart AquaClean Dashboard",
                lblPond: "Select Target Pond:",
                lblVolume: "Pond Water Volume (Cubic Meters - m³):",
                lblPh: "Water pH Level (Ideal: 6.5 - 8.5):",
                lblTemp: "Water Temperature (°C):",
                lblAmmonia: "Ammonia Level (mg/L - Ideal: < 0.05):",
                btnText: "Analyze Water Quality",
                lblHist: "Pond Scan Log History",
                lblClear: "Clear All",
                btnWa: "📲 Share Alert to Cooperative via WhatsApp",
                invalid: "Please fill in all water metrics correctly before analyzing.",
                
                perfectTitle: "✅ Water Quality: Excellent",
                perfectDesc: "All parameters line up perfectly with safe aquaculture metabolic baselines.",
                perfectAction: "No corrective chemical treatment needed. Maintain normal daily feed metrics.",
                
                dangerTitle: "🚨 Water Quality: CRITICAL DANGER",
                timeLabel: "⏳ Estimated Survival Window if Unchanged: "
            },
            rw: {
                title: "Isuzumiro rya AquaClean",
                lblPond: "Hitamo idamu iyapimwa:",
                lblVolume: "Ingano y'Amazi mu Kidamu (Cubic Meters - m³):",
                lblPh: "Igipimo cya pH (Ikigero cyiza: 6.5 - 8.5):",
                lblTemp: "Ubushyuhe bw'Amazi (°C):",
                lblAmmonia: "Igipimo cy'Amonyake (mg/L - Icyiza: < 0.05):",
                btnText: "Pima Ubuzima bw'Amazi",
                lblHist: "Urutonde rw'Ibyapimwe Ndoshya",
                lblClear: "Siba Byose",
                btnWa: "📲 Yohereza Ubutumwa Kuri WhatsApp ya Koperative",
                invalid: "Shyiramo ibipimo byose neza mbere yo gupima.",
                
                perfectTitle: "✅ Amazi Ni Meza Cyane",
                perfectDesc: "Ibipimo byose by'amazi bitoshye neza ntiwabuza amafi gukura neza.",
                perfectAction: "Nta muti ukenewe vuba. Komeza ukurikize amabwiriza asanzwe yo kugaburira.",
                
                dangerTitle: "🚨 Ubuzima bw'Amazi: BYAKOMEYE",
                timeLabel: "⏳ Igihe Gisgaye Amafi Ataratangira Gupfa: "
            }
        };

        window.onload = function() {
            renderHistoryLogs();
        };

        function switchLanguage() {
            currentLang = currentLang === "en" ? "rw" : "en";
            document.getElementById("langBtn").innerText = currentLang === "en" ? "Kinyarwanda" : "English";
            
            document.getElementById("lblTitle").innerText = translations[currentLang].title;
            document.getElementById("lblPond").innerText = translations[currentLang].lblPond;
            document.getElementById("lblVolume").innerText = translations[currentLang].lblVolume;
            document.getElementById("lblPh").innerText = translations[currentLang].lblPh;
            document.getElementById("lblTemp").innerText = translations[currentLang].lblTemp;
            document.getElementById("lblAmmonia").innerText = translations[currentLang].lblAmmonia;
            document.getElementById("btnText").innerText = translations[currentLang].btnText;
            document.getElementById("lblHist").innerText = translations[currentLang].lblHist;
            document.getElementById("lblClear").innerText = translations[currentLang].lblClear;
            document.getElementById("btnWa").innerText = translations[currentLang].btnWa;
        }

        function runWaterDiagnostic() {
            const pond = document.getElementById("valPond").value;
            const volume = parseFloat(document.getElementById("valVolume").value) || 0;
            const ph = parseFloat(document.getElementById("valPh").value);
            const temp = parseFloat(document.getElementById("valTemp").value);
            const ammonia = parseFloat(document.getElementById("valAmmonia").value);
            const resBox = document.getElementById("advisorResult");
            const countdownContainer = document.getElementById("timeToCrisisContainer");
            const waButton = document.getElementById("btnWa");

            if (isNaN(ph) || isNaN(temp) || isNaN(ammonia) || isNaN(volume)) {
                alert(translations[currentLang].invalid);
                return;
            }

            resBox.style.display = "block";
            countdownContainer.innerHTML = ""; 
            let status = "", description = "", action = "", isSafe = true;
            let criticalParameter = "";
            let severityDelta = 0;

            let maxAmmonia = 0.05; let minPh = 6.5; let maxPh = 8.5; let maxTemp = 32;

            if (pond === "Pond B") { 
                maxAmmonia = 0.08; minPh = 6.0; maxPh = 9.0; maxTemp = 34;
            } else if (pond === "Pond C") { 
                maxAmmonia = 0.02; minPh = 7.0; maxPh = 8.0; maxTemp = 30;
            }

            // EVALUATE WATER METRIC CROSSING RULES
            if (ammonia >= maxAmmonia) {
                isSafe = false; criticalParameter = "ammonia"; severityDelta = ammonia - maxAmmonia;
                
                // Flush Volume Calculation Logic (25% of total cubic metrics capacity)
                let flushAmount = Math.round(volume * 0.25);
                if (pond === "Pond C") {
                    action = currentLang === "en" 
                        ? `🚨 NURSERY EMERGENCY: Fragile baby fry cannot survive this level of ammonia! Stop feeding immediately. Flush out exactly ${flushAmount} m³ of pond water and refresh with clean water.` 
                        : `🚨 INYEGAMO MU KIGERAGEZO: Umwana w'ifi ntashobora kubaho muri iyi amonyake! Hagarika ibiryo ubu ngubu. Suka hanze m³ ${flushAmount} z'amazi uyasimbuze amashya meza.`;
                } else {
                    action = currentLang === "en"
                        ? `Ammonia toxicity detected! Stop feeding immediately, flush out exactly ${flushAmount} m³ of your total volume, and refill with fresh water input.`
                        : `Amonyake yabayemo uburozi! Hagarika ibiryo ubu ngubu, urasuka m³ ${flushAmount} z'amazi mu kidamu cyawe maze ushyiremo amashya.`;
                }
            } else if (ph < minPh) {
                isSafe = false; criticalParameter = "phLow"; severityDelta = minPh - ph;
                // SCIENTIFIC QUANTIFICATION: Approx 25 grams of agricultural limestone required per m³ per unit drop
                let limestoneDose = Math.round(volume * 25 * severityDelta);
                
                action = currentLang === "en"
                    ? `Water is dangerously acidic! Gradually add approximately ${limestoneDose} grams (${(limestoneDose/1000).toFixed(2)} kg) of agricultural limestone (Calcium Carbonate) across the surface to stabilize the layout.`
                    : `Amazi afite aside nyinshi cyane! Shiyiramo garama hafi ya ${limestoneDose} (${(limestoneDose/1000).toFixed(2)} kg) z'ishwagara y'ubuhinzi (limestone) mu mazi gahoro gahoro.`;
            } else if (ph > maxPh) {
                isSafe = false; criticalParameter = "phHigh"; severityDelta = ph - maxPh;
                // DOSING METRIC: Approx 15 grams of industrial agricultural gypsum needed per m³ to safely reduce pH scaling limits
                let gypsumDose = Math.round(volume * 15 * severityDelta);
                
                action = currentLang === "en"
                    ? `Water is too alkaline! Add approximately ${gypsumDose} grams (${(gypsumDose/1000).toFixed(2)} kg) of organic compost matter or gypsum to safely scale down the pH matrix.`
                    : `Amazi ntarimo aside rwose! Shyiramo garama hafi ya ${gypsumDose} (${(gypsumDose/1000).toFixed(2)} kg) z'imborera cyangwa isiha (gypsum) ngo ugabanye igipimo cya pH.`;
            } else if (temp > maxTemp) {
                isSafe = false; criticalParameter = "temp"; severityDelta = temp - maxTemp;
                action = currentLang === "en"
                    ? "Temperature is too high, decreasing oxygen capacity! Activate water aerators immediately or introduce fresh shaded water inflow."
                    : "Ubushyuhe buratanye cyane, umwuka uba muke! Vangavanga amazi ukoresheje ibyuma cyangwa ushyiremo andi makonje.";
            }

            // PREPARE AND COMPASS METADATA FOR WHATSAPP RELAY INTERFACES
            lastReportedPond = pond;
            lastReportedMetrics = `pH: ${ph} | Temp: ${temp}°C | Ammonia: ${ammonia} mg/L | Vol: ${volume} m³`;

            if (isSafe) {
                resBox.style.backgroundColor = "#dcfce7"; resBox.style.color = "#15803d";
                status = translations[currentLang].perfectTitle;
                description = currentLang === "en"
                    ? `Excellent structural configuration verified specifically for ${pond} requirements.`
                    : `Ubuzima bw'amazi ni bwiza cyane bwagenewe mu buryo bwa gihanga bwa ${pond}.`;
                action = translations[currentLang].perfectAction;
                waButton.style.display = "none"; // Hide share option if safe
            } else {
                resBox.style.backgroundColor = "#fee2e2"; resBox.style.color = "#b91c1c";
                status = translations[currentLang].dangerTitle;
                description = `Diagnostic threshold breached for ${pond}: pH=${ph}, Temp=${temp}°C, Ammonia=${ammonia}mg/L.`;
                waButton.style.display = "block"; // Show emergency broadcast control

                let hoursRemaining = 72;
                if (criticalParameter === "ammonia") {
                    if (severityDelta > 0.1) hoursRemaining = 4;
                    else if (severityDelta > 0.05) hoursRemaining = 12;
                    else hoursRemaining = 36;
                } else if (criticalParameter === "phLow" || criticalParameter === "phHigh") {
                    if (severityDelta > 1.5) hoursRemaining = 6;
                    else if (severityDelta > 0.8) hoursRemaining = 24;
                    else hoursRemaining = 48;
                } else if (criticalParameter === "temp") {
                    if (severityDelta > 4) hoursRemaining = 8;
                    else if (severityDelta > 2) hoursRemaining = 24;
                    else hoursRemaining = 48;
                }

                if (pond === "Pond C") hoursRemaining = Math.round(hoursRemaining * 0.5);

                let timeString = hoursRemaining >= 24 
                    ? `${Math.round(hoursRemaining / 24)} ${currentLang === 'en' ? 'Day(s)' : 'Iminsi'}`
                    : `${hoursRemaining} ${currentLang === 'en' ? 'Hour(s)' : 'Amasaha'}`;

                countdownContainer.innerHTML = `
                    <div class="countdown-box">
                        ${translations[currentLang].timeLabel} <span style="font-size:16px; text-decoration: underline;">${timeString}</span>
                    </div>
                `;
            }

            lastReportedAction = action;
            document.getElementById("resStatus").innerText = status;
            document.getElementById("resAlert").innerText = description;
            document.getElementById("resAction").innerText = action;

            saveLogToMemory(pond, ph, temp, ammonia, isSafe ? "HEALTHY" : "CRITICAL");
        }

        // DYNAMIC WHATSAPP LINK ENCODER FUNCTION
        function sendWhatsAppBroadcast() {
            // Package string contents cleanly using standard URI line breaks (%0A)
            let whatsappPayload = 
                `🚨 *SMART AQUACLEAN EMERGENCY ADVISORY* 🚨%0A%0A` +
                `*Target Unit:* ${lastReportedPond}%0A` +
                `*Telemetry Data:* ${lastReportedMetrics}%0A%0A` +
                `*⚠️ Critical Action Required:* ${lastReportedAction}%0A%0A` +
                `_Report generated automatically via offline mobile diagnostic app execution loop._`;

            // Open intent targeting browser system links to route to mobile app instantly
            window.open(`https://wa.me/?text=${whatsappPayload}`, '_blank');
        }

        function saveLogToMemory(pondName, p, t, a, outcome) {
            let currentLogs = JSON.parse(localStorage.getItem("aquaLogs")) || [];
            const timeStamp = new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
            
            const targetEntry = { pond: pondName, metrics: `pH:${p} | T:${t}°C | Am:${a}`, status: outcome, time: timeStamp };
            currentLogs.unshift(targetEntry);
            localStorage.setItem("aquaLogs", JSON.stringify(currentLogs));
            renderHistoryLogs();
        }

        function renderHistoryLogs() {
            const displayContainer = document.getElementById("historyLogsList");
            displayContainer.innerHTML = "";
            let currentLogs = JSON.parse(localStorage.getItem("aquaLogs")) || [];
            
            currentLogs.forEach(log => {
                const row = document.createElement("div");
                row.className = "history-item";
                const badgeColor = log.status === "HEALTHY" ? "color:#15803d;" : "color:#b91c1c; font-weight:bold;";
                
                row.innerHTML = `
                    <span><strong>[${log.time}] ${log.pond}:</strong> ${log.metrics}</span>
                    <span style="${badgeColor}">${log.status}</span>
                `;
                displayContainer.appendChild(row);
            });
        }

        function clearLogs() {
            localStorage.removeItem("aquaLogs");
            renderHistoryLogs();
        }
    </script>
</body>
</html>
