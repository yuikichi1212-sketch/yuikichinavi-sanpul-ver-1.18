<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ゆいきちナビ</title>

    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

    <link rel="stylesheet" href="https://unpkg.com/leaflet-routing-machine@3.2.12/dist/leaflet-routing-machine.css" />
    <script src="https://unpkg.com/leaflet-routing-machine@3.2.12/dist/leaflet-routing-machine.js"></script>

    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        /* ========== テーマ設定 (完全な白ベース) ========== */
        :root {
            --pure-white: #FFFFFF;
            --text-main: #1C1C1E;
            --text-sub: #8E8E93;
            --accent-blue: #007AFF; /* Apple風の青 */
            --shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
            --border-light: #E5E5EA;
        }

        * { box-sizing: border-box; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        body, html { margin: 0; padding: 0; height: 100%; overflow: hidden; background: var(--pure-white); color: var(--text-main); }

        /* ========== 1. ゆいきちアカウント (白テーマ) ========== */
        #auth-screen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: var(--pure-white);
            display: flex; justify-content: center; align-items: center; z-index: 9999;
        }
        .auth-card {
            width: 90%; max-width: 400px; padding: 40px 30px; text-align: center;
        }
        .auth-logo { font-size: 28px; font-weight: 800; margin-bottom: 30px; color: var(--text-main); letter-spacing: 1px; }
        .auth-step { display: none; animation: fadeIn 0.3s ease forwards; }
        .auth-step.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        .auth-input {
            width: 100%; padding: 16px; margin-bottom: 10px; border: 1px solid var(--border-light);
            border-radius: 12px; font-size: 16px; outline: none; background: #F2F2F7;
        }
        .auth-input:focus { border-color: var(--accent-blue); background: var(--pure-white); }

        .auth-btn {
            width: 100%; padding: 16px; background: var(--accent-blue); color: var(--pure-white);
            border: none; border-radius: 12px; font-size: 16px; font-weight: bold;
            cursor: pointer; margin-top: 10px; transition: 0.2s;
        }
        .auth-btn:active { transform: scale(0.96); }
        .auth-btn-outline { background: var(--pure-white); color: var(--accent-blue); border: 2px solid var(--accent-blue); margin-top: 15px; }
        .auth-btn-text { background: transparent; color: var(--text-sub); font-size: 14px; margin-top: 15px; border: none; cursor: pointer; }
        .error-msg { color: #FF3B30; font-size: 13px; min-height: 20px; margin-bottom: 10px; text-align: left; }

        /* ========== 2. ナビアプリ本体 ========== */
        #app-screen { display: none; position: relative; width: 100%; height: 100%; }
        #map { width: 100%; height: 100%; z-index: 1; background: #F9F9F9; }

        /* 検索バー (Google風) */
        .search-container {
            position: absolute; top: 40px; left: 50%; transform: translateX(-50%);
            width: 90%; max-width: 500px; z-index: 1000; display: flex; gap: 10px;
        }
        .search-box {
            flex: 1; background: var(--pure-white); border-radius: 25px; padding: 12px 20px;
            box-shadow: var(--shadow); border: 1px solid var(--border-light); display: flex; align-items: center;
        }
        .search-box input { border: none; outline: none; flex: 1; font-size: 16px; margin-left: 10px; background: transparent; }
        .search-btn {
            background: var(--accent-blue); color: var(--pure-white); border: none;
            width: 50px; height: 50px; border-radius: 25px; box-shadow: var(--shadow);
            cursor: pointer; display: flex; justify-content: center; align-items: center; font-size: 18px;
        }

        /* アカウントアイコン */
        .profile-btn {
            position: absolute; top: 100px; right: 20px; z-index: 1000;
            width: 45px; height: 45px; background: var(--pure-white); border-radius: 50%;
            box-shadow: var(--shadow); border: 1px solid var(--border-light);
            display: flex; justify-content: center; align-items: center;
            cursor: pointer; font-size: 18px; color: var(--text-main);
        }

        /* 現在地ボタン */
        .locate-btn {
            position: absolute; top: 160px; right: 20px; z-index: 1000;
            width: 45px; height: 45px; background: var(--pure-white); border-radius: 50%;
            box-shadow: var(--shadow); border: 1px solid var(--border-light);
            display: flex; justify-content: center; align-items: center;
            cursor: pointer; font-size: 18px; color: var(--accent-blue);
        }

        /* Apple風ボトムシート (検索結果 & ナビ開始) */
        .bottom-sheet {
            position: absolute; bottom: 0; left: 0; width: 100%;
            background: var(--pure-white); border-radius: 24px 24px 0 0;
            padding: 24px 20px 40px 20px; box-shadow: 0 -10px 30px rgba(0,0,0,0.08);
            z-index: 1000; transform: translateY(100%); transition: transform 0.3s cubic-bezier(0.1, 0.8, 0.2, 1);
        }
        .bottom-sheet.show { transform: translateY(0); }
        .sheet-handle { width: 40px; height: 5px; background: #E5E5EA; border-radius: 5px; margin: 0 auto 20px auto; }
        
        .place-name { font-size: 22px; font-weight: bold; margin-bottom: 5px; }
        .place-address { font-size: 14px; color: var(--text-sub); margin-bottom: 20px; }
        
        /* ナビゲーション情報 */
        .nav-stats { display: none; align-items: baseline; gap: 15px; margin-bottom: 20px; padding: 15px; background: #F2F2F7; border-radius: 12px;}
        .nav-time { font-size: 28px; font-weight: 800; color: #34C759; } /* 緑色 */
        .nav-dist { font-size: 16px; font-weight: 600; color: var(--text-main); }

        .start-nav-btn {
            width: 100%; padding: 16px; background: var(--accent-blue); color: var(--pure-white);
            border: none; border-radius: 16px; font-size: 18px; font-weight: bold;
            cursor: pointer; display: flex; justify-content: center; align-items: center; gap: 10px;
        }

        /* 余計なルート案内テキストを隠す */
        .leaflet-routing-container { display: none !important; }
        
        /* プロフィールモーダル */
        #profile-modal {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.3); z-index: 2000; justify-content: center; align-items: center;
        }
        .modal-content {
            background: var(--pure-white); width: 85%; max-width: 350px; padding: 30px;
            border-radius: 20px; text-align: left; box-shadow: var(--shadow); position: relative;
        }
        .close-modal { position: absolute; top: 15px; right: 20px; font-size: 24px; cursor: pointer; color: var(--text-sub); }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="auth-card">
            <div class="auth-logo">ゆいきちナビ Pro</div>
            
            <div id="step-start" class="auth-step active">
                <button class="auth-btn" onclick="showAuthStep('reg-user')">新しいアカウントを作成</button>
                <button class="auth-btn auth-btn-outline" onclick="showAuthStep('login-user')">ログイン</button>
            </div>

            <div id="step-reg-user" class="auth-step">
                <h3 style="margin-top:0;">アカウント名</h3>
                <input type="text" id="reg-username" class="auth-input" placeholder="5文字以上で入力">
                <div id="err-reg-user" class="error-msg"></div>
                <button class="auth-btn" onclick="checkRegUser()">次へ</button>
                <button class="auth-btn-text" onclick="showAuthStep('start')">キャンセル</button>
            </div>

            <div id="step-reg-pass" class="auth-step">
                <h3 style="margin-top:0;">パスワード</h3>
                <input type="password" id="reg-password" class="auth-input" placeholder="英字と数字を含めて5桁以上">
                <div id="err-reg-pass" class="error-msg"></div>
                <button class="auth-btn" onclick="registerUser()">登録して開始</button>
                <button class="auth-btn-text" onclick="showAuthStep('reg-user')">戻る</button>
            </div>

            <div id="step-login-user" class="auth-step">
                <h3 style="margin-top:0;">ログイン</h3>
                <input type="text" id="login-username" class="auth-input" placeholder="アカウント名">
                <div id="err-login-user" class="error-msg"></div>
                <button class="auth-btn" onclick="checkLoginUser()">次へ</button>
                <button class="auth-btn-text" onclick="showAuthStep('start')">キャンセル</button>
            </div>

            <div id="step-login-pass" class="auth-step">
                <h3 style="margin-top:0;">パスワード</h3>
                <input type="password" id="login-password" class="auth-input" placeholder="パスワード">
                <div id="err-login-pass" class="error-msg"></div>
                <button class="auth-btn" onclick="loginUser()">ログイン</button>
                <button class="auth-btn-text" onclick="showAuthStep('login-user')">戻る</button>
            </div>
        </div>
    </div>

    <div id="app-screen">
        <div id="map"></div>

        <div class="search-container">
            <div class="search-box">
                <i class="fa-solid fa-bars" style="color: var(--text-sub); margin-right: 5px;"></i>
                <input type="text" id="search-input" placeholder="場所や住所を検索...">
            </div>
            <button class="search-btn" onclick="searchPlace()"><i class="fa-solid fa-magnifying-glass"></i></button>
        </div>

        <div class="profile-btn" onclick="openProfile()"><i class="fa-solid fa-user"></i></div>
        <div class="locate-btn" onclick="locateUser()"><i class="fa-solid fa-location-crosshairs"></i></div>

        <div class="bottom-sheet" id="bottom-sheet">
            <div class="sheet-handle"></div>
            <div class="place-name" id="bs-name">検索結果</div>
            <div class="place-address" id="bs-address">---</div>
            
            <div class="nav-stats" id="nav-stats">
                <div class="nav-time" id="nav-time">--分</div>
                <div class="nav-dist" id="nav-dist">-- km</div>
            </div>

            <button class="start-nav-btn" id="start-nav-btn" onclick="startNavigation()">
                <i class="fa-solid fa-location-arrow"></i> 経路を検索・ナビ開始
            </button>
            <button class="auth-btn-text" style="width:100%; margin-top:10px;" onclick="closeBottomSheet()">閉じる</button>
        </div>
    </div>

    <div id="profile-modal">
        <div class="modal-content">
            <div class="close-modal" onclick="closeProfile()">&times;</div>
            <h2 style="margin-top: 0; border-bottom: 1px solid #eee; padding-bottom: 10px;">アカウント情報</h2>
            <p style="color: var(--text-sub); font-size: 14px;">アカウント名</p>
            <p style="font-size: 18px; font-weight: bold; margin-top: -5px;" id="disp-username">---</p>
            <p style="color: var(--text-sub); font-size: 14px;">パスワード</p>
            <p style="font-size: 18px; font-weight: bold; margin-top: -5px;" id="disp-password">---</p>
            <button class="auth-btn" style="background: #FF3B30;" onclick="logoutApp()">ログアウト</button>
        </div>
    </div>

    <script>
        /* --- アカウント認証ロジック --- */
        const DB_KEY = 'yuikichi_db';
        let tempUser = '';

        window.onload = () => {
            const loggedInUser = localStorage.getItem('yuikichi_logged_in');
            if (loggedInUser) startApp(loggedInUser);
        };

        function showAuthStep(stepId) {
            document.querySelectorAll('.auth-step').forEach(el => el.classList.remove('active'));
            document.getElementById(`step-${stepId}`).classList.add('active');
            document.querySelectorAll('.error-msg').forEach(el => el.innerText = '');
        }

        function checkRegUser() {
            const val = document.getElementById('reg-username').value;
            const err = document.getElementById('err-reg-user');
            if (val.length < 5) return err.innerText = '5文字以上で入力してください';
            const db = JSON.parse(localStorage.getItem(DB_KEY) || '{}');
            if (db[val]) return err.innerText = 'その名前はすでに使われています';
            tempUser = val; showAuthStep('reg-pass');
        }

        function registerUser() {
            const val = document.getElementById('reg-password').value;
            const err = document.getElementById('err-reg-pass');
            if (!/^(?=.*[A-Za-z])(?=.*\d).{5,}$/.test(val)) return err.innerText = '英字と数字を含めて5桁以上にしてください';
            const db = JSON.parse(localStorage.getItem(DB_KEY) || '{}');
            db[tempUser] = { password: val };
            localStorage.setItem(DB_KEY, JSON.stringify(db));
            localStorage.setItem('yuikichi_logged_in', tempUser);
            startApp(tempUser);
        }

        function checkLoginUser() {
            const val = document.getElementById('login-username').value;
            const err = document.getElementById('err-login-user');
            const db = JSON.parse(localStorage.getItem(DB_KEY) || '{}');
            if (!db[val]) return err.innerText = 'アカウントが見つかりません';
            tempUser = val; showAuthStep('login-pass');
        }

        function loginUser() {
            const val = document.getElementById('login-password').value;
            const err = document.getElementById('err-login-pass');
            const db = JSON.parse(localStorage.getItem(DB_KEY) || '{}');
            if (db[tempUser].password !== val) return err.innerText = 'パスワードが違います';
            localStorage.setItem('yuikichi_logged_in', tempUser);
            startApp(tempUser);
        }

        function logoutApp() {
            localStorage.removeItem('yuikichi_logged_in');
            document.getElementById('profile-modal').style.display = 'none';
            document.getElementById('app-screen').style.display = 'none';
            document.getElementById('auth-screen').style.display = 'flex';
            showAuthStep('start');
            document.getElementById('login-password').value = '';
            document.getElementById('reg-password').value = '';
        }

        /* --- マップ＆ナビロジック --- */
        let map, userMarker, searchMarker, routingControl;
        let userLat = 35.6812, userLng = 139.7671; // 初期値
        let targetLat = null, targetLng = null;

        function startApp(username) {
            document.getElementById('auth-screen').style.display = 'none';
            document.getElementById('app-screen').style.display = 'block';
            
            const db = JSON.parse(localStorage.getItem(DB_KEY) || '{}');
            document.getElementById('disp-username').innerText = username;
            document.getElementById('disp-password').innerText = db[username].password;

            if (!map) initMap();
        }

        function initMap() {
            // 絶対に読み込める標準のOpenStreetMapを採用
            map = L.map('map', { zoomControl: false }).setView([userLat, userLng], 15);
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '&copy; OpenStreetMap contributors'
            }).addTo(map);

            locateUser();
        }

        // 現在地取得
        function locateUser() {
            if (!navigator.geolocation) return alert("GPS非対応のブラウザです。");
            navigator.geolocation.getCurrentPosition(pos => {
                userLat = pos.coords.latitude; userLng = pos.coords.longitude;
                
                if (!userMarker) {
                    const icon = L.divIcon({
                        className: 'dummy',
                        html: '<div style="width:20px;height:20px;background:#007AFF;border:3px solid white;border-radius:50%;box-shadow:0 0 10px rgba(0,0,0,0.3);"></div>',
                        iconSize: [26, 26], iconAnchor: [13, 13]
                    });
                    userMarker = L.marker([userLat, userLng], {icon: icon}).addTo(map);
                } else {
                    userMarker.setLatLng([userLat, userLng]);
                }
                map.flyTo([userLat, userLng], 16);
            }, () => { alert("位置情報が取得できませんでした。"); });
        }

        // 検索機能 -> ボトムシート表示
        async function searchPlace() {
            const query = document.getElementById('search-input').value;
            if (!query) return;

            const btn = document.querySelector('.search-btn');
            btn.innerHTML = '<i class="fa-solid fa-spinner fa-spin"></i>';

            try {
                // Nominatim APIで検索
                const res = await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(query)}`);
                const data = await res.json();
                
                if (data.length > 0) {
                    targetLat = parseFloat(data[0].lat);
                    targetLng = parseFloat(data[0].lon);
                    const displayName = data[0].display_name.split(',')[0]; // 最初の名前だけ取得
                    
                    if (searchMarker) map.removeLayer(searchMarker);
                    searchMarker = L.marker([targetLat, targetLng]).addTo(map);
                    
                    map.flyTo([targetLat, targetLng], 16);

                    // ボトムシートの更新と表示
                    document.getElementById('bs-name').innerText = query;
                    document.getElementById('bs-address').innerText = data[0].display_name;
                    document.getElementById('nav-stats').style.display = 'none'; // ルート検索前は隠す
                    document.getElementById('start-nav-btn').innerHTML = '<i class="fa-solid fa-location-arrow"></i> 経路を検索・ナビ開始';
                    document.getElementById('bottom-sheet').classList.add('show');
                    
                    // 既存のルートがあれば消す
                    if (routingControl) { map.removeControl(routingControl); routingControl = null; }

                } else {
                    alert("見つかりませんでした。");
                }
            } catch (e) {
                alert("検索エラーが発生しました。");
            }
            btn.innerHTML = '<i class="fa-solid fa-magnifying-glass"></i>';
        }

        // ナビ開始 (ルート検索)
        function startNavigation() {
            if (!targetLat || !targetLng) return;

            const btn = document.getElementById('start-nav-btn');
            btn.innerHTML = '<i class="fa-solid fa-spinner fa-spin"></i> ルート計算中...';

            if (routingControl) map.removeControl(routingControl);

            // ルート検索の実行
            routingControl = L.Routing.control({
                waypoints: [ L.latLng(userLat, userLng), L.latLng(targetLat, targetLng) ],
                router: L.Routing.osrmv1({ serviceUrl: 'https://router.project-osrm.org/route/v1' }),
                lineOptions: { styles: [{color: '#007AFF', opacity: 0.8, weight: 6}] },
                createMarker: function() { return null; },
                addWaypoints: false, draggableWaypoints: false, fitSelectedRoutes: true
            }).on('routesfound', function(e) {
                // 計算結果をボトムシートに表示
                const summary = e.routes[0].summary;
                const timeMin = Math.round(summary.totalTime / 60);
                const distKm = (summary.totalDistance / 1000).toFixed(1);
                
                document.getElementById('nav-time').innerText = timeMin + "分";
                document.getElementById('nav-dist').innerText = distKm + " km";
                document.getElementById('nav-stats').style.display = 'flex';
                
                btn.innerHTML = '<i class="fa-solid fa-car"></i> 出発！ (案内中)';
                btn.style.background = '#34C759'; // 緑色に変更
                
            }).addTo(map);
        }

        function closeBottomSheet() { document.getElementById('bottom-sheet').classList.remove('show'); }
        function openProfile() { document.getElementById('profile-modal').style.display = 'flex'; }
        function closeProfile() { document.getElementById('profile-modal').style.display = 'none'; }
    </script>
</body>
</html>
