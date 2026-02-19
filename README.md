<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ゆいきちナビ - 3D Navi App</title>

    <script src="https://unpkg.com/maplibre-gl@3.3.1/dist/maplibre-gl.js"></script>
    <link href="https://unpkg.com/maplibre-gl@3.3.1/dist/maplibre-gl.css" rel="stylesheet" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        /* ========== テーマカラー設定 ========== */
        :root {
            --primary-yellow: #FFD54F; /* 優しい黄色 */
            --light-yellow: #FFFDE7;
            --white: #FFFFFF;
            --text-dark: #4E342E; /* 焦げ茶色で優しい印象に */
            --error-red: #FF5252;
            --shadow: 0 8px 30px rgba(0, 0, 0, 0.1);
        }

        * { box-sizing: border-box; font-family: 'Nunito', 'Rounded Mplus 1c', sans-serif; }
        body, html { margin: 0; padding: 0; height: 100%; overflow: hidden; background: var(--light-yellow); color: var(--text-dark); }

        /* ========== 1. ゆいきちアカウント認証画面 ========== */
        #auth-screen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(135deg, var(--light-yellow) 0%, #FFF9C4 100%);
            display: flex; justify-content: center; align-items: center; z-index: 9999;
        }

        .auth-card {
            background: var(--white); width: 90%; max-width: 400px; padding: 40px 30px;
            border-radius: 30px; box-shadow: var(--shadow); text-align: center;
            position: relative; overflow: hidden;
        }

        .auth-logo { font-size: 24px; font-weight: bold; margin-bottom: 30px; color: var(--primary-yellow); }

        .auth-step { display: none; animation: slideIn 0.4s ease forwards; }
        .auth-step.active { display: block; }

        @keyframes slideIn { from { opacity: 0; transform: translateX(20px); } to { opacity: 1; transform: translateX(0); } }

        .auth-input {
            width: 100%; padding: 15px; margin-bottom: 10px; border: 2px solid #EEE;
            border-radius: 15px; font-size: 16px; outline: none; transition: 0.3s;
        }
        .auth-input:focus { border-color: var(--primary-yellow); }

        .auth-btn {
            width: 100%; padding: 15px; background: var(--primary-yellow); color: var(--text-dark);
            border: none; border-radius: 15px; font-size: 16px; font-weight: bold;
            cursor: pointer; margin-top: 10px; box-shadow: 0 4px 15px rgba(255, 213, 79, 0.4);
            transition: 0.2s;
        }
        .auth-btn:active { transform: scale(0.95); }
        .auth-btn-outline { background: transparent; border: 2px solid var(--primary-yellow); margin-top: 15px; box-shadow: none; }
        .auth-btn-text { background: transparent; box-shadow: none; color: #888; font-size: 14px; margin-top: 10px; }
        
        .error-msg { color: var(--error-red); font-size: 13px; min-height: 20px; margin-bottom: 10px; text-align: left; padding-left: 5px; }

        /* ========== 2. ナビアプリ画面 ========== */
        #app-screen { display: none; position: relative; width: 100%; height: 100%; }
        #map { width: 100%; height: 100%; }

        /* 検索バー */
        .search-container {
            position: absolute; top: 20px; left: 50%; transform: translateX(-50%);
            width: 90%; max-width: 500px; z-index: 10; display: flex; gap: 10px;
        }
        .search-box {
            flex: 1; background: var(--white); border-radius: 25px; padding: 12px 20px;
            box-shadow: var(--shadow); display: flex; align-items: center; border: 2px solid var(--primary-yellow);
        }
        .search-box input { border: none; outline: none; flex: 1; font-size: 16px; margin-left: 10px; color: var(--text-dark); background: transparent;}
        .search-btn {
            background: var(--primary-yellow); color: var(--text-dark); border: none;
            width: 50px; height: 50px; border-radius: 25px; box-shadow: var(--shadow);
            cursor: pointer; display: flex; justify-content: center; align-items: center; font-size: 18px;
        }

        /* アカウントアイコン */
        .profile-btn {
            position: absolute; top: 80px; right: 20px; z-index: 10;
            width: 50px; height: 50px; background: var(--white); border-radius: 25px;
            box-shadow: var(--shadow); display: flex; justify-content: center; align-items: center;
            cursor: pointer; font-size: 20px; color: var(--primary-yellow); border: 2px solid var(--primary-yellow);
        }

        /* 下部コントロール (3Dナビ開始ボタン) */
        .bottom-controls {
            position: absolute; bottom: 30px; left: 50%; transform: translateX(-50%); z-index: 10; width: 90%; max-width: 400px;
        }
        .navi-mode-btn {
            width: 100%; padding: 18px; background: var(--white); color: var(--text-dark);
            border: 3px solid var(--primary-yellow); border-radius: 25px; font-size: 18px; font-weight: bold;
            box-shadow: var(--shadow); cursor: pointer; transition: 0.3s;
            display: flex; justify-content: center; align-items: center; gap: 10px;
        }
        .navi-mode-btn.active { background: var(--primary-yellow); color: var(--text-dark); border-color: var(--white); }

        /* カスタムの現在地マーカー（矢印） */
        .user-arrow {
            width: 0; height: 0;
            border-left: 12px solid transparent; border-right: 12px solid transparent; border-bottom: 25px solid #FF5252;
            transform-origin: center 15px; filter: drop-shadow(0 4px 6px rgba(0,0,0,0.4));
            transition: transform 0.1s ease-out;
        }

        /* アカウント情報モーダル */
        #profile-modal {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.4); z-index: 2000; justify-content: center; align-items: center;
            backdrop-filter: blur(5px);
        }
        .modal-content {
            background: var(--light-yellow); width: 85%; max-width: 350px; padding: 30px;
            border-radius: 25px; text-align: center; box-shadow: var(--shadow); position: relative;
        }
        .close-modal { position: absolute; top: 15px; right: 20px; font-size: 24px; cursor: pointer; color: #888; }
        .info-box { background: var(--white); padding: 15px; border-radius: 15px; margin-bottom: 15px; text-align: left; border: 1px solid #EEE;}
        .info-label { font-size: 12px; color: #888; margin-bottom: 5px; }
        .info-value { font-size: 18px; font-weight: bold; }
        .logout-btn { background: var(--error-red); color: white; width: 100%; padding: 15px; border: none; border-radius: 15px; font-weight: bold; font-size: 16px; margin-top: 10px; cursor: pointer; }

    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="auth-card">
            <div class="auth-logo">🌟 ゆいきちアカウント</div>
            
            <div id="step-start" class="auth-step active">
                <p style="margin-bottom: 20px; font-size: 14px;">アプリを利用するには<br>ログインが必要です。</p>
                <button class="auth-btn" onclick="showAuthStep('reg-user')">新しくアカウントを作る</button>
                <button class="auth-btn auth-btn-outline" onclick="showAuthStep('login-user')">ログインする</button>
            </div>

            <div id="step-reg-user" class="auth-step">
                <h3>アカウント名を決める</h3>
                <p style="font-size: 12px; color: #888;">5文字以上で入力してください</p>
                <input type="text" id="reg-username" class="auth-input" placeholder="アカウント名">
                <div id="err-reg-user" class="error-msg"></div>
                <button class="auth-btn" onclick="checkRegUser()">次へ進む</button>
                <button class="auth-btn-text" onclick="showAuthStep('start')">最初に戻る</button>
            </div>

            <div id="step-reg-pass" class="auth-step">
                <h3>パスワードを決める</h3>
                <p style="font-size: 12px; color: #888;">文字と数字を両方含めて5桁以上</p>
                <input type="password" id="reg-password" class="auth-input" placeholder="パスワード">
                <div id="err-reg-pass" class="error-msg"></div>
                <button class="auth-btn" onclick="registerUser()">アカウント作成＆開始！</button>
                <button class="auth-btn-text" onclick="showAuthStep('reg-user')">戻る</button>
            </div>

            <div id="step-login-user" class="auth-step">
                <h3>ログイン</h3>
                <input type="text" id="login-username" class="auth-input" placeholder="アカウント名">
                <div id="err-login-user" class="error-msg"></div>
                <button class="auth-btn" onclick="checkLoginUser()">次へ</button>
                <button class="auth-btn-text" onclick="showAuthStep('start')">最初に戻る</button>
            </div>

            <div id="step-login-pass" class="auth-step">
                <h3>パスワード入力</h3>
                <input type="password" id="login-password" class="auth-input" placeholder="パスワード">
                <div id="err-login-pass" class="error-msg"></div>
                <button class="auth-btn" onclick="loginUser()">ログインして開始！</button>
                <button class="auth-btn-text" onclick="showAuthStep('login-user')">戻る</button>
            </div>
        </div>
    </div>

    <div id="app-screen">
        <div id="map"></div>

        <div class="search-container">
            <div class="search-box">
                <i class="fa-solid fa-magnifying-glass" style="color: var(--primary-yellow);"></i>
                <input type="text" id="search-input" placeholder="場所や施設を検索...">
            </div>
            <button class="search-btn" onclick="searchPlace()"><i class="fa-solid fa-location-arrow"></i></button>
        </div>

        <div class="profile-btn" onclick="openProfile()">
            <i class="fa-solid fa-user"></i>
        </div>

        <div class="bottom-controls">
            <button id="navi-btn" class="navi-mode-btn" onclick="toggleNaviMode()">
                <i class="fa-solid fa-cube"></i> 3Dナビモード開始
            </button>
        </div>
    </div>

    <div id="profile-modal">
        <div class="modal-content">
            <div class="close-modal" onclick="closeProfile()">&times;</div>
            <h2 style="color: var(--primary-yellow); margin-top: 0;">アカウント情報</h2>
            
            <div class="info-box">
                <div class="info-label">アカウント名</div>
                <div class="info-value" id="disp-username">---</div>
            </div>
            
            <div class="info-box">
                <div class="info-label">パスワード (秘密🤫)</div>
                <div class="info-value" id="disp-password">---</div>
            </div>

            <button class="logout-btn" onclick="logoutApp()">ログアウトする</button>
        </div>
    </div>

    <script>
        /* ----------------------------------------------------------------
           A. アカウント認証ロジック (localStorageを用いたシミュレーション)
        ----------------------------------------------------------------- */
        const DB_KEY = 'yuikichi_db';
        let tempUser = '';

        // 初期起動チェック
        window.onload = () => {
            const loggedInUser = localStorage.getItem('yuikichi_logged_in');
            if (loggedInUser) { startApp(loggedInUser); }
        };

        function showAuthStep(stepId) {
            document.querySelectorAll('.auth-step').forEach(el => el.classList.remove('active'));
            document.getElementById(`step-${stepId}`).classList.add('active');
            document.querySelectorAll('.error-msg').forEach(el => el.innerText = '');
        }

        // --- 新規登録 ---
        function checkRegUser() {
            const val = document.getElementById('reg-username').value;
            const err = document.getElementById('err-reg-user');
            if (val.length < 5) return err.innerText = 'アカウント名は5文字以上にしてください！';
            
            const db = JSON.parse(localStorage.getItem(DB_KEY) || '{}');
            if (db[val]) return err.innerText = 'その名前はすでに使われています！';
            
            tempUser = val;
            showAuthStep('reg-pass');
        }

        function registerUser() {
            const val = document.getElementById('reg-password').value;
            const err = document.getElementById('err-reg-pass');
            // 正規表現: 英字と数字を両方含み、5文字以上
            const regex = /^(?=.*[A-Za-z])(?=.*\d).{5,}$/;
            
            if (!regex.test(val)) return err.innerText = '文字と数字を両方入れて5桁以上にしてください！';
            
            const db = JSON.parse(localStorage.getItem(DB_KEY) || '{}');
            db[tempUser] = { password: val }; // データ保存
            localStorage.setItem(DB_KEY, JSON.stringify(db));
            localStorage.setItem('yuikichi_logged_in', tempUser); // ログイン状態にする
            startApp(tempUser);
        }

        // --- ログイン ---
        function checkLoginUser() {
            const val = document.getElementById('login-username').value;
            const err = document.getElementById('err-login-user');
            const db = JSON.parse(localStorage.getItem(DB_KEY) || '{}');
            
            if (!db[val]) return err.innerText = 'アカウントが見つかりません。間違ってない？';
            
            tempUser = val;
            showAuthStep('login-pass');
        }

        function loginUser() {
            const val = document.getElementById('login-password').value;
            const err = document.getElementById('err-login-pass');
            const db = JSON.parse(localStorage.getItem(DB_KEY) || '{}');
            
            if (db[tempUser].password !== val) return err.innerText = 'パスワードが違います！';
            
            localStorage.setItem('yuikichi_logged_in', tempUser);
            startApp(tempUser);
        }

        // --- ログアウト ---
        function logoutApp() {
            localStorage.removeItem('yuikichi_logged_in');
            document.getElementById('profile-modal').style.display = 'none';
            document.getElementById('app-screen').style.display = 'none';
            document.getElementById('auth-screen').style.display = 'flex';
            document.getElementById('login-password').value = '';
            showAuthStep('start');
        }


        /* ----------------------------------------------------------------
           B. ナビアプリ・マップロジック
        ----------------------------------------------------------------- */
        let map, userMarker, userArrowEl;
        let isNaviMode = false;
        let mapInitialized = false;
        let currentUserLat = 35.6812, currentUserLng = 139.7671; // 初期値(東京)

        function startApp(username) {
            document.getElementById('auth-screen').style.display = 'none';
            document.getElementById('app-screen').style.display = 'block';
            
            // アカウント情報をモーダルにセット
            const db = JSON.parse(localStorage.getItem(DB_KEY) || '{}');
            document.getElementById('disp-username').innerText = username;
            document.getElementById('disp-password').innerText = db[username].password;

            if (!mapInitialized) initMap();
        }

        function initMap() {
            // 白基調の地図スタイル (CARTO Positron) を使用し、黄色テーマに合わせる
            map = new maplibregl.Map({
                container: 'map',
                style: 'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
                center: [currentUserLng, currentUserLat],
                zoom: 15,
                pitch: 0 // 最初は真上から
            });

            // カスタム矢印マーカーの作成
            userArrowEl = document.createElement('div');
            userArrowEl.className = 'user-arrow';
            userMarker = new maplibregl.Marker({ element: userArrowEl, rotationAlignment: 'map' })
                .setLngLat([currentUserLng, currentUserLat])
                .addTo(map);

            // 現在地の継続取得
            if (navigator.geolocation) {
                navigator.geolocation.watchPosition(pos => {
                    currentUserLat = pos.coords.latitude;
                    currentUserLng = pos.coords.longitude;
                    userMarker.setLngLat([currentUserLng, currentUserLat]);
                    
                    if (isNaviMode) { // ナビモード中は現在地を常に画面中央に
                        map.setCenter([currentUserLng, currentUserLat]);
                    }
                }, err => console.log(err), { enableHighAccuracy: true });
            }
            mapInitialized = true;
        }

        // --- 検索機能 (完全無料で動くNominatim APIを使用) ---
        async function searchPlace() {
            const query = document.getElementById('search-input').value;
            if (!query) return;
            
            const btn = document.querySelector('.search-btn');
            btn.innerHTML = '<i class="fa-solid fa-spinner fa-spin"></i>'; // ロード中アイコン

            try {
                const res = await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(query)}`);
                const data = await res.json();
                
                if (data.length > 0) {
                    const lat = parseFloat(data[0].lat);
                    const lon = parseFloat(data[0].lon);
                    
                    // 検索結果に黄色いピンを立てる
                    new maplibregl.Marker({ color: '#FFD54F' })
                        .setLngLat([lon, lat])
                        .setPopup(new maplibregl.Popup().setText(data[0].display_name))
                        .addTo(map)
                        .togglePopup();
                        
                    map.flyTo({ center: [lon, lat], zoom: 16 });
                } else {
                    alert("場所が見つかりませんでした...");
                }
            } catch (e) {
                alert("検索エラーが発生しました。");
            }
            btn.innerHTML = '<i class="fa-solid fa-location-arrow"></i>';
        }

        // --- スマホの向き（コンパス）連動機能 ---
        function handleOrientation(event) {
            if (!isNaviMode) return;
            
            let heading = 0;
            // iOS用の絶対方位
            if (event.webkitCompassHeading) {
                heading = event.webkitCompassHeading;
            } 
            // Android等 (北を基準としたalpha値)
            else if (event.alpha !== null) {
                heading = 360 - event.alpha;
            }

            // マーカー（矢印）を進行方向に向ける
            userArrowEl.style.transform = `rotate(${heading}deg)`;
            
            // 地図自体を進行方向に向ける（カーナビのように奥が進行方向になる）
            map.setBearing(heading);
        }

        // --- 3Dナビモードの切り替え ---
        function toggleNaviMode() {
            isNaviMode = !isNaviMode;
            const btn = document.getElementById('navi-btn');

            if (isNaviMode) {
                btn.classList.add('active');
                btn.innerHTML = '<i class="fa-solid fa-compass"></i> 3Dナビ中 (方向連動)';
                
                // 地図を傾けて3D視点（鳥瞰図）にし、現在地にズーム
                map.flyTo({
                    center: [currentUserLng, currentUserLat],
                    zoom: 18,
                    pitch: 65, // ここで地図を寝かせて3D感を出す！
                    duration: 1500
                });

                // デバイスの向きセンサーの許可を要求 (iOS 13+対応)
                if (typeof DeviceOrientationEvent !== 'undefined' && typeof DeviceOrientationEvent.requestPermission === 'function') {
                    DeviceOrientationEvent.requestPermission()
                        .then(response => {
                            if (response == 'granted') {
                                window.addEventListener('deviceorientation', handleOrientation);
                            } else {
                                alert("方向センサーの許可が必要です。");
                            }
                        }).catch(console.error);
                } else {
                    // Android等の場合
                    window.addEventListener('deviceorientationabsolute', handleOrientation);
                    window.addEventListener('deviceorientation', handleOrientation);
                }

            } else {
                btn.classList.remove('active');
                btn.innerHTML = '<i class="fa-solid fa-cube"></i> 3Dナビモード開始';
                
                // 平面マップに戻す
                map.flyTo({ pitch: 0, bearing: 0, zoom: 15, duration: 1500 });
                
                // センサー連動を解除
                window.removeEventListener('deviceorientation', handleOrientation);
                window.removeEventListener('deviceorientationabsolute', handleOrientation);
            }
        }

        // --- アカウント情報モーダル制御 ---
        function openProfile() { document.getElementById('profile-modal').style.display = 'flex'; }
        function closeProfile() { document.getElementById('profile-modal').style.display = 'none'; }
    </script>
</body>
</html>
