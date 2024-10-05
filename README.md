<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>طلب توصيل - الزبون</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background-color: #f0f0f0;
            margin: 0;
            padding: 0;
        }
        h1 {
            color: #333;
            margin-top: 20px;
        }
        #requestRide {
            padding: 15px 30px;
            font-size: 16px;
            background-color: #28a745;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        #requestRide:hover {
            background-color: #218838;
        }
        #output {
            margin-top: 15px;
            color: #555;
        }
        #map {
            height: 400px;
            width: 100%;
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <h1>طلب توصيل</h1>
    <button id="requestRide">طلب أقرب كابتن</button>
    <p id="output">في انتظار طلب...</p>
    <div id="map"></div>

    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-database.js"></script>

    <script>
        // إعداد Firebase
        const firebaseConfig = {
            apiKey: "AIzaSyCow0kJbcJ7OUzaHOBqXp1hF1LPSxb6dBo",
            authDomain: "alnashme-tawsel.firebaseapp.com",
            databaseURL: "https://alnashme-tawsel-default-rtdb.firebaseio.com/",
            projectId: "alnashme-tawsel",
            storageBucket: "gs://alnashme-tawsel.appspot.com",
            messagingSenderId: "76863416572",
            appId: "1:76863416572:android:e75569b3ef02142baf8f79"
        };

        // Initialize Firebase
        const app = firebase.initializeApp(firebaseConfig);
        const database = firebase.database();

        // مفتاح OpenCage API
        const openCageApiKey = "afa6985b839946d6ab60715d07e2af2c";
        let map, customerLatLng;

        // طلب الإذن من المستخدم لتحديد الموقع
        function requestLocationPermission() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(showPosition, showError);
            } else {
                document.getElementById('output').innerText = "المتصفح الخاص بك لا يدعم تحديد الموقع.";
            }
        }

        // عرض الموقع الحالي على الخريطة
        function showPosition(position) {
            const latitude = position.coords.latitude;
            const longitude = position.coords.longitude;
            customerLatLng = { lat: latitude, lng: longitude };

            // إعداد الخريطة وعرض الموقع الحالي
            map = new google.maps.Map(document.getElementById('map'), {
                center: customerLatLng,
                zoom: 14
            });
            new google.maps.Marker({
                position: customerLatLng,
                map: map,
                title: "موقعك الحالي"
            });

            // تحويل الإحداثيات إلى عنوان
            getAddressFromCoordinates(latitude, longitude).then((address) => {
                document.getElementById('output').innerText = 'تم تحديد الموقع: ' + address;

                // إرسال الطلب إلى Firebase
                firebase.database().ref('rideRequests').push({
                    latitude: latitude,
                    longitude: longitude,
                    address: address,
                    status: 'pending'
                });
            });
        }

        // دالة لتحويل الإحداثيات إلى عنوان باستخدام OpenCage API
        function getAddressFromCoordinates(latitude, longitude) {
            const url = `https://api.opencagedata.com/geocode/v1/json?q=${latitude},${longitude}&key=${openCageApiKey}`;
            return fetch(url)
                .then(response => response.json())
                .then(data => {
                    return data.results[0].formatted;
                })
                .catch(error => {
                    console.error('Error fetching address:', error);
                });
        }

        // التعامل مع أخطاء تحديد الموقع
        function showError(error) {
            switch(error.code) {
                case error.PERMISSION_DENIED:
                    document.getElementById('output').innerText = "تم رفض طلب الوصول إلى الموقع.";
                    break;
                case error.POSITION_UNAVAILABLE:
                    document.getElementById('output').innerText = "الموقع غير متوفر.";
                    break;
                case error.TIMEOUT:
                    document.getElementById('output').innerText = "انتهت مهلة الحصول على الموقع.";
                    break;
                case error.UNKNOWN_ERROR:
                    document.getElementById('output').innerText = "حدث خطأ غير معروف.";
                    break;
            }
        }

        // عند الضغط على زر "طلب سيارة"
        document.getElementById('requestRide').addEventListener('click', function() {
            requestLocationPermission();
        });

        // تحميل خريطة Google Maps API
        function loadGoogleMapsScript() {
            const script = document.createElement('script');
            script.src = `https://maps.googleapis.com/maps/api/js?key=YOUR_GOOGLE_MAPS_API_KEY&callback=initMap`;
            script.async = true;
            script.defer = true;
            document.body.appendChild(script);
        }

        loadGoogleMapsScript();
    </script>
</body>
</html>
