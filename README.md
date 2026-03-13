# <!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>九图全球-全球物流统计系统</title>
<style>
:root { --primary: #2c3e50; -- 口音：#27ae60； --蓝色：#2980b9； --橙色：#e67e22； }
body { font-family: 'Segoe UI', system-ui, sans-serif;背景：#f4f7f6；显示：柔性；调整内容：居中；内边距：20px； }
.container { 背景：白色；内边距：25px；边框半径：15px；盒子阴影：0 10px 30px rgba(0,0,0,0.1);宽度：100%； max-width: 600px; }
h2 { text-align: center; color: var(--primary); margin-bottom: 20px; border-bottom: 3px solid var(--accent); padding-bottom: 10px; }
.grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
.full { grid-column: span 2; }
.input-group { margin-bottom: 10px; }
label { display: block; margin-bottom: 5px; font-weight: 600; color: #444; font-size: 13px; }
select, input { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 8px; box-sizing: border-box; font-size: 15px; }
button { width: 100%; background: var(--primary); color: white; border: none; padding: 15px; border-radius: 8px; cursor: pointer; font-size: 16px; font-weight: bold; margin-top: 15px; transition: 0.3s; }
button:hover { background: var(--blue); }
.result-box { margin-top: 20px; display: none; }
.card { background: #fff; border: 1px solid #eee; border-radius: 8px; padding: 15px; margin-bottom: 15px; border-left: 5px solid var(--blue); }
.res-row { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #f9f9f9; .weight
-tag { color: var(--orange); font-weight: bold; }
.final-panel { background: #2c3e50; color: white; padding: 20px; border-radius: 10px; text-align: center; }
.final-val { font-size: 2.2em; font-weight: bold; color: #2ecc71; margin: 5px 0; }
.container-table { width: 100%; border-collapse: collapse; margin-top: 10px; }
.container-table th, .container-table td { border: 1px solid #eee; padding: 10px; text-align: center; font-size: 13px; }
</style>
</head>
<body>
<div class="container">
<h2>地毯装箱与计费核算</h2>
<div class="grid">
<div class="input-group full">
<label>运输方式(计费系数)</label>
<select id="vFactor">
<option value="5000">国际快递(除以5000)</option>
<option value="6000">空运/海运拼箱(除以6000)</option>
</select>
</div>
<div class="input-group full">
<label>打包方式</label>
<select id="packType" onchange="toggleInputs()">
<option value="roll">卷装地毯(按方体截面)</option>
<option value="flat">折叠/箱装(标准长方体)</option>
</select>
</div>
<div class="input-group">
<label id="dim1Label">卷轴长度(cm)</label>
<input type="number" id="dim1" placeholder="L">
</div>
<div class="input-group">
<label id="dim2Label">直径/宽度(cm)</label>
<input type="number" id="dim2" placeholder="D/W">
</div>
<div id="hGroup" class="input-group" style="display:none">
<label>高度(cm)</label>
<input type="number" id="dim3" placeholder="H">
</div>
<div class="input-group">
<label>单件实重(kg)</label>
<input type="number" id="singleWeight" placeholder="测量重量">
</div>
<div class="input-group">
<label>总件数</label>
<input type="number" id="qty" value="1">
</div>
</div>
<button onclick="calculateAll()">执行物流核算</button>
<div id="result" class="result-box">
<div class="card">
<div class="res-row"><span>总体积:</span> <span><strong id="totalCbm">0</strong> CBM</span></div>
<div class="res-row"><span>总实重:<span class="weight-tag"><span id="totalActual">0</span> kg</span></div>
<div class="res-row"><span>体积重：</span> <span class="weight-tag"><span id="volWeight">0</span> kg</span></div>
</div>
<div class="final-panel">
<div style="font-size: 13px; opacity: 0.8;">最终计费重量 (Chargeable Weight)</div>
<div class="final-val" id="finalWeight">0<small style="font-size:15px; margin-left:5px;">kg</small></div>
<div id="weightNote" style="font-size: 12px; color: #bdc3c7;"></div>
</div>
<div class="card" style="border-left-color: var(--accent); margin-top: 15px;">
<label style="color: var(--accent); font-weight: bold;">📊 整柜装载量预估 (扣除10%损耗):</label>
<table class="container-table">
<thead><tr><th>柜型</th><th>预估装数</th></tr></thead>
<tbody>
<tr><td>20GP 小柜</td><td id="num20">0 件</td></tr>
<tr><td>40GP 大柜</td><td id="num40">0 件</td></tr>
<tr><td>40HQ 高柜</td><td id="num40hq">0 件</td></tr>
</tbody>
</table>
</div>
</div>
</div>
<script>
function toggleInputs() {
const type = document.getElementById('packType').value;
const hGroup = document.getElementById('hGroup');
document.getElementById('dim1Label').innerText = (type === 'roll') ? "卷轴长度 (cm)" : "长度 (cm)";
hGroup.style.display = (type === 'flat') ? "block" : "none";
}
function calculateAll() {
const type = document.getElementById('packType').value;
const d1 = parseFloat(document.getElementById('dim1').value) || 0;
const d2 = parseFloat(document.getElementById('dim2').value) || 0;
const d3 = parseFloat(document.getElementById('dim3').value) || 0;
const singleW = parseFloat(document.getElementById('singleWeight').value) || 0;
const qty = parseInt(document.getElementById('qty').value) || 1;
const factor = parseInt(document.getElementById('vFactor').value);
let unitCbm = (type === 'roll') ? (d1 * d2 * d2 / 1000000) : (d1 * d2 * d3 / 1000000);
const totalCbm = unitCbm * qty;
const totalActualWeight = singleW * qty;
const totalVolWeight = (totalCbm * 1000000 / factor);
const finalWeight = Math.max(totalActualWeight, totalVolWeight);
const unitWithLoss = unitCbm * 1.1; // 预留 10% 空隙
const calcBox = (cap) => unitWithLoss > 0 ? Math.floor(cap / unitWithLoss) : 0;

document.getElementById('totalCbm').innerText = totalCbm.toFixed(3);
document.getElementById('totalActual').innerText = totalActualWeight.toFixed(2);
document.getElementById('volWeight').innerText = totalVolWeight.toFixed(2);
document.getElementById('finalWeight').innerHTML = finalWeight.toFixed(2) + '<small style="font-size:15px; margin-left:5px;">kg</small>';
document.getElementById('weightNote').innerText = totalVolWeight > totalActualWeight ? "⚠️泡货开关" : "✅实重开关";
document.getElementById('num20').innerText = calcBox(28) + " 件";
document.getElementById('num40').innerText = calcBox(58) + " 件";
document.getElementById('num40hq').innerText = calcBox(68) + " 件";
document.getElementById('结果').style.display = '块';
}
</脚本>
</body>
</html>
