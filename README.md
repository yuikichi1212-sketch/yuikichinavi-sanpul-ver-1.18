<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Ultimate Hybrid Navi</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<link rel="stylesheet" href="https://unpkg.com/leaflet-routing-machine@3.2.12/dist/leaflet-routing-machine.css" />
<script src="https://unpkg.com/leaflet-routing-machine@3.2.12/dist/leaflet-routing-machine.js"></script>

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

<style>
    :root {
        --primary-blue: #007AFF;
        --glass-bg: rgba(255, 255, 255, 0.85);
        --glass-border: rgba(255, 255, 255, 0.4);
        --blur: blur(20px);
        --shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
        --text-main: #1c1c1e;
        --text-sub: #8e8e93;
    }

    body, html {
        margin: 0; padding: 0; height: 100%; overflow: hidden;
        font-family: -apple-system, BlinkMacSystemFont, "Roboto", sans-serif;
    }

    /* 地図コンテナ */
    #map {
        position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 1;
    }

    /* UIレイヤー共通設定 */
    .ui-layer {
        position: absolute; z-index: 1000;
    }

    /* 1. Google風検索バー & カテゴリチップ */
    .top-bar {
        top: 0; left: 0; width: 100%;
        padding: 15px 0;
        background: linear-gradient(to bottom, rgba(255,255,255,0.9) 0%, rgba(255,255,255,0) 100%);
        pointer-events: none; /* 下の地図を触れるように */
    }

    .search-box {
        pointer-events: auto;
        width: 90%; max-width: 500px; margin: 0 auto;
        background: #fff;
        border-radius: 30px;
        box-shadow: 0 4px 15px rgba(0,0,0,0.15);
        display: flex; align-items: center; padding: 12px 20px;
        transition: all 0.3s ease;
    }
    
    .search-box:focus-within {
        box-shadow: 0 6px 20px rgba(0,0,0,0.2);
    }

    .search-input {
        border: none; outline: none; flex: 1; font-size: 16px; margin-left: 10px;
    }

    .category-chips {
        pointer-events: auto;
        display: flex; gap: 10px; overflow-x: auto;
        padding: 15px 20px; scrollbar-width: none;
        -webkit-overflow-scrolling: touch;
    }
    .category-chips::-webkit-scrollbar { display: none; }

    .chip {
        background: #fff; padding: 8px 16px; border-radius: 20px;
        box-shadow: 0 2px 8px rgba(0,0,0,0.1); font-size: 14px; font-weight: 500;
        white-space: nowrap; cursor: pointer; display: flex; align-items: center; gap: 6px;
        transition: transform 0.1s;
    }
    .chip:active { transform: scale(0.95); }
    .chip i { color: var(--primary-blue); }

    /* 2. Apple風フローティングツール & 現在地 */
    .map-controls {
        right: 15px; top: 180px;
        display: flex; flex-direction: column; gap: 12px;
    }

    .control-btn {
        width: 48px; height: 48px;
        background: var(--glass-bg); backdrop-filter: var(--blur);
        border: 1px solid var(--glass-border);
        border-radius: 12px;
        display: flex; justify-content: center; align-items: center;
        box-shadow: var(--shadow); cursor: pointer;
        font-size: 20px; color: var(--primary-blue);
        transition: 0.2s;
    }
    .control-btn:active { background: #fff; }

    /* 3. Apple風ボトムシート (ナビゲーション情報) */
    .bottom-sheet {
        bottom: 20px; left: 50%; transform: translateX(-50%);
        width: 94%; max-width: 500px;
        background: var(--glass-bg);
        backdrop-filter: var(--blur);
        -webkit-backdrop-filter: var(--blur);
        border-radius: 24px;
        padding: 24px;
        box-shadow: 0 -10px 40px rgba(0,0,0,0.1);
        border: 1px solid var(--glass-border);
        transition: transform 0.3s ease;
    }

    .sheet-handle {
        width: 40px; height: 5px; background: #ccc; border-radius: 5px;
        margin: 0 auto 20px auto;
    }

    .route-details {
        display: none; /* ルート検索時に表示 */
    }
    
    .place-info {
        /* デフォルト表示 */
    }

    .place-title { font-size: 22px; font-weight: bold; margin-bottom: 5px; color: var(--text-main); }
    .place-subtitle { font-size: 14px; color: var(--text-sub); margin-bottom: 20px; }

    .nav-stats {
        display: flex; gap: 20px; align-items: baseline; margin-bottom: 20px;
    }
    .nav-time { font-size: 32px; font-weight: 800; color: #34C759; } /* Apple Green */
    .nav-dist { font-size: 16px; color: var(--text-sub); font-weight: 600; }

    .action-btn {
        width: 100%; padding: 16px;
        background: var(--primary-blue); color: white;
        border: none; border-radius: 16px;
        font-size: 18px; font-weight: bold;
        cursor: pointer; box-shadow: 0 4px 15px rgba(0,122,255,0.3);
        transition: transform 0.1s;
    }
    .action-btn:active { transform: scale(0.98); }

    /* ルート案内パネルのカスタマイズ（ライブラリの見た目を上書き） */
    .leaflet-routing-container {
        display: none !important; /* デフォルトのパネルは隠して自作UIを使う */
    }
</style>
</head>
<body>

    <div id="map"></div>

    <div class="ui-layer top-bar">
        <div class="search-box">
            <i class="fa-solid fa-bars" style="color:#666; margin-right:10px;"></i>
            <input type="text" class="search-input" placeholder="場所を検索...">
            <i class="fa-solid fa-microphone" style="color:var(--primary-blue);"></i>
            <div style="width:30px; height:30px; background:#ddd; border-radius:50%; margin-left:15px; overflow:hidden;">
                <img src="https://api.dicebear.com/7.x/avataaars/svg?seed=Felix" alt="User" width="100%">
            </div>
        </div>
        <div class="category-chips">
            <div class="chip" onclick="searchNear('restaurant')"><i class="fa-solid fa-utensils"></i> レストラン</div>
            <div class="chip" onclick="searchNear('gas_station')"><i class="fa-solid fa-gas-pump"></i> ガソリン</div>
            <div class="chip" onclick="searchNear('cafe')"><i class="fa-solid fa-coffee"></i> カフェ</div>
            <div class="chip" onclick="searchNear('convenience_store')"><i class="fa-solid fa-store"></i> コンビニ</div>
            <div class="chip" onclick="searchNear('parking')"><i class="fa-solid fa-square-parking"></i> 駐車場</div>
        </div>
    </div>

    <div class="ui-layer map-controls">
        <div class="control-btn" onclick="map.setZoom(map.getZoom() + 1)"><i class="fa-solid fa-plus"></i></div>
        <div class="control-btn" onclick="map.setZoom(map.getZoom() - 1)"><i class="fa-solid fa-minus"></i></div>
        <div class="control-btn" onclick="locateUser()"><i class="fa-solid fa-location-crosshairs"></i></div>
        <div class="control-btn" style="color: #333;"><i class="fa-solid fa-layer-group"></i></div>
    </div>

    <div class="ui-layer bottom-sheet" id="bottomSheet">
        <div class="sheet-handle"></div>
        
        <div id="defaultInfo" class="place-info">
            <div class="place-title">現在地周辺を探索</div>
            <div class="place-subtitle">地図をタップして目的地を設定してください</div>
            <div style="display:flex; gap:10px; overflow-x:auto; padding-bottom:10px;">
                <img src="https://source.unsplash.com/random/100x100?city" style="border-radius:10px;">
                <img src="https://source.unsplash.com/random/100x100?map" style="border-radius:10px;">
                <img src="https://source.unsplash.com/random/100x100?street" style="border-radius:10px;">
            </div>
        </div>

        <div id="routeInfo" class="route-details">
            <div class="place-title">ドロップされた地点</div>
            <div class="place-subtitle">経由地なし • 通常の交通状況</div>
            <div class="nav-stats">
                <div class="nav-time" id="timeDisplay">--分</div>
                <div class="nav-dist" id="distDisplay">-- km</div>
            </div>
            <button class="action-btn" onclick="startNavigation()">出発</button>
        </div>
    </div>

<script>
    // 1. 地図の初期化 (デフォルトは東京駅周辺)
    const map = L.map('map', { zoomControl: false }).setView([35.681236, 139.767125], 15);

    // 2. 地図タイル読み込み (CartoDB Voyager - Appleマップに近いクリーンなデザイン)
    L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png', {
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OSM</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
        subdomains: 'abcd',
        maxZoom: 20
    }).addTo(map);

    // 変数
    let userMarker = null;
    let destinationMarker = null;
    let routingControl = null;

    // 3. 現在地取得機能
    function locateUser() {
        if (!navigator.geolocation) {
            alert("このブラウザはGPSに対応していません。");
            return;
        }
        
        navigator.geolocation.getCurrentPosition(
            (position) => {
                const lat = position.coords.latitude;
                const lng = position.coords.longitude;
                const userLoc = [lat, lng];

                // 現在地マーカー (青い丸)
                if (userMarker) {
                    userMarker.setLatLng(userLoc);
                } else {
                    const icon = L.divIcon({
                        className: 'user-marker',
                        html: '<div style="width:20px;height:20px;background:#007AFF;border:3px solid white;border-radius:50%;box-shadow:0 3px 10px rgba(0,0,0,0.3);"></div>',
                        iconSize: [26, 26],
                        iconAnchor: [13, 13]
                    });
                    userMarker = L.marker(userLoc, {icon: icon}).addTo(map);
                }

                map.flyTo(userLoc, 16, { animate: true, duration: 1.5 });
            },
            () => {
                alert("位置情報の取得に失敗しました。");
            }
        );
    }

    // 初回ロード時に現在地取得
    locateUser();

    // 4. マップクリックでルート検索
    map.on('click', function(e) {
        if (!userMarker) {
            alert("まず右側の現在地ボタンを押して位置を取得してください！");
            return;
        }

        const userLatLng = userMarker.getLatLng();
        const destLatLng = e.latlng;

        // 既存のルートとマーカーを削除
        if (routingControl) {
            map.removeControl(routingControl);
            routingControl = null;
        }
        if (destinationMarker) {
            map.removeLayer(destinationMarker);
        }

        // 目的地のマーカー
        destinationMarker = L.marker(destLatLng).addTo(map)
            .bindPopup("目的地")
            .openPopup();

        // UI切り替え
        document.getElementById('defaultInfo').style.display = 'none';
        document.getElementById('routeInfo').style.display = 'block';
        document.getElementById('timeDisplay').innerText = "計算中...";

        // ルート計算 (OSRM Demo Server使用)
        routingControl = L.Routing.control({
            waypoints: [
                L.latLng(userLatLng.lat, userLatLng.lng),
                L.latLng(destLatLng.lat, destLatLng.lng)
            ],
            router: L.Routing.osrmv1({
                serviceUrl: 'https://router.project-osrm.org/route/v1' // 無料デモ用API
            }),
            lineOptions: {
                styles: [{color: '#007AFF', opacity: 0.7, weight: 6}] // Apple風の青いライン
            },
            createMarker: function() { return null; }, // デフォルトマーカーを非表示
            addWaypoints: false,
            draggableWaypoints: false,
            fitSelectedRoutes: true
        }).on('routesfound', function(e) {
            const routes = e.routes;
            const summary = routes[0].summary;
            
            // 時間と距離を更新
            const timeMin = Math.round(summary.totalTime / 60);
            const distKm = (summary.totalDistance / 1000).toFixed(1);
            
            document.getElementById('timeDisplay').innerText = timeMin + "分";
            document.getElementById('distDisplay').innerText = distKm + " km";
            
        }).addTo(map);
    });

    // 5. ナビ開始ボタン（シミュレーション）
    function startNavigation() {
        alert("ナビゲーションを開始します！\n(実際の音声案内はAPIキー制限のためこのデモでは省略されています)");
        // ここでカメラを斜めにして追従モードにする処理などが本来入ります
    }

    // 6. カテゴリ検索（ダミー）
    function searchNear(type) {
        alert(type + " を検索中...\n(Google Places APIキーがあればここにピンが降ります)");
    }

</script>
</body>
</html>
