[index.html](https://github.com/user-attachments/files/28490313/index.html)
from pathlib import Path

html = r'''<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover"/>
  <title>競馬予想画像メーカー</title>
  <script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>
  <style>
    * {
      box-sizing: border-box;
      -webkit-text-size-adjust: 100%;
      text-size-adjust: 100%;
    }

    body {
      font-family: -apple-system, BlinkMacSystemFont, "Hiragino Sans", "Yu Gothic", "Segoe UI", sans-serif;
      margin: 0;
      background: #eeeeee;
      color: #222;
      padding: 10px;
    }

    .app {
      width: 100%;
      max-width: 480px;
      margin: 0 auto;
    }

    h1 {
      font-size: 20px;
      text-align: center;
      margin: 8px 0 12px;
    }

    .box {
      background: #fff;
      padding: 12px;
      border-radius: 14px;
      margin-bottom: 12px;
      box-shadow: 0 2px 8px rgba(0,0,0,.08);
    }

    label {
      display: block;
      font-size: 14px;
      font-weight: 800;
      margin-top: 10px;
    }

    input, textarea {
      width: 100%;
      padding: 12px;
      margin-top: 5px;
      border-radius: 10px;
      border: 1px solid #bbb;
      font-size: 16px;
      background: #fff;
    }

    textarea {
      min-height: 82px;
      resize: vertical;
      line-height: 1.45;
    }

    button {
      width: 100%;
      padding: 13px;
      margin-top: 11px;
      border: none;
      border-radius: 12px;
      font-size: 16px;
      font-weight: 900;
      color: white;
      background: #111;
      cursor: pointer;
    }

    .green { background: #087738; }
    .blue { background: #1557b0; }

    .hint {
      font-size: 12px;
      color: #666;
      line-height: 1.55;
      margin-top: 8px;
    }

    .card {
      width: 100%;
      aspect-ratio: 1 / 1;
      background:
        linear-gradient(rgba(0,0,0,.52), rgba(0,0,0,.68)),
        radial-gradient(circle at top, #3aa45c, #06371d);
      color: #fff;
      padding: 7% 7% 6%;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      overflow: hidden;
    }

    .top {
      text-align: center;
      border-top: 4px solid #fff;
      border-bottom: 4px solid #fff;
      padding: 5% 2%;
    }

    .date {
      font-size: 5vw;
      max-font-size: 28px;
      letter-spacing: .08em;
      font-weight: 700;
      line-height: 1.1;
    }

    .race {
      font-size: 8vw;
      font-weight: 1000;
      margin-top: 3%;
      line-height: 1.08;
      overflow-wrap: anywhere;
    }

    .predictions {
      margin-top: 8%;
    }

    .horse {
      display: flex;
      align-items: center;
      gap: 4%;
      margin: 5% 0;
      font-size: 6.2vw;
      font-weight: 950;
      line-height: 1.18;
      overflow-wrap: anywhere;
    }

    .mark {
      width: 15%;
      aspect-ratio: 1 / 1;
      min-width: 15%;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      background: #fff;
      color: #111;
      border-radius: 50%;
      font-size: 7vw;
      font-weight: 1000;
      line-height: 1;
    }

    .horse-name {
      min-width: 0;
      flex: 1;
    }

    .bets {
      margin-top: 7%;
      background: rgba(255,255,255,.15);
      border: 2px solid rgba(255,255,255,.8);
      padding: 5%;
      font-size: 5.5vw;
      font-weight: 900;
      line-height: 1.35;
      white-space: pre-wrap;
      overflow-wrap: anywhere;
      max-height: 28%;
      overflow: hidden;
    }

    .footer {
      text-align: center;
      font-size: 4.2vw;
      letter-spacing: .08em;
      font-weight: 700;
      border-top: 2px solid rgba(255,255,255,.75);
      padding-top: 4%;
    }

    @media (min-width: 480px) {
      .date { font-size: 24px; }
      .race { font-size: 38px; }
      .horse { font-size: 30px; }
      .mark { font-size: 34px; }
      .bets { font-size: 27px; }
      .footer { font-size: 20px; }
    }

    #generatedImage {
      width: 100%;
      display: none;
      border-radius: 10px;
      border: 1px solid #ddd;
    }

    .image-title {
      font-weight: 900;
      margin-bottom: 6px;
    }
  </style>
</head>
<body>
  <div class="app">
    <h1>競馬予想画像メーカー</h1>

    <div class="box">
      <label>日付</label>
      <input id="dateInput" type="date">

      <label>レース名</label>
      <input id="raceInput" type="text" placeholder="例：安田記念">

      <label>◎ 本命</label>
      <input id="honmeiInput" type="text" placeholder="例：7 ソウルラッシュ">

      <label>○ 対抗</label>
      <input id="taikouInput" type="text" placeholder="例：13 ジャンタルマンタル">

      <label>買い目</label>
      <textarea id="betsInput" placeholder="例：ワイド 7-13&#10;馬連 7-13"></textarea>

      <button onclick="updateCard()">プレビュー更新</button>
      <button class="green" onclick="makeImage()">SNS用画像を作る</button>
      <button class="blue" onclick="downloadImage()">PNGを保存</button>

      <div class="hint">
        iPhoneでは「SNS用画像を作る」を押した後、下の完成画像を長押し保存してください。
      </div>
    </div>

    <div class="box">
      <div class="hint" style="margin-bottom:8px;">プレビュー</div>
      <div id="card" class="card">
        <div>
          <div class="top">
            <div id="dateText" class="date">2026.06.07</div>
            <div id="raceText" class="race">安田記念</div>
          </div>

          <div class="predictions">
            <div class="horse"><span class="mark">◎</span><span id="honmeiText" class="horse-name">7 ソウルラッシュ</span></div>
            <div class="horse"><span class="mark">○</span><span id="taikouText" class="horse-name">13 ジャンタルマンタル</span></div>
          </div>

          <div id="betsText" class="bets">買い目
ワイド 7-13</div>
        </div>

        <div class="footer">#競馬予想</div>
      </div>
    </div>

    <div class="box">
      <div class="image-title">完成画像</div>
      <div class="hint">表示された画像を長押し保存してください。</div>
      <img id="generatedImage" alt="生成された競馬予想画像">
    </div>
  </div>

  <script>
    let latestImageData = "";

    function formatDate(value) {
      if (!value) return "日付未入力";
      return value.replaceAll("-", ".");
    }

    function updateCard() {
      const date = document.getElementById("dateInput").value;
      const race = document.getElementById("raceInput").value.trim();
      const honmei = document.getElementById("honmeiInput").value.trim();
      const taikou = document.getElementById("taikouInput").value.trim();
      const bets = document.getElementById("betsInput").value.trim();

      document.getElementById("dateText").textContent = formatDate(date);
      document.getElementById("raceText").textContent = race || "レース名";
      document.getElementById("honmeiText").textContent = honmei || "本命馬";
      document.getElementById("taikouText").textContent = taikou || "対抗馬";
      document.getElementById("betsText").textContent = "買い目\n" + (bets || "未入力");
    }

    async function makeImage() {
      updateCard();

      const card = document.getElementById("card");
      const canvas = await html2canvas(card, {
        backgroundColor: null,
        scale: 3,
        useCORS: true
      });

      latestImageData = canvas.toDataURL("image/png");

      const img = document.getElementById("generatedImage");
      img.src = latestImageData;
      img.style.display = "block";
      img.scrollIntoView({ behavior: "smooth", block: "center" });
    }

    async function downloadImage() {
      if (!latestImageData) {
        await makeImage();
      }

      const link = document.createElement("a");
      link.download = "keiba-yosou.png";
      link.href = latestImageData;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    }

    window.addEventListener("load", updateCard);
  </script>
</body>
</html>
'''

path = Path("/mnt/data/index.html")
path.write_text(html, encoding="utf-8")
path.as_posix()
