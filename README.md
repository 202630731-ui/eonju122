<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>오늘 뭐먹지? - 메뉴 추천</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    }
    body {
      background-color: #f5f6f8;
      color: #333;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      padding: 20px;
    }
    .card {
      background: white;
      width: 100%;
      max-width: 480px;
      padding: 28px 24px;
      border-radius: 20px;
      box-shadow: 0 10px 25px rgba(0,0,0,0.06);
      text-align: center;
    }
    h1 {
      font-size: 1.6rem;
      color: #ff5252;
      margin-bottom: 6px;
    }
    .sub {
      color: #777;
      font-size: 0.9rem;
      margin-bottom: 24px;
    }
    .step-title {
      font-size: 1.05rem;
      font-weight: 600;
      margin-bottom: 16px;
      color: #444;
    }
    .grid {
      display: grid;
      gap: 10px;
    }
    .grid-2 { grid-template-columns: repeat(2, 1fr); }
    .grid-3 { grid-template-columns: repeat(3, 1fr); }
    
    button.btn {
      padding: 14px 8px;
      border: 1.5px solid #eaeaea;
      background: #fafafa;
      border-radius: 12px;
      font-size: 0.95rem;
      font-weight: 600;
      cursor: pointer;
      color: #444;
      transition: all 0.2s;
    }
    button.btn:hover {
      border-color: #ff5252;
      color: #ff5252;
      background: #fff5f5;
    }
    .section {
      display: none;
    }
    .section.active {
      display: block;
      animation: fadeIn 0.3s ease;
    }
    .result-box {
      margin-top: 10px;
      padding: 20px;
      background: #fff8f8;
      border-radius: 16px;
      border: 1px solid #ffe3e3;
    }
    .food-img {
      width: 100%;
      height: 200px;
      object-fit: cover;
      border-radius: 12px;
      margin: 14px 0;
    }
    .food-title {
      font-size: 1.5rem;
      font-weight: bold;
      color: #d32f2f;
      margin-bottom: 4px;
    }
    .food-desc {
      font-size: 0.88rem;
      color: #666;
      margin-bottom: 16px;
    }
    .map-group {
      display: flex;
      gap: 8px;
      margin-top: 12px;
    }
    .map-btn {
      flex: 1;
      padding: 10px 0;
      border-radius: 8px;
      text-decoration: none;
      font-size: 0.85rem;
      font-weight: bold;
      color: white;
    }
    .map-btn.naver { background-color: #03cf5d; }
    .map-btn.kakao { background-color: #ffeb00; color: #3c1e1e; }
    .map-btn.google { background-color: #4285f4; }
    
    .reset-btn {
      margin-top: 18px;
      background: none;
      border: none;
      color: #888;
      text-decoration: underline;
      cursor: pointer;
      font-size: 0.85rem;
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(6px); }
      to { opacity: 1; transform: translateY(0); }
    }
  </style>
</head>
<body>

<div class="card">
  <h1>오늘 뭐먹지? 🍲</h1>
  <p class="sub">고민은 끝! 딱 맞는 메뉴를 찾아드립니다.</p>

  <!-- 1단계: 카테고리 -->
  <div id="step1" class="section active">
    <div class="step-title">1. 어떤 스타일을 원하시나요?</div>
    <div class="grid grid-2">
      <button class="btn" onclick="setCategory('중식')">중식 🥢</button>
      <button class="btn" onclick="setCategory('일식')">일식 🍣</button>
      <button class="btn" onclick="setCategory('양식')">양식 🍝</button>
      <button class="btn" onclick="setCategory('기타')">기타 (한식/분식) 🍲</button>
    </div>
  </div>

  <!-- 2단계: 종류 -->
  <div id="step2" class="section">
    <div class="step-title">2. 어떤 종류의 식사인가요?</div>
    <div class="grid grid-3">
      <button class="btn" onclick="setType('밥류')">밥류</button>
      <button class="btn" onclick="setType('국류')">국류</button>
      <button class="btn" onclick="setType('면류')">면류</button>
      <button class="btn" onclick="setType('육류')">육류</button>
      <button class="btn" onclick="setType('생선류')">생선류</button>
    </div>
  </div>

  <!-- 3단계: 결과 -->
  <div id="step3" class="section">
    <div class="result-box">
      <div style="font-size:0.85rem; color:#888;">오늘의 추천 메뉴</div>
      <div id="result-name" class="food-title">메뉴 이름</div>
      <div id="result-desc" class="food-desc">메뉴 설명</div>
      <img id="result-img" class="food-img" src="" alt="음식 사진">
      
      <div style="font-size: 0.8rem; font-weight: bold; color:#555; margin-top:8px;">📍 근처 맛집 검색하기</div>
      <div class="map-group">
        <a id="link-naver" class="map-btn naver" target="_blank">네이버 지도</a>
        <a id="link-kakao" class="map-btn kakao" target="_blank">카카오맵</a>
        <a id="link-google" class="map-btn google" target="_blank">구글 지도</a>
      </div>
    </div>
    <button class="reset-btn" onclick="reset()">🔄 다시 고르기</button>
  </div>
</div>

<script>
  // 중복 없는 20가지 메뉴 데이터
  const foods = {
    '중식': {
      '밥류': { name: '중화 볶음밥', desc: '고소한 불향이 살아있는 깔끔한 볶음밥', img: 'https://images.unsplash.com/photo-1603133872878-684f208fb84b?w=600' },
      '국류': { name: '마라탕', desc: '매콤하고 얼얼한 중독성 있는 국물요리', img: 'https://images.unsplash.com/photo-1569718212165-3a8278d5f624?w=600' },
      '면류': { name: '짜장면', desc: '달콤 짭조름한 국민 중화 면요리', img: 'https://images.unsplash.com/photo-1585032226651-759b368d7246?w=600' },
      '육류': { name: '탕수육', desc: '겉은 바삭하고 속은 촉촉한 고기 튀김', img: 'https://images.unsplash.com/photo-1541544741938-0af808871cc0?w=600' },
      '생선류': { name: '칠리새우', desc: '새콤달콤한 소스가 어우러진 통통한 새우 요리', img: 'https://images.unsplash.com/photo-1565557623262-b51c2513a641?w=600' }
    },
    '일식': {
      '밥류': { name: '가츠동', desc: '부드러운 달걀과 바삭한 돈가스가 얹어진 덮밥', img: 'https://images.unsplash.com/photo-1565299585323-38d6b0865b47?w=600' },
      '국류': { name: '돈코츠 라멘', desc: '진한 돼지사골 육수가 일품인 일본식 라멘', img: 'https://images.unsplash.com/photo-1569718212165-3a8278d5f624?w=600' },
      '면류': { name: '야키소바', desc: '특제 간장 소스에 볶아낸 볶음면', img: 'https://images.unsplash.com/photo-1617093727343-374698b1b08d?w=600' },
      '육류': { name: '돈카츠', desc: '두툼한 돼지고기를 바삭하게 튀겨낸 메뉴', img: 'https://images.unsplash.com/photo-1591814468924-caf88d1232e1?w=600' },
      '생선류': { name: '모둠 초밥', desc: '신선한 해산물과 밥의 정갈한 조화', img: 'https://images.unsplash.com/photo-1579871494447-9811cf80d66c?w=600' }
    },
    '양식': {
      '밥류': { name: '해산물 리소토', desc: '부드러운 크림과 해산물이 어우러진 리소토', img: 'https://images.unsplash.com/photo-1633964913295-ceb43826e7c9?w=600' },
      '국류': { name: '클램 차우더', desc: '조개 향이 가득한 따뜻하고 고소한 수프', img: 'https://images.unsplash.com/photo-1547592166-23ac45744acd?w=600' },
      '면류': { name: '토마토 파스타', desc: '상큼한 토마토 소스가 일품인 파스타', img: 'https://images.unsplash.com/photo-1551183053-bf91a1d81141?w=600' },
      '육류': { name: '비프 스테이크', desc: '육즙이 풍부하게 살아있는 소고기 구이', img: 'https://images.unsplash.com/photo-1544025162-d76694265947?w=600' },
      '생선류': { name: '연어 스테이크', desc: '겉은 바삭하고 속은 부드러운 연어 요리', img: 'https://images.unsplash.com/photo-1519708227418-c8fd9a32b7a2?w=600' }
    },
    '기타': {
      '밥류': { name: '비빔밥', desc: '각종 나물과 고추장을 쓱쓱 비벼먹는 한식', img: 'https://images.unsplash.com/photo-1553163147-622ab57be1c7?w=600' },
      '국류': { name: '김치찌개', desc: '칼칼하고 오랫동안 끓여낸 정통 한국의 맛', img: 'https://images.unsplash.com/photo-1583032015879-e5022cb87c3b?w=600' },
      '면류': { name: '떡볶이', desc: '매콤달콤함이 돋보이는 대표 분식 메뉴', img: 'https://images.unsplash.com/photo-1626082927389-6cd097cdc6ec?w=600' },
      '육류': { name: '삼겹살 구이', desc: '지글지글 노릇하게 구워낸 국민 돼지고기', img: 'https://images.unsplash.com/photo-1529692236671-f1f6cf9683ba?w=600' },
      '생선류': { name: '고등어 구이', desc: '짭조름하고 고소해 밥도둑인 생선 구이', img: 'https://images.unsplash.com/photo-1534604973900-c43ab4c2e0ab?w=600' }
    }
  };

  let category = '';
  let type = '';

  function setCategory(c) {
    category = c;
    document.getElementById('step1').classList.remove('active');
    document.getElementById('step2').classList.add('active');
  }

  function setType(t) {
    type = t;
    document.getElementById('step2').classList.remove('active');
    document.getElementById('step3').classList.add('active');
    
    renderResult();
  }

  function renderResult() {
    const item = foods[category][type];
    
    document.getElementById('result-name').innerText = item.name;
    document.getElementById('result-desc').innerText = item.desc;
    document.getElementById('result-img').src = item.img;

    const query = encodeURIComponent(item.name + " 맛집");
    document.getElementById('link-naver').href = `https://map.naver.com/v5/search/${query}`;
    document.getElementById('link-kakao').href = `https://map.kakao.com/link/search/${query}`;
    document.getElementById('link-google').href = `https://www.google.com/maps/search/${query}`;
  }

  function reset() {
    category = '';
    type = '';
    document.getElementById('step3').classList.remove('active');
    document.getElementById('step1').classList.add('active');
  }
</script>

</body>
</html>
