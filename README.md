# notion_timezoneconverter
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <title>Timezone Converter Widget</title>
  <script src="https://cdn.jsdelivr.net/npm/luxon@3/build/global/luxon.min.js"></script>
  <style>
    :root {
      --bg: #ffffff;
      --text: #111;
      --sub: #666;
      --border: #ddd;
      --accent: #2563eb;
    }

    @media (prefers-color-scheme: dark) {
      :root {
        --bg: #111;
        --text: #f5f5f5;
        --sub: #aaa;
        --border: #333;
      }
    }

    body {
      font-family: system-ui, -apple-system, sans-serif;
      background: var(--bg);
      color: var(--text);
      margin: 0;
      padding: 16px;
    }

    input, select, button {
      width: 100%;
      padding: 10px;
      margin-top: 8px;
      border-radius: 8px;
      border: 1px solid var(--border);
      background: var(--bg);
      color: var(--text);
      font-size: 14px;
    }

    button {
      background: var(--accent);
      color: white;
      border: none;
      cursor: pointer;
    }

    button.secondary {
      background: transparent;
      color: var(--sub);
      border: 1px dashed var(--border);
    }

    .result {
      margin-top: 14px;
      padding: 12px;
      border-radius: 10px;
      border: 1px solid var(--border);
      font-size: 14px;
      line-height: 1.6;
    }

    .small {
      color: var(--sub);
      font-size: 12px;
    }

    .row {
      display: flex;
      gap: 8px;
    }

    .row button {
      flex: 1;
    }
  </style>
</head>
<body>

  <input id="input" placeholder="예: 2025년 12월 23일 20:00 EST" />

  <select id="target">
    <option value="Asia/Seoul">KST (한국)</option>
    <option value="UTC">UTC</option>
    <option value="America/New_York">EST / EDT</option>
    <option value="Europe/London">GMT / BST</option>
    <option value="America/Los_Angeles">PST / PDT</option>
  </select>

  <div class="row">
    <button onclick="convert()">변환</button>
    <button class="secondary" onclick="swap()">스왑</button>
  </div>

  <div id="output" class="result"></div>

<script>
const { DateTime } = luxon;

const TZ_MAP = {
  UTC: "UTC",
  GMT: "UTC",
  KST: "Asia/Seoul",
  EST: "America/New_York",
  EDT: "America/New_York",
  CST: "America/Chicago",
  CDT: "America/Chicago",
  PST: "America/Los_Angeles",
  PDT: "America/Los_Angeles"
};

const regex =
/(\d{4})[년\-\/\.]\s*(\d{1,2})[월\-\/\.]\s*(\d{1,2})[일]?\s*(\d{1,2}):(\d{2})\s*(AM|PM)?\s*([A-Za-z\/_]+)$/i;

function convert() {
  const input = document.getElementById("input").value.trim();
  const targetZone = document.getElementById("target").value;
  const match = input.match(regex);

  if (!match) {
    output("❌ 입력 형식을 인식할 수 없습니다.");
    return;
  }

  let [_, y, m, d, h, min, ampm, tz] = match;
  h = parseInt(h);

  if (ampm) {
    ampm = ampm.toUpperCase();
    if (ampm === "PM" && h < 12) h += 12;
    if (ampm === "AM" && h === 12) h = 0;
  }

  const sourceZone = TZ_MAP[tz.toUpperCase()] || tz;

  const dt = DateTime.fromObject({
    year: +y,
    month: +m,
    day: +d,
    hour: +h,
    minute: +min
  }, { zone: sourceZone });

  if (!dt.isValid) {
    output("❌ 시간대가 올바르지 않습니다.");
    return;
  }

  const target = dt.setZone(targetZone);
  const kst = dt.setZone("Asia/Seoul");
  const utc = dt.setZone("UTC");

  const relative = kst.toRelativeCalendar({ locale: "ko" });

  const dstNote = dt.isInDST ? "🌞 서머타임 적용 중" : "";

  output(`
<b>🎯 선택한 시간대</b><br>
${target.toFormat("yyyy-LL-dd HH:mm ZZZZ")}<br><br>

<b>🇰🇷 한국 (KST)</b><br>
${kst.toFormat("yyyy-LL-dd HH:mm")}<br>

<b>🌍 UTC</b><br>
${utc.toFormat("yyyy-LL-dd HH:mm")}<br><br>

<span class="small">🕰 ${relative || ""} ${dstNote}</span><br><br>

<button onclick="copyText('${kst.toISO()}')">📋 KST ISO 복사</button>
  `);
}

function output(html) {
  document.getElementById("output").innerHTML = html;
}

function copyText(text) {
  navigator.clipboard.writeText(text);
  alert("복사됨");
}

function swap() {
  const out = document.getElementById("output").innerText;
  if (out) document.getElementById("input").value = out.split("\n")[0];
}
</script>

</body>
</html>
