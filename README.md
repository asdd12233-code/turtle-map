# turtle-map
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>늑대거북이 지도</title>
  <script src="https://dapi.kakao.com/v2/maps/sdk.js?appkey=5653bbc7096bc2e00e8cb33b638f53f6&autoload=false"></script>
  <style>
    *{box-sizing:border-box;margin:0;padding:0}
    html,body{width:100%;height:100%;background:#1a1a1a}
    .map-card{background:#1a1a1a;width:100%;height:100vh;display:flex;flex-direction:column}
    .map-header{padding:14px 20px 10px;display:flex;align-items:center;justify-content:space-between;flex-shrink:0}
    .map-title{font-size:14px;font-weight:900;color:#fff;font-family:system-ui,sans-serif}
    .live-badge{display:flex;align-items:center;gap:5px;background:rgba(255,255,255,.08);border-radius:999px;padding:4px 10px;font-size:12px;font-weight:700;color:#00ff88;font-family:system-ui,sans-serif}
    .live-dot{width:7px;height:7px;border-radius:50%;background:#00ff88;animation:pulse 1.2s ease-in-out infinite}
    @keyframes pulse{0%,100%{opacity:1;transform:scale(1)}50%{opacity:.4;transform:scale(.7)}}
    #kakao-map{width:100%;flex:1}
    .progress-wrap{padding:12px 20px 16px;flex-shrink:0}
    .progress-meta{display:flex;justify-content:space-between;font-size:12px;font-weight:700;margin-bottom:8px;font-family:system-ui,sans-serif}
    .progress-start,.progress-end{color:#888}
    .progress-pct{color:#00ff88}
    .progress-bar-bg{background:#2a2a2a;border-radius:999px;height:6px;overflow:hidden}
    .progress-bar-fill{height:100%;background:linear-gradient(90deg,#00cc66,#00ff88);border-radius:999px;transition:width 1s linear;width:0%}
  </style>
</head>
<body>
  <div class="map-card">
    <div class="map-header">
      <div class="map-title">🐢 중곡역 → 합정역</div>
      <div class="live-badge"><div class="live-dot"></div>LIVE</div>
    </div>
    <div id="kakao-map"></div>
    <div class="progress-wrap">
      <div class="progress-meta">
        <span class="progress-start">중곡역</span>
        <span class="progress-pct" id="progress-pct">0.0%</span>
        <span class="progress-end">합정역</span>
      </div>
      <div class="progress-bar-bg">
        <div class="progress-bar-fill" id="progress-bar"></div>
      </div>
    </div>
  </div>

  <script>
  kakao.maps.load(function() {
    var START_KST = new Date('2026-04-20T02:30:00+09:00').getTime();
    var END_KST   = new Date('2026-05-02T00:00:00+09:00').getTime();

    var ROUTE = [
      [37.5637,127.0847],[37.5628,127.0820],[37.5615,127.0775],
      [37.5600,127.0730],[37.5590,127.0690],[37.5578,127.0650],
      [37.5560,127.0600],[37.5540,127.0555],[37.5515,127.0510],
      [37.5490,127.0460],[37.5465,127.0410],[37.5440,127.0360],
      [37.5415,127.0300],[37.5400,127.0240],[37.5390,127.0180],
      [37.5445,127.0117],[37.5435,127.0060],[37.5425,127.0000],
      [37.5420,126.9940],[37.5415,126.9880],[37.5408,126.9830],
      [37.5398,126.9780],[37.5385,126.9730],[37.5365,126.9680],
      [37.5345,126.9650],[37.5320,126.9640],[37.5530,126.9050],
      [37.5520,126.9080],[37.5510,126.9110],[37.5500,126.9140]
    ];

    function getProgress() {
      var now = Date.now();
      if (now <= START_KST) return 0;
      if (now >= END_KST) return 1;
      return (now - START_KST) / (END_KST - START_KST);
    }

    function getPos(p) {
      var total = ROUTE.length-1, fi = p*total, idx = Math.floor(fi), frac = fi-idx;
      if (idx >= total) return ROUTE[total];
      var a = ROUTE[idx], b = ROUTE[idx+1];
      return [a[0]+(b[0]-a[0])*frac, a[1]+(b[1]-a[1])*frac];
    }

    var initP = getProgress(), initPos = getPos(initP);
    var map = new kakao.maps.Map(document.getElementById('kakao-map'), {
      center: new kakao.maps.LatLng(initPos[0], initPos[1]), level: 7
    });

    setTimeout(function() {
      map.relayout();
      map.setCenter(new kakao.maps.LatLng(initPos[0], initPos[1]));
    }, 300);

    map.addControl(new kakao.maps.ZoomControl(), kakao.maps.ControlPosition.RIGHT);

    new kakao.maps.Polyline({
      map:map,
      path: ROUTE.map(function(p){ return new kakao.maps.LatLng(p[0],p[1]); }),
      strokeWeight:5, strokeColor:'#00CC66', strokeOpacity:0.9, strokeStyle:'solid'
    });

    new kakao.maps.Marker({map:map, position:new kakao.maps.LatLng(ROUTE[0][0],ROUTE[0][1]), title:'중곡역'});
    new kakao.maps.Marker({map:map, position:new kakao.maps.LatLng(ROUTE[29][0],ROUTE[29][1]), title:'합정역'});

    var turtleImg = new kakao.maps.MarkerImage(
      'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NCIgaGVpZ2h0PSI0NCIgdmlld0JveD0iMCAwIDQ0IDQ0Ij48Y2lyY2xlIGN4PSIyMiIgY3k9IjIyIiByPSIyMCIgZmlsbD0iIzAwY2M2NiIgb3BhY2l0eT0iMC4zIi8+PHRleHQgeD0iMjIiIHk9IjMwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBmb250LXNpemU9IjI0Ij7wn5CCzIPC90ZXh0Pjwvc3ZnPg==',
      new kakao.maps.Size(44,44), {offset: new kakao.maps.Point(22,22)}
    );

    var marker = new kakao.maps.Marker({
      map:map, position:new kakao.maps.LatLng(initPos[0],initPos[1]),
      image:turtleImg, title:'늑대거북이 현재 위치', zIndex:10
    });

    function update() {
      var prog = getProgress(), pos = getPos(prog);
      marker.setPosition(new kakao.maps.LatLng(pos[0],pos[1]));
      var pct = (prog*100).toFixed(1);
      document.getElementById('progress-pct').textContent = pct+'%';
      document.getElementById('progress-bar').style.width = pct+'%';
    }

    update();
    setInterval(update, 1000);
  });
  </script>
</body>
</html>
