<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>本格ナビアプリ</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f0f0f0; margin: 0; padding: 20px; }
        #map { height: 500px; width: 100%; margin-top: 20px; }
        h1 { text-align: center; color: #333; }
        .container { width: 80%; margin: auto; }
        form { display: flex; justify-content: space-between; margin-bottom: 20px; }
        input[type="text"] { width: 70%; padding: 10px; border: 1px solid #ccc; border-radius: 5px; }
        input[type="submit"] { padding: 10px 20px; border: none; border-radius: 5px; background-color: #28a745; color: white; cursor: pointer; }
        input[type="submit"]:hover { background-color: #218838; }
    </style>
</head>
<body>
    <h1>本格ナビアプリ✨</h1>
    <div class="container">
        <form id="locationForm">
            <input type="text" id="destination" placeholder="行き先を入力してください" required>
            <input type="submit" value="ナビする">
        </form>
        <div id="map"></div>
    </div>
    
    <script>
        let map;
        let currentLocationMarker;

        function initMap() {
            const initialLocation = { lat: 35.681236, lng: 139.767125 }; // 東京駅
            map = new google.maps.Map(document.getElementById("map"), {
                zoom: 15,
                center: initialLocation,
            });
            currentLocationMarker = new google.maps.Marker({
                position: initialLocation,
                map: map,
            });
            
            // 現在地を取得するボタン
            getCurrentLocation();
        }

        function getCurrentLocation() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition((position) => {
                    const pos = {
                        lat: position.coords.latitude,
                        lng: position.coords.longitude,
                    };
                    currentLocationMarker.setPosition(pos);
                    map.setCenter(pos);
                }, () => {
                    handleLocationError(true, map.getCenter());
                });
            } else {
                handleLocationError(false, map.getCenter());
            }
        }

        function handleLocationError(browserHasGeolocation, pos) {
            alert(browserHasGeolocation 
                ? "エラー: アクセスが拒否されました。" 
                : "エラー: ジオロケーションがサポートされていません。");
            map.setCenter(pos);
        }

        document.getElementById("locationForm").onsubmit = function(event) {
            event.preventDefault();
            const destination = document.getElementById("destination").value;
            const geocoder = new google.maps.Geocoder();
            geocoder.geocode({ 'address': destination }, function(results, status) {
                if (status === 'OK') {
                    map.setCenter(results[0].geometry.location);
                    new google.maps.Marker({
                        map: map,
                        position: results[0].geometry.location,
                    });
                } else {
                    alert('地図の場所を見つけられませんでした: ' + status);
                }
            });
        };
    </script>
    <script async defer src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&callback=initMap"></script>
</body>
</html>
