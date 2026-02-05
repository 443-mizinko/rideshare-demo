# rideshare-demo
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>相乗りタクシー マッチング（完成・統合版）</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <style>
    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, sans-serif;
      background: #f2f2f2;
    }

    .app {
      max-width: 390px;
      margin: 0 auto;
      background: #ffffff;
      min-height: 100vh;
    }

    .header {
      background: #000;
      color: #fff;
      padding: 16px;
      text-align: center;
    }

    .section {
      padding: 12px;
    }

    .box {
      background: #fafafa;
      padding: 12px;
      border-radius: 10px;
      margin-bottom: 12px;
    }

    label {
      display: block;
      font-size: 14px;
      margin-bottom: 6px;
    }

    input[type="number"] {
      width: 100%;
      padding: 6px;
      font-size: 14px;
      margin-top: 4px;
    }

    .name {
      font-weight: bold;
    }

    .price {
      font-size: 22px;
      font-weight: bold;
    }

    .myprice {
      font-size: 26px;
      font-weight: bold;
      color: #d32f2f;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      font-size: 13px;
    }

    th, td {
      border-bottom: 1px solid #ddd;
      padding: 6px;
      text-align: center;
    }

    .highlight {
      background: #e0e0e0;
    }

    .note {
      font-size: 12px;
      color: #666;
      text-align: center;
      margin-top: 8px;
    }
  </style>
</head>

<body>
<div class="app">

  <div class="header">
    <h1>福タク</h1>
  </div>

  <div class="section">

    <!-- 入力 -->
    <div class="box">
      <label>
        あなたのグループ人数
        <input type="number" id="myGroup" value="1" min="1">
      </label>

      <label>
        目的地までの距離（m）
        <input type="number" id="myDistance" value="300">
      </label>
    </div>

    <!-- 相手選択 -->
    <div class="box">
      <p class="name">相乗り相手を選択</p>

      <label><input type="radio" name="partner" value="A" checked> Aさん（2人・カップル・20代・300m）</label>
      <label><input type="radio" name="partner" value="B"> Bさん（1人・男・40代・250m）</label>
      <label><input type="radio" name="partner" value="C"> Cさん（4人・男２女２・10代・600m）</label>
    </div>

    <!-- 結果 -->
    <div class="box" id="result"></div>

    <p class="note">
      ※1.1kmまで620円、以降252mごとに100円（切り上げ）<br>
      ※検証用デモ（仮データ）
    </p>

  </div>
</div>

<script>
  // ===== 料金計算 =====
  function calcFare(distance) {
    const BASE_DISTANCE = 1100;
    const BASE_FARE = 620;
    const STEP_DISTANCE = 252;
    const STEP_FARE = 100;

    if (distance <= BASE_DISTANCE) {
      return BASE_FARE;
    }
    return BASE_FARE + Math.ceil((distance - BASE_DISTANCE) / STEP_DISTANCE) * STEP_FARE;
  }

  // 仮データ
  const partners = {
    A: { name: "A", group: 2, distance: 300 },
    B: { name: "B", group: 1, distance: 250 },
    C: { name: "C", group: 4, distance: 600 }
  };

  // 入力・選択変更で即再計算
  document.addEventListener("input", recalc);
  document.addEventListener("change", recalc);

  function recalc() {
    const myGroup = Number(document.getElementById("myGroup").value);
    const myDistance = Number(document.getElementById("myDistance").value);
    const selected = document.querySelector('input[name="partner"]:checked').value;
    const result = document.getElementById("result");

    if (myGroup <= 0 || myDistance <= 0) {
      result.textContent = "入力値が不正です。";
      return;
    }

    let rows = "";
    let selectedBlock = "";

    Object.values(partners).forEach(p => {
      const rideDistance = Math.max(myDistance, p.distance);
      const totalFare = calcFare(rideDistance);
      const myFare = Math.round(totalFare / (myGroup + p.group));
      const isSelected = p.name === selected;

      rows += `
        <tr class="${isSelected ? "highlight" : ""}">
          <td>${p.name}さん</td>
          <td>${p.group}</td>
          <td>${rideDistance}m</td>
          <td>${totalFare}円</td>
          <td>${myFare}円</td>
        </tr>
      `;

      if (isSelected) {
        selectedBlock = `
          <p><strong>選択中：${p.name}さん</strong></p>
          <p class="price">総額：${totalFare}円</p>
          <p class="myprice">あなたの負担額：${myFare}円</p>
        `;
      }
    });

    result.innerHTML = `
      ${selectedBlock}
      <hr>
      <table>
        <tr>
          <th>相手</th>
          <th>人数</th>
          <th>距離</th>
          <th>総額</th>
          <th>負担</th>
        </tr>
        ${rows}
      </table>
    `;
  }

  // 初期表示
  recalc();
</script>

</body>
</html>
