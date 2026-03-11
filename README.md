[GeoV9.txt](https://github.com/user-attachments/files/25902840/GeoV9.txt)
<!DOCTYPE html>
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <!-- 破解圖片防盜鏈 -->
    <meta name="referrer" content="no-referrer">
    <title>香港地理達人 (全平台支援)</title>
    
    <!-- 穩定 CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.css" />
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.js"></script>
    <!-- Panzoom 縮放引擎 -->
    <script src="https://cdn.jsdelivr.net/npm/@panzoom/panzoom@4.5.1/dist/panzoom.min.js"></script>

    <style>
        /* 修復手機高度 Bug，全面使用 fixed 和 inset: 0 */
        body, html { margin: 0; padding: 0; width: 100%; height: 100%; background-color: #111827; color: white; font-family: 'PingFang HK', 'Microsoft JhengHei', sans-serif; overflow: hidden; position: fixed; }
        
        .fullscreen-layer { position: fixed; top: 0; left: 0; right: 0; bottom: 0; }
        
        /* 圖片區塊 */
        #photo-wrapper { background-color: #1a202c; display: flex; justify-content: center; align-items: center; z-index: 1; touch-action: none; cursor: grab; }
        #photo-wrapper:active { cursor: grabbing; }
        #location-photo { width: 100%; height: 100%; object-fit: contain; display: none; }

        /* 載入畫面 */
        #loading-overlay { background: rgba(17, 24, 39, 0.95); z-index: 20; display: flex; flex-direction: column; justify-content: center; align-items: center; }

        /* UI 層 (頂部資訊) */
        #ui-layer { z-index: 10; padding: 15px; pointer-events: none; display: flex; flex-direction: column; justify-content: flex-start; }
        .top-bar { display: flex; justify-content: space-between; pointer-events: auto; }
        .info-box { background: rgba(0, 0, 0, 0.7); padding: 8px 16px; border-radius: 8px; font-size: 1.1rem; font-weight: bold; border: 1px solid rgba(255,255,255,0.2); backdrop-filter: blur(4px); }
        .timer-warning { color: #fc8181; animation: pulse 1s infinite; }

        /* =========================================
           全新設計：固定於右下角的懸浮地圖按鈕 (FAB)
           ========================================= */
        #fab-map-btn {
            position: fixed; bottom: 25px; right: 25px; z-index: 30; pointer-events: auto;
            background-color: #3b82f6; color: white; width: 70px; height: 70px; border-radius: 50%;
            display: flex; justify-content: center; align-items: center; font-size: 1.8rem;
            box-shadow: 0 10px 25px rgba(0,0,0,0.6); border: 3px solid rgba(255,255,255,0.3);
            cursor: pointer; transition: transform 0.2s, background-color 0.2s;
        }
        #fab-map-btn:hover { transform: scale(1.05); background-color: #2563eb; }
        #fab-map-btn::after {
            content: "估位置"; position: absolute; bottom: -22px; font-size: 0.8rem; 
            font-weight: bold; background: rgba(0,0,0,0.6); padding: 2px 6px; border-radius: 4px; white-space: nowrap;
        }

        /* 彈出式地圖視窗 (Modal) */
        #map-modal {
            background: rgba(0, 0, 0, 0.9); z-index: 40; display: none;
            justify-content: center; align-items: center; padding: 15px; pointer-events: auto;
        }
        #map-container { 
            width: 100%; max-width: 800px; height: 80%; background: white; border-radius: 16px; 
            overflow: hidden; display: flex; flex-direction: column; position: relative;
            box-shadow: 0 15px 40px rgba(0,0,0,0.8); border: 4px solid #4b5563;
        }
        
        .close-btn {
            position: absolute; top: 15px; right: 15px; z-index: 1000;
            background: #ef4444; color: white; border: none; border-radius: 50%; width: 45px; height: 45px;
            font-size: 1.5rem; cursor: pointer; box-shadow: 0 4px 10px rgba(0,0,0,0.4); display: flex; justify-content: center; align-items: center;
        }
        
        #guess-map { flex-grow: 1; width: 100%; background-color: #e5e5e5; cursor: crosshair; }
        
        .map-actions { display: flex; padding: 10px; background: #f3f4f6; gap: 10px; }
        .map-actions button { flex: 1; padding: 15px; font-size: 1.2rem; font-weight: bold; border-radius: 8px; transition: 0.2s; color: white; }
        #guess-btn { background-color: #10b981; }
        #guess-btn:disabled { background-color: #9ca3af; cursor: not-allowed; }

        /* 其他覆蓋層 */
        #result-overlay { background: rgba(17, 24, 39, 0.98); z-index: 50; display: flex; flex-direction: column; justify-content: center; align-items: center; pointer-events: auto; }
        #result-map { width: 90%; max-width: 800px; height: 40%; border-radius: 12px; margin-bottom: 20px; border: 2px solid #374151; background-color: #e5e5e5; }
        #end-screen, #start-screen { background: rgba(17, 24, 39, 0.98); z-index: 60; display: flex; flex-direction: column; justify-content: center; align-items: center; overflow-y: auto; }

        @media (max-width: 768px) {
            .info-box { font-size: 0.95rem; padding: 6px 10px; }
            #fab-map-btn { width: 60px; height: 60px; font-size: 1.5rem; bottom: 30px; right: 20px; }
        }

        @keyframes pulse { 0% { transform: scale(1); } 50% { transform: scale(1.05); } 100% { transform: scale(1); } }
    </style>
</head>
<body>

    <!-- 1. 初始畫面 -->
    <div id="start-screen" class="fullscreen-layer">
        <div class="bg-gray-800 p-8 md:p-10 rounded-2xl shadow-2xl text-center border border-gray-700 max-w-lg w-full m-4">
            <h1 class="text-4xl font-bold mb-4 text-emerald-400"><i class="fas fa-search-location mr-3"></i>香港地理達人</h1>
            <div class="inline-block bg-emerald-900 text-emerald-300 px-3 py-1 rounded-full text-sm font-semibold mb-6 border border-emerald-700">全平台完美支援版</div>
            <p class="text-gray-300 mb-6 text-left leading-relaxed text-sm md:text-base">
                <i class="fas fa-check-circle mr-2 text-green-400"></i><b>手機/平板完美修復：</b> 地圖不會再消失，相片比例正常！<br>
                <i class="fas fa-map-pin mr-2 text-blue-400"></i><b>懸浮地圖按鈕：</b> 點擊右下角藍色圓形按鈕打開地圖。<br>
                <i class="fas fa-robot mr-2 text-yellow-400"></i><b>自動過濾與防重：</b> 實時連接維基百科抽取香港 18 區實景，絕不重複。
            </p>
            <button onclick="startGame()" class="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-4 px-6 rounded-lg text-xl transition duration-200 shadow-lg">
                開始挑戰 <i class="fas fa-play ml-2"></i>
            </button>
        </div>
    </div>

    <!-- 2. 遊戲主畫面 -->
    <div id="game-container" class="fullscreen-layer" style="display: none;">
        
        <div id="photo-wrapper" class="fullscreen-layer">
            <img id="location-photo" src="" alt="香港風景">
        </div>
        
        <div id="loading-overlay" class="fullscreen-layer" style="display: none;">
            <i class="fas fa-compass fa-spin text-6xl text-blue-400 mb-4"></i>
            <h2 class="text-2xl font-bold text-white mb-2" id="loading-text">正在尋找香港秘境...</h2>
            <p class="text-gray-400 text-sm">請耐心等候高清實景載入</p>
        </div>

        <div id="ui-layer" class="fullscreen-layer">
            <div class="top-bar">
                <div class="info-box"><i class="fas fa-flag-checkered mr-1"></i> <span id="round-display">1 / 5</span></div>
                <div class="info-box" id="timer-box"><i class="fas fa-clock mr-1 text-blue-400"></i> <span id="time-display">--:--</span></div>
                <div class="info-box text-emerald-400"><i class="fas fa-star mr-1"></i> <span id="score-display">0</span></div>
            </div>
            
            <div class="w-full flex justify-center mt-4 pointer-events-none">
                <div class="bg-gray-800 bg-opacity-80 px-4 py-2 rounded-full text-sm font-bold text-gray-200 shadow-lg">
                    <i class="fas fa-hand-pointer mr-1"></i> 雙指放大圖片尋找細節
                </div>
            </div>
        </div>
        
        <!-- 全新：固定懸浮地圖按鈕 -->
        <div id="fab-map-btn" onclick="openMap()">
            <i class="fas fa-map-marked-alt"></i>
        </div>
        
        <!-- 彈出式地圖視窗 -->
        <div id="map-modal" class="fullscreen-layer">
            <div id="map-container">
                <button class="close-btn" onclick="closeMap()"><i class="fas fa-times"></i></button>
                <div id="guess-map"></div>
                <div class="map-actions">
                    <button onclick="closeMap()" class="bg-gray-500 hover:bg-gray-600">收起地圖</button>
                    <button id="guess-btn" onclick="submitGuess()" disabled>確定位置</button>
                </div>
            </div>
        </div>
    </div>

    <!-- 3. 回合結算畫面 -->
    <div id="result-overlay" class="fullscreen-layer" style="display: none;">
        <h2 class="text-3xl font-bold mb-2 text-white" id="result-title">回合結果</h2>
        <p class="text-gray-300 mb-4 text-lg md:text-xl bg-gray-800 px-4 py-2 rounded-lg border border-gray-600 text-center max-w-md w-11/12" id="result-location-name">實際地點：-</p>
        
        <div id="result-map"></div>
        
        <div class="flex gap-4 md:gap-8 mb-6 text-center bg-gray-800 p-4 md:p-6 rounded-xl border border-gray-700 shadow-xl w-11/12 max-w-md justify-center">
            <div>
                <div class="text-gray-400 text-xs md:text-sm mb-1">誤差距離</div>
                <div class="text-2xl md:text-3xl font-bold text-yellow-400"><span id="distance-display">0</span> km</div>
            </div>
            <div class="w-px bg-gray-600"></div>
            <div>
                <div class="text-gray-400 text-xs md:text-sm mb-1">獲得分數</div>
                <div class="text-2xl md:text-3xl font-bold text-emerald-400">+<span id="points-display">0</span></div>
            </div>
        </div>
        <button id="next-round-btn" onclick="nextRound()" class="bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-3 px-10 rounded-full text-xl transition duration-200 shadow-lg">
            下一回合 <i class="fas fa-arrow-right ml-2"></i>
        </button>
    </div>

    <!-- 4. 遊戲結束畫面 -->
    <div id="end-screen" class="fullscreen-layer" style="display: none;">
        <div class="bg-gray-800 p-8 md:p-10 rounded-2xl shadow-2xl text-center border border-gray-700 max-w-lg w-full m-4">
            <i id="grade-icon" class="fas fa-trophy text-6xl mb-4 text-yellow-500"></i>
            <h1 class="text-4xl font-bold mb-2 text-white">遊戲結束！</h1>
            
            <div class="bg-gray-900 rounded-lg p-6 mb-6 border border-gray-700">
                <div class="text-gray-400 text-sm mb-1">總得分</div>
                <div class="text-5xl font-black text-emerald-400"><span id="final-score">0</span><span class="text-2xl text-gray-500 ml-2">/ 25000</span></div>
            </div>

            <div class="mb-8 p-5 bg-gray-700 rounded-xl shadow-inner text-left">
                <div class="text-gray-400 text-sm mb-1 uppercase tracking-wider">你的地理評級</div>
                <h2 id="grade-title" class="text-3xl font-bold mb-2 text-yellow-400">正在計算...</h2>
                <p id="grade-desc" class="text-gray-200 text-sm md:text-base">...</p>
            </div>
            
            <button onclick="resetGame()" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-4 px-6 rounded-lg text-lg transition duration-200">
                <i class="fas fa-redo mr-2"></i> 再玩一次 (全新題庫)
            </button>
        </div>
    </div>

    <script>
        // ==========================================
        // 核心邏輯 (全平台穩定修復)
        // ==========================================
        
        const hkSeedCenters = [
            {lat: 22.28, lng: 114.15}, {lat: 22.27, lng: 114.17}, {lat: 22.28, lng: 114.22},
            {lat: 22.24, lng: 114.15}, {lat: 22.31, lng: 114.17}, {lat: 22.33, lng: 114.16},
            {lat: 22.32, lng: 114.19}, {lat: 22.34, lng: 114.19}, {lat: 22.31, lng: 114.22},
            {lat: 22.37, lng: 114.11}, {lat: 22.35, lng: 114.13}, {lat: 22.39, lng: 113.97},
            {lat: 22.44, lng: 114.02}, {lat: 22.50, lng: 114.13}, {lat: 22.44, lng: 114.16},
            {lat: 22.38, lng: 114.19}, {lat: 22.38, lng: 114.27}, {lat: 22.29, lng: 113.94},
            {lat: 22.25, lng: 113.86}, {lat: 22.20, lng: 114.02}, {lat: 22.22, lng: 114.11}
        ];

        let map, resultMap, panzoomInstance;
        let guessMarker = null; let actualMarker = null; let resultLine = null;
        let currentRound = 1; const maxRounds = 5; let totalScore = 0;
        let timerInterval; let timeLeft = 120;
        let currentActualLocation = null;
        let usedLocationNames = new Set(); 

        function getDistanceFromLatLonInKm(lat1, lon1, lat2, lon2) {
            const R = 6371; 
            const dLat = (lat2 - lat1) * Math.PI / 180;
            const dLon = (lon2 - lon1) * Math.PI / 180;
            const a = Math.sin(dLat/2) * Math.sin(dLat/2) + Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) * Math.sin(dLon/2) * Math.sin(dLon/2);
            return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
        }

        // --- 修復 iPad 放大的關鍵：徹底重置縮放引擎 ---
        function initOrResetPanzoom() {
            const elem = document.getElementById('location-photo');
            const wrapper = document.getElementById('photo-wrapper');
            
            // 如果之前有縮放引擎，徹底銷毀並移除舊的 Event Listener
            if (panzoomInstance) {
                wrapper.removeEventListener('wheel', panzoomInstance.zoomWithWheel);
                panzoomInstance.destroy();
                panzoomInstance = null;
            }
            
            // 強制重置圖片的 CSS Transform，確保 1:1 比例
            elem.style.transform = 'none';
            
            // 重新建立乾淨的縮放引擎
            panzoomInstance = Panzoom(elem, { 
                maxScale: 8, 
                minScale: 1, 
                contain: 'outside', 
                step: 0.3 
            });
            wrapper.addEventListener('wheel', panzoomInstance.zoomWithWheel);
        }

        function initMaps() {
            if (map) return;
            const hkCenter = [22.3193, 114.1694];
            
            map = L.map('guess-map', { center: hkCenter, zoom: 10, zoomControl: false });
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 18, attribution: '&copy; OSM' }).addTo(map);
            map.on('click', function(e) {
                if (guessMarker) guessMarker.setLatLng(e.latlng); 
                else guessMarker = L.marker(e.latlng).addTo(map);
                
                document.getElementById('guess-btn').disabled = false;
            });

            resultMap = L.map('result-map', { center: hkCenter, zoom: 11 });
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(resultMap);
        }

        // 打開與關閉彈出式地圖
        function openMap() {
            document.getElementById('map-modal').style.display = 'flex';
            // 讓瀏覽器有時間顯示 Modal，然後強迫 Leaflet 重新計算大小 (極重要)
            setTimeout(() => { map.invalidateSize(); }, 100);
        }

        function closeMap() {
            document.getElementById('map-modal').style.display = 'none';
        }

        async function fetchRandomWikiLocation() {
            const center = hkSeedCenters[Math.floor(Math.random() * hkSeedCenters.length)];
            const url = `https://zh.wikipedia.org/w/api.php?action=query&format=json&prop=coordinates|pageimages&piprop=original&generator=geosearch&ggsradius=8000&ggscoord=${center.lat}|${center.lng}&ggslimit=50&origin=*`;

            try {
                const response = await fetch(url);
                const data = await response.json();
                if (!data.query || !data.query.pages) return null;
                
                const validPages = Object.values(data.query.pages).filter(p => {
                    if (!p.coordinates || !p.original) return false;
                    const imgUrl = p.original.source.toLowerCase();
                    const title = p.title;
                    
                    if (imgUrl.endsWith('.svg') || imgUrl.endsWith('.png') || imgUrl.endsWith('.gif')) return false;
                    if (imgUrl.includes('map') || imgUrl.includes('logo') || imgUrl.includes('icon')) return false;
                    if (title.includes('站') || title.includes('綫') || title.includes('車廠') || title.includes('交匯處')) return false;
                    
                    const scaryOrReligious = ['佛', '觀音', '神像', '廟', '寺', '大士', '菩薩', '天后'];
                    if (scaryOrReligious.some(keyword => title.includes(keyword))) return false;

                    const yearMatch = imgUrl.match(/(18|19)\d{2}/);
                    if (yearMatch) return false;
                    const oldKeywords = ['old', 'bw', 'black_and_white', 'historical', 'vintage', '舊', '昔日', '歷史'];
                    if (oldKeywords.some(keyword => imgUrl.includes(keyword) || title.includes(keyword))) return false;

                    if (usedLocationNames.has(title)) return false;

                    return true;
                });

                if (validPages.length === 0) return null;
                const selectedPage = validPages[Math.floor(Math.random() * validPages.length)];
                usedLocationNames.add(selectedPage.title);
                
                return {
                    name: selectedPage.title,
                    lat: selectedPage.coordinates[0].lat,
                    lng: selectedPage.coordinates[0].lon,
                    img: selectedPage.original.source
                };
            } catch (error) { return null; }
        }

        function startGame() {
            document.getElementById('start-screen').style.display = 'none';
            document.getElementById('game-container').style.display = 'block';
            usedLocationNames.clear(); 
            initMaps();
            startRound();
        }

        async function startRound() {
            if (currentRound > maxRounds) { showEndScreen(); return; }

            document.getElementById('round-display').innerText = `${currentRound} / ${maxRounds}`;
            document.getElementById('result-overlay').style.display = 'none';
            closeMap();
            
            if (guessMarker) { map.removeLayer(guessMarker); guessMarker = null; }
            document.getElementById('guess-btn').disabled = true;
            
            const loadingOverlay = document.getElementById('loading-overlay');
            const imgElement = document.getElementById('location-photo');
            
            clearInterval(timerInterval);
            document.getElementById('time-display').innerText = "--:--";
            imgElement.style.display = 'none'; 
            loadingOverlay.style.display = 'flex';

            let locationData = null;
            let attempts = 0;
            while (!locationData && attempts < 10) { 
                locationData = await fetchRandomWikiLocation();
                attempts++;
            }

            if (!locationData) {
                // 如果連續失敗，稍後再試，避免無窮迴圈
                setTimeout(startRound, 1000);
                return;
            }

            currentActualLocation = locationData;

            // --- 解決手機沒圖片的關鍵：確保 onload 正確觸發並處理 UI ---
            imgElement.onload = function() {
                loadingOverlay.style.display = 'none'; 
                imgElement.style.display = 'block'; 
                
                // 圖片完全載入後，才初始化縮放引擎，防止計算錯誤導致過度放大
                initOrResetPanzoom();
                
                timeLeft = 120; updateTimerDisplay();
                document.getElementById('timer-box').classList.remove('timer-warning');
                timerInterval = setInterval(() => {
                    timeLeft--; updateTimerDisplay();
                    if (timeLeft <= 10) document.getElementById('timer-box').classList.add('timer-warning');
                    if (timeLeft <= 0) { clearInterval(timerInterval); submitGuess(true); }
                }, 1000);
            };
            
            imgElement.onerror = function() {
                usedLocationNames.delete(currentActualLocation.name);
                startRound(); 
            };

            // 觸發載入
            imgElement.src = currentActualLocation.img;
        }

        function updateTimerDisplay() {
            const m = Math.floor(timeLeft / 60).toString().padStart(2, '0');
            const s = (timeLeft % 60).toString().padStart(2, '0');
            document.getElementById('time-display').innerText = `${m}:${s}`;
        }

        function submitGuess(timeUp = false) {
            clearInterval(timerInterval);
            closeMap();
            
            let gLat = 22.3, gLng = 114.17;
            if (guessMarker) { const pos = guessMarker.getLatLng(); gLat = pos.lat; gLng = pos.lng; }

            const distanceKm = getDistanceFromLatLonInKm(currentActualLocation.lat, currentActualLocation.lng, gLat, gLng);
            
            let points = Math.round(5000 * Math.exp(-distanceKm / 2.5));
            if (distanceKm < 0.1) points = 5000;
            if (timeUp && !guessMarker) points = 0;

            totalScore += points;
            document.getElementById('score-display').innerText = totalScore;
            showResultScreen(currentActualLocation.lat, currentActualLocation.lng, gLat, gLng, distanceKm, points, timeUp);
        }

        function showResultScreen(aLat, aLng, gLat, gLng, distanceKm, points, timeUp) {
            document.getElementById('result-overlay').style.display = 'flex';
            document.getElementById('result-title').innerText = timeUp && !guessMarker ? "時間到！" : "回合結果";
            
            const wikiSearchUrl = `https://www.google.com/search?q=${encodeURIComponent(currentActualLocation.name + " 香港")}`;
            document.getElementById('result-location-name').innerHTML = `
                答案：<span class="text-white font-bold">${currentActualLocation.name}</span> 
                <a href="${wikiSearchUrl}" target="_blank" class="ml-2 text-blue-400 hover:text-blue-300 text-sm underline inline-block"><i class="fas fa-external-link-alt"></i> 搜尋</a>
            `;
            
            document.getElementById('distance-display').innerText = guessMarker ? distanceKm.toFixed(2) : "-";
            document.getElementById('points-display').innerText = points;
            
            if (currentRound >= maxRounds) {
                document.getElementById('next-round-btn').innerHTML = "查看評分結果 <i class='fas fa-flag-checkered ml-2'></i>";
            } else {
                document.getElementById('next-round-btn').innerHTML = "下一回合 <i class='fas fa-arrow-right ml-2'></i>";
            }

            if (actualMarker) resultMap.removeLayer(actualMarker);
            if (resultLine) resultMap.removeLayer(resultLine);
            
            setTimeout(() => {
                resultMap.invalidateSize();
                const greenIcon = new L.Icon({ iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png', shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png', iconSize: [25, 41], iconAnchor: [12, 41] });
                actualMarker = L.marker([aLat, aLng], {icon: greenIcon}).addTo(resultMap);

                let bounds;
                if (guessMarker) {
                    const temp = L.marker([gLat, gLng]).addTo(resultMap);
                    resultLine = L.polyline([[aLat, aLng], [gLat, gLng]], {color: 'black', dashArray: '5, 10'}).addTo(resultMap);
                    bounds = L.latLngBounds([[aLat, aLng], [gLat, gLng]]);
                    guessMarker.resultMapCopy = temp;
                } else { bounds = L.latLngBounds([[aLat, aLng], [aLat, aLng]]); }
                
                resultMap.fitBounds(bounds, {padding: [50, 50], maxZoom: 15});
            }, 100);
        }

        function nextRound() { currentRound++; startRound(); }

        function showEndScreen() {
            document.getElementById('game-container').style.display = 'none';
            document.getElementById('end-screen').style.display = 'flex';
            document.getElementById('final-score').innerText = totalScore;

            let title, desc, iconClass, colorClass;

            if (totalScore >= 20000) {
                title = "香港地理達人 (S級)";
                desc = "太強了！你簡直係人肉 Google Map，全香港每一條街都難唔到你！";
                iconClass = "fa-crown"; colorClass = "text-yellow-400";
            } else if (totalScore >= 15000) {
                title = "地道香港人 (A級)";
                desc = "你非常熟悉香港地理，平時一定成日四處去玩，方向感極佳！";
                iconClass = "fa-medal"; colorClass = "text-emerald-400";
            } else if (totalScore >= 10000) {
                title = "普通市民 (B級)";
                desc = "對自己住嗰區或者旺區好熟，但去到新界或者離島可能就會迷路啦。";
                iconClass = "fa-thumbs-up"; colorClass = "text-blue-400";
            } else if (totalScore >= 5000) {
                title = "週末遊客 (C級)";
                desc = "你淨係認得啲出名地標，快啲放假去多啲香港唔同地方行下啦！";
                iconClass = "fa-suitcase-rolling"; colorClass = "text-orange-400";
            } else {
                title = "超級路痴 (D級)";
                desc = "你需要強烈溫書！你確定你平時出街唔會蕩失路？😂";
                iconClass = "fa-map-signs"; colorClass = "text-red-400";
            }

            const gradeTitleEl = document.getElementById('grade-title');
            const gradeIconEl = document.getElementById('grade-icon');
            
            gradeTitleEl.innerText = title;
            gradeTitleEl.className = `text-3xl font-bold mb-2 ${colorClass}`;
            document.getElementById('grade-desc').innerText = desc;
            gradeIconEl.className = `fas ${iconClass} text-6xl mb-4 ${colorClass}`;
        }

        function resetGame() {
            currentRound = 1; totalScore = 0;
            document.getElementById('score-display').innerText = "0";
            document.getElementById('end-screen').style.display = 'none';
            usedLocationNames.clear(); 
            document.getElementById('game-container').style.display = 'block';
            startRound();
        }
    </script>
</body>
</html>
