<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>西進武嶺雙向戰術配瓦模擬器-精緻版</title>
    <style>
        body { font-family: 'PingFang TC', 'Microsoft JhengHei', sans-serif; background-color: #f0f3f5; color: #333; padding: 20px; }
        .container { max-width: 1050px; margin: 0 auto; background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 20px rgba(0,0,0,0.08); }
        h2 { text-align: center; color: #1a365d; margin-bottom: 5px; }
        .subtitle { text-align: center; color: #718096; margin-bottom: 25px; font-size: 14px; }
       
        .grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 15px; background: #edf2f7; padding: 20px; border-radius: 8px; margin-bottom: 15px; }
        .target-box { background: #e2e8f0; border: 2px solid #4a5568; }
        .input-group { margin-bottom: 5px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; color: #2d3748; font-size: 14px; }
        input, select { width: 100%; padding: 8px 10px; border: 1px solid #cbd5e0; border-radius: 6px; box-sizing: border-box; font-size: 15px; }
       
        .mode-notice { font-size: 13px; color: #2c5282; background: #ebf8ff; padding: 10px; border-radius: 6px; text-align: center; margin-bottom: 20px; border: 1px solid #bee3f8; }
       
        table { width: 100%; border-collapse: collapse; margin-top: 15px; background: white; table-layout: fixed; }
        th, td { border: 1px solid #e2e8f0; padding: 10px 8px; text-align: center; font-size: 13px; word-wrap: break-word; }
        th { background-color: #4a5568; color: white; font-size: 14px; }
        tr:nth-child(even) { background-color: #f7fafc; }
        .total-row { background-color: #cbd5e0 !important; font-weight: bold; color: #1a365d; font-size: 14px; }
       
        /* 寬度分配 */
        th:nth-child(1) { width: 18%; } /* 路段 */
        th:nth-child(2) { width: 14%; } /* 本段幾何 */
        th:nth-child(3) { width: 15%; } /* 累計幾何 */
        th:nth-child(4) { width: 10%; } /* 衰退 */
        th:nth-child(5) { width: 13%; } /* 配瓦 */
        th:nth-child(6) { width: 10%; } /* 推力比 */
        th:nth-child(7) { width: 11%; } /* 本段預估 */
        th:nth-child(8) { width: 11%; } /* 累計時間 */

        .watt-input { width: 70px; text-align: center; padding: 5px; border: 2px solid #3182ce; border-radius: 4px; font-weight: bold; color: #2b6cb0; font-size: 14px; }
        .watt-input:focus { border-color: #2b6cb0; background-color: #fffaf0; }
       
        /* 下方動態總計看板 */
        .summary-dashboard { margin-top: 30px; display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 20px; background: #2d3748; padding: 20px; border-radius: 8px; color: white; text-align: center; }
        .summary-box { border-right: 1px solid #4a5568; }
        .summary-box:last-child { border-right: none; }
        .summary-title { font-size: 14px; color: #cbd5e0; margin-bottom: 5px; }
        .summary-value { font-size: 26px; font-weight: bold; color: #63b3ed; }
        .summary-value.highlight { color: #f6e05e; }
       
        .accum-cell { color: #4a5568; font-weight: 500; } /* 移除底色，保持乾淨文字 */
        .accum-time-cell { color: #2b6cb0; font-weight: bold; background-color: #e6fffa; }
        .warn-text { font-size: 12px; color: #718096; margin-top: 15px; text-align: center; }
        .time-input-container { display: flex; gap: 5px; align-items: center; }
        .time-input-container input { text-align: center; }
    </style>
</head>
<body>

<div class="container">
    <h2>🏔️ 西進武嶺戰術配瓦模擬器</h2>
    <div class="subtitle">手動微調各區段瓦數，即時動態推算完賽時間與累積數據</div>
   
    <!-- 基礎設定與逆向目標設定區 -->
    <div class="grid">
        <div class="input-group">
            <label for="weight">騎士體重 (kg):</label>
            <input type="number" id="weight" value="89" oninput="updateAllCalculations('manual')">
        </div>
        <div class="input-group">
            <label for="bikeWeight">車重+裝備 (kg):</label>
            <input type="number" id="bikeWeight" value="10" oninput="updateAllCalculations('manual')">
        </div>
        <div class="input-group">
            <label for="ftp">騎士 FTP (W):</label>
            <input type="number" id="ftp" value="250" oninput="updateAllCalculations('manual')">
        </div>
        <div class="input-group target-box" style="padding: 5px 10px; border-radius: 6px;">
            <label for="target_hours" style="color: #1a365d;">🎯 目標完賽時間:</label>
            <div class="time-input-container">
                <input type="number" id="target_hours" value="4" min="2" max="10" oninput="updateAllCalculations('reverse')"> <span style="font-size:12px;color:#1a365d">時</span>
                <input type="number" id="target_minutes" value="30" min="0" max="59" oninput="updateAllCalculations('reverse')"> <span style="font-size:12px;color:#1a365d">分</span>
            </div>
        </div>
    </div>
   
    <div class="mode-notice">
        💡 <b>車錶實戰應用：</b> 將黃金配瓦下的<b>「累計里程」</b>與<b>「累計完成時間」</b>記在號碼牌或上管貼紙上。6/22 比賽當天抵達補給點時，對照碼錶時間即可知道戰術執行進度！
    </div>

    <!-- 互動表格 -->
    <table>
        <thead>
            <tr>
                <th>路段名稱<br>(起點 ➔ 終點)</th>
                <th>分段幾何<br>(距離/爬升)</th>
                <th>分段累計幾何<br>(總里程/總爬升)</th>
                <th>海拔<br>衰退率</th>
                <th>手動配瓦</th>
                <th>即時<br>推力比</th>
                <th>該段預估<br>時間</th>
                <th style="background-color: #319795;">累計完成<br>時間</th>
            </tr>
        </thead>
        <tbody id="segmentsTable">
            <!-- 由 JS 動態生成 -->
        </tbody>
        <tfoot>
            <tr class="total-row">
                <td>總計 / 平均</td>
                <td><span id="footDistance">-</span> km<br>+<span id="footClimb">-</span>m</td>
                <td class="accum-cell">—</td>
                <td>-</td>
                <td>平均：<span id="avgWatt">-</span> W</td>
                <td>平均：<span id="avgWkg">-</span> W/kg</td>
                <td>—</td>
                <td class="accum-time-cell"><span id="footTime">-</span></td>
            </tr>
        </tfoot>
    </table>
   
    <!-- 下方總累計看板 -->
    <div class="summary-dashboard">
        <div class="summary-box">
            <div class="summary-title">🏁 總累計里程</div>
            <div class="summary-value" id="totalDistance">0.00 km</div>
        </div>
        <div class="summary-box">
            <div class="summary-title">📈 總累計爬升</div>
            <div class="summary-value" id="totalClimb">0 m</div>
        </div>
        <div class="summary-box">
            <div class="summary-title">⏱️ 預估總完成時間</div>
            <div class="summary-value highlight" id="totalTime">0 小時 00 分鐘</div>
        </div>
    </div>
</div>

<script>
// 7大精確路段大數據資料庫
const segmentsData = [
    { name: "1. 埔里 ➔ 人止關", dist: 16.08, climb: 334, altitudeEff: 1.00, dbWeight: 1.15, ifMod: 1.05 },
    { name: "2. 人止關 ➔ 霧社", dist: 5.30, climb: 294, altitudeEff: 1.00, dbWeight: 1.02, ifMod: 1.00 },
    { name: "3. 霧社 ➔ 最高小七", dist: 11.98, climb: 797, altitudeEff: 0.97, dbWeight: 1.03, ifMod: 0.99 },
    { name: "4. 最高小七 ➔ 翠峰", dist: 6.00, climb: 284, altitudeEff: 0.95, dbWeight: 1.03, ifMod: 0.97 },
    { name: "5. 翠峰 ➔ 鳶峰", dist: 6.60, climb: 464, altitudeEff: 0.91, dbWeight: 1.05, ifMod: 0.94 },
    { name: "6. 鳶峰 ➔ 昆陽", dist: 5.10, climb: 327, altitudeEff: 0.87, dbWeight: 1.08, ifMod: 0.91 },
    { name: "7. 昆陽 ➔ 武嶺", dist: 2.10, climb: 197, altitudeEff: 0.82, dbWeight: 1.12, ifMod: 0.88 }
];

window.onload = function() {
    buildTableStructure();
    updateAllCalculations('reverse');
};

function buildTableStructure() {
    const tableBody = document.getElementById('segmentsTable');
    tableBody.innerHTML = "";

    segmentsData.forEach((seg, index) => {
        let row = `<tr>
            <td style="text-align:left; font-weight:bold;">${seg.name}</td>
            <td>${seg.dist.toFixed(2)} km<br>+${seg.climb}m</td>
            <td class="accum-cell" id="accum_geo_${index}">-</td>
            <td style="color:#e53e3e;">-${Math.round((1-seg.altitudeEff)*100)}%</td>
            <td>
                <input type="number" class="watt-input" id="watt_${index}" value="180" oninput="updateAllCalculations('manual')"> W
            </td>
            <td><span id="wkg_${index}">-</span> W/kg</td>
            <td style="font-weight:500; color:#2d3748;"><span id="time_${index}">-</span></td>
            <td class="accum-time-cell" id="accum_time_${index}">-</td>
        </tr>`;
        tableBody.innerHTML += row;
    });
}

function calculateSingleSegmentTime(watt, seg, totalWeight, weight) {
    let currentWatt = watt;
    if (isNaN(currentWatt) || currentWatt <= 50) currentWatt = 50;
   
    let gravityWork = (9.81 * totalWeight * seg.climb) / (currentWatt * 0.90);
    let rollingWork = (0.005 * 9.81 * totalWeight * seg.dist * 1000) / (currentWatt * 0.90);
   
    let weightPenalty = weight > 85 ? (1 + (weight - 85) * 0.002) : 1;
    return (gravityWork + rollingWork) * seg.dbWeight * weightPenalty;
}

function formatAccumTime(totalSeconds) {
    let hrs = Math.floor(totalSeconds / 3600);
    let mins = Math.floor((totalSeconds % 3600) / 60);
    let secs = Math.round(totalSeconds % 60);
   
    if (hrs > 0) {
        return `${hrs}h ${mins}m ${secs}s`;
    } else {
        return `${mins}m ${secs}s`;
    }
}

function updateAllCalculations(mode) {
    const weight = parseFloat(document.getElementById('weight').value) || 89;
    const bikeWeight = parseFloat(document.getElementById('bikeWeight').value) || 10;
    const totalWeight = weight + bikeWeight;

    // 1. 逆向推算邏輯
    if (mode === 'reverse') {
        const tHours = parseFloat(document.getElementById('target_hours').value) || 4;
        const tMins = parseFloat(document.getElementById('target_minutes').value) || 30;
        const targetTotalSeconds = (tHours * 3600) + (tMins * 60);

        let lowBaseWatt = 50;
        let highBaseWatt = 600;
        let estimatedBaseWatt = 180;

        for (let iter = 0; iter < 25; iter++) {
            estimatedBaseWatt = (lowBaseWatt + highBaseWatt) / 2;
            let simulatedTotalSeconds = 0;

            segmentsData.forEach(seg => {
                let segWatt = estimatedBaseWatt * seg.ifMod * seg.altitudeEff;
                simulatedTotalSeconds += calculateSingleSegmentTime(segWatt, seg, totalWeight, weight);
            });

            if (simulatedTotalSeconds > targetTotalSeconds) {
                lowBaseWatt = estimatedBaseWatt;
            } else {
                highBaseWatt = estimatedBaseWatt;
            }
        }

        segmentsData.forEach((seg, index) => {
            let finalSegWatt = Math.round(estimatedBaseWatt * seg.ifMod * seg.altitudeEff);
            document.getElementById(`watt_${index}`).value = finalSegWatt;
        });
    }

    // 2. 正向渲染、動態加總與分段累計運算
    let totalSeconds = 0;
    let totalDistSum = 0;
    let totalClimbSum = 0;
    let totalWattWork = 0;

    segmentsData.forEach((seg, index) => {
        let currentWatt = parseFloat(document.getElementById(`watt_${index}`).value) || 50;

        let currentWkg = currentWatt / weight;
        document.getElementById(`wkg_${index}`).innerText = currentWkg.toFixed(2);

        let segTimeSeconds = calculateSingleSegmentTime(currentWatt, seg, totalWeight, weight);
       
        totalSeconds += segTimeSeconds;
        totalDistSum += seg.dist;
        totalClimbSum += seg.climb;
        totalWattWork += currentWatt * segTimeSeconds;

        // 即時插入分段累計幾何 (乾淨無背景色)
        document.getElementById(`accum_geo_${index}`).innerHTML = `${totalDistSum.toFixed(2)} km<br>+${totalClimbSum}m`;

        let segMin = Math.floor(segTimeSeconds / 60);
        let segSec = Math.round(segTimeSeconds % 60);
        document.getElementById(`time_${index}`).innerText = `${segMin}分 ${segSec}秒`;

        // 即時插入分段累計完成時間
        document.getElementById(`accum_time_${index}`).innerText = formatAccumTime(totalSeconds);
    });

    // 3. 頁尾總計/平均計算
    let calculatedAvgWatt = totalWattWork / totalSeconds;
    let calculatedAvgWkg = calculatedAvgWatt / weight;

    document.getElementById('footDistance').innerText = totalDistSum.toFixed(2);
    document.getElementById('footClimb').innerText = totalClimbSum;
    document.getElementById('avgWatt').innerText = Math.round(calculatedAvgWatt);
    document.getElementById('avgWkg').innerText = calculatedAvgWkg.toFixed(2);
   
    let totalHours = Math.floor(totalSeconds / 3600);
    let totalMinutes = Math.floor((totalSeconds % 3600) / 60);
    document.getElementById('footTime').innerText = `${totalHours} 小時 ${totalMinutes} 分鐘`;

    // 4. 更新最下方大看板
    document.getElementById('totalDistance').innerText = totalDistSum.toFixed(2) + " km";
    document.getElementById('totalClimb').innerText = totalClimbSum + " m";
    document.getElementById('totalTime').innerText = `${totalHours} 小時 ${totalMinutes} 分鐘`;

    if (mode === 'manual') {
        document.getElementById('target_hours').value = totalHours;
        document.getElementById('target_minutes').value = totalMinutes;
    }
}
</script>

</body>
</html># 
wuling-calculator
