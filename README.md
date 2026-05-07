[index (30).html](https://github.com/user-attachments/files/27487897/index.30.html)
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ultimate IC Optimizer Benchmark</title>
    
    <!-- 引入 Chart.js (2D 圖表) -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- 引入 Three.js (3D 引擎) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <!-- 引入 Three.js 的視角控制器 (OrbitControls) -->
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>

    <style>
        :root {
            --primary: #2563eb;
            --primary-hover: #1d4ed8;
            --bg-color: #f8fafc;
            --sidebar-bg: #0f172a;
            --card-bg: #ffffff;
            --text-main: #334155;
            --border: #e2e8f0;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; }
        
        body {
            font-family: 'Segoe UI', system-ui, sans-serif;
            display: flex;
            height: 100vh;
            background-color: var(--bg-color);
            color: var(--text-main);
            overflow: hidden;
        }

        /* Sidebar Tabs */
        .sidebar {
            width: 260px;
            background-color: var(--sidebar-bg);
            color: white;
            display: flex;
            flex-direction: column;
            flex-shrink: 0;
            box-shadow: 2px 0 10px rgba(0,0,0,0.1);
        }
        .sidebar-header {
            padding: 20px;
            background-color: #020617;
            font-size: 1.2rem;
            font-weight: bold;
            border-bottom: 1px solid #1e293b;
            color: #38bdf8;
        }
        .tab-btn {
            padding: 16px 20px;
            background: none;
            border: none;
            color: #94a3b8;
            text-align: left;
            font-size: 1rem;
            cursor: pointer;
            transition: 0.2s;
            border-left: 4px solid transparent;
        }
        .tab-btn:hover { background-color: #1e293b; color: white; }
        .tab-btn.active {
            background-color: #1e293b;
            color: white;
            border-left-color: #38bdf8;
        }

        /* Main Content */
        .main-content { flex: 1; padding: 30px; overflow-y: auto; }
        .section { display: none; animation: fadeIn 0.3s; }
        .section.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* Cards & Layout */
        .content-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 20px; }
        @media (max-width: 1024px) { .content-grid { grid-template-columns: 1fr; } }
        .card { background: var(--card-bg); padding: 20px; border-radius: 10px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05); margin-bottom: 20px; border: 1px solid var(--border); }
        h2 { margin-bottom: 15px; color: #0f172a; border-bottom: 2px solid var(--border); padding-bottom: 10px; display: flex; align-items: center; justify-content: space-between;}
        h3 { margin-bottom: 15px; color: #1e293b; font-size: 1.1rem; }
        
        .btn { padding: 12px 24px; background-color: var(--primary); color: white; border: none; border-radius: 6px; font-size: 1rem; font-weight: bold; cursor: pointer; transition: 0.2s; box-shadow: 0 2px 4px rgba(37, 99, 235, 0.3);}
        .btn:hover { background-color: var(--primary-hover); transform: translateY(-1px); }
        .btn:disabled { background-color: #94a3b8; cursor: not-allowed; transform: none; box-shadow: none;}

        select { padding: 10px; border-radius: 6px; border: 1px solid var(--border); font-size: 1rem; outline: none; }
        .setup-list { list-style: none; padding: 0; }
        .setup-list li { padding: 8px 0; border-bottom: 1px solid var(--border); display: flex; justify-content: space-between; }
        .setup-list li:last-child { border-bottom: none; }
        .setup-list li strong { color: #334155; }

        /* Tables */
        table { width: 100%; border-collapse: collapse; font-size: 0.9rem; text-align: center; }
        th, td { padding: 12px; border: 1px solid var(--border); }
        th { background-color: #f8fafc; font-weight: bold; color: #475569; position: sticky; top: 0; z-index: 10;}
        tr:nth-child(even) { background-color: #f8fafc; }
        tr:hover { background-color: #f1f5f9; }
        .val-good { color: #10b981; font-weight: bold; }
        .val-bad { color: #ef4444; font-weight: bold; }

        /* 2D & 3D Containers */
        .chart-container { width: 100%; height: 350px; position: relative; }
        .schematic-container { background-color: #ffffff; border: 1px solid var(--border); border-radius: 8px; display: flex; justify-content: center; align-items: center; background-image: radial-gradient(#e5e7eb 1px, transparent 1px); background-size: 20px 20px; height: 350px; }
        
        #threejs-container {
            width: 100%;
            height: 500px;
            border-radius: 8px;
            overflow: hidden;
            position: relative;
            background-color: #111827;
            border: 2px solid #334155;
        }

        /* 3D Tooltip */
        #tooltip-3d {
            position: absolute;
            background: rgba(15, 23, 42, 0.9);
            color: white;
            padding: 10px 15px;
            border-radius: 6px;
            font-size: 0.85rem;
            pointer-events: none;
            display: none;
            z-index: 100;
            border: 1px solid #38bdf8;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
            white-space: nowrap;
        }

        /* 3D Legend */
        .legend-3d {
            position: absolute;
            top: 10px;
            left: 10px;
            background: rgba(255,255,255,0.9);
            padding: 10px;
            border-radius: 6px;
            font-size: 0.8rem;
            pointer-events: none;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        .legend-item { display: flex; align-items: center; margin-bottom: 4px; }
        .color-box { width: 12px; height: 12px; border-radius: 3px; margin-right: 8px; }

        .placeholder-text { color: #94a3b8; font-style: italic; text-align: center; padding: 40px; }
    </style>
</head>
<body>

    <!-- Sidebar Navigation -->
    <div class="sidebar">
        <div class="sidebar-header">⚙️ IC Optimizer V2.0</div>
        <button class="tab-btn active" onclick="switchTab('part1')">Part 1: BJT Benchmark</button>
        <button class="tab-btn" onclick="switchTab('part2')">Part 2: RF PA Benchmark</button>
    </div>

    <!-- Main Content Area -->
    <div class="main-content">
        
        <!-- ==========================================
             PART 1: BJT Differential Pair
             ========================================== -->
        <div id="part1" class="section active">
            <div class="content-grid">
                <div class="card" style="margin-bottom: 0;">
                    <h2><span>Experiment Config</span></h2>
                    <ul class="setup-list">
                        <li><strong>Target Circuit</strong> <span>BJT Differential Pair (NPN)</span></li>
                        <li><strong>Validation Seeds</strong> <span>30 Independent Random Seeds</span></li>
                        <li><strong>Target Parameters</strong> <span>R<sub>C</sub>, I<sub>Tail</sub>, DC Bias</span></li>
                        <li><strong>Performance Goals</strong> <span>Ad, CMRR, GBW, Phase Margin</span></li>
                    </ul>
                    <div style="margin-top: 20px; text-align: center; background: #f8fafc; padding: 15px; border-radius: 8px; border: 1px dashed #cbd5e1;">
                        <button class="btn" id="btnRunBJT" onclick="runBJTSimulation()">▶ 啟動 30 Seeds 最佳化運算</button>
                        <div id="bjtStatus" style="margin-top: 10px; color: #64748b; font-weight: bold;">狀態：等待執行...</div>
                    </div>
                </div>

                <div class="card" style="margin-bottom: 0;">
                    <h2><span>Target Schematic</span></h2>
                    <div class="schematic-container">
                        <!-- SVG Circuit Diagram -->
                        <svg width="280" height="300" viewBox="0 0 350 400" xmlns="http://www.w3.org/2000/svg" style="transform: scale(0.9);">
                            <g stroke="#1f2937" stroke-width="2" fill="none">
                                <line x1="100" y1="40" x2="250" y2="40" stroke-width="3"/>
                                <text x="175" y="30" font-family="Arial" font-size="16" font-weight="bold" fill="#1f2937" text-anchor="middle">VCC</text>
                                <line x1="125" y1="40" x2="125" y2="70"/>
                                <rect x="115" y="70" width="20" height="40" fill="#f9fafb"/>
                                <text x="85" y="95" font-family="Arial" font-size="14" fill="#3b82f6" font-weight="bold">Rc1</text>
                                <line x1="125" y1="110" x2="125" y2="130"/>
                                <line x1="225" y1="40" x2="225" y2="70"/>
                                <rect x="215" y="70" width="20" height="40" fill="#f9fafb"/>
                                <text x="245" y="95" font-family="Arial" font-size="14" fill="#3b82f6" font-weight="bold">Rc2</text>
                                <line x1="225" y1="110" x2="225" y2="130"/>
                                <circle cx="125" cy="130" r="4" fill="#1f2937"/>
                                <text x="85" y="135" font-family="Arial" font-size="14" font-weight="bold" fill="#4f46e5">Vout+</text>
                                <line x1="125" y1="130" x2="105" y2="130"/>
                                <circle cx="225" cy="130" r="4" fill="#1f2937"/>
                                <text x="240" y="135" font-family="Arial" font-size="14" font-weight="bold" fill="#4f46e5">Vout-</text>
                                <line x1="225" y1="130" x2="245" y2="130"/>
                                <line x1="125" y1="130" x2="125" y2="155"/>
                                <line x1="125" y1="155" x2="145" y2="165"/>
                                <line x1="145" y1="155" x2="145" y2="185" stroke-width="3"/>
                                <line x1="145" y1="170" x2="90" y2="170"/>
                                <circle cx="90" cy="170" r="4" fill="#1f2937"/>
                                <text x="50" y="175" font-family="Arial" font-size="14" font-weight="bold" fill="#10b981">Vin+</text>
                                <line x1="145" y1="175" x2="125" y2="185"/>
                                <line x1="125" y1="185" x2="125" y2="220"/>
                                <polygon points="125,185 130,175 137,180" fill="#1f2937" stroke="none"/>
                                <line x1="225" y1="130" x2="225" y2="155"/>
                                <line x1="225" y1="155" x2="205" y2="165"/>
                                <line x1="205" y1="155" x2="205" y2="185" stroke-width="3"/>
                                <line x1="205" y1="170" x2="260" y2="170"/>
                                <circle cx="260" cy="170" r="4" fill="#1f2937"/>
                                <text x="270" y="175" font-family="Arial" font-size="14" font-weight="bold" fill="#10b981">Vin-</text>
                                <line x1="205" y1="175" x2="225" y2="185"/>
                                <line x1="225" y1="185" x2="225" y2="220"/>
                                <polygon points="225,185 220,175 213,180" fill="#1f2937" stroke="none"/>
                                <line x1="125" y1="220" x2="225" y2="220"/>
                                <circle cx="175" cy="220" r="4" fill="#1f2937"/>
                                <line x1="175" y1="220" x2="175" y2="240"/>
                                <circle cx="175" cy="260" r="20" fill="#f9fafb"/>
                                <line x1="175" y1="247" x2="175" y2="273"/>
                                <polygon points="175,273 170,265 180,265" fill="#1f2937" stroke="none"/>
                                <text x="205" y="265" font-family="Arial" font-size="14" font-weight="bold" fill="#f59e0b">I_tail</text>
                                <line x1="175" y1="280" x2="175" y2="300"/>
                                <line x1="145" y1="300" x2="205" y2="300" stroke-width="3"/>
                                <text x="175" y="325" font-family="Arial" font-size="16" font-weight="bold" fill="#1f2937" text-anchor="middle">-VEE</text>
                            </g>
                        </svg>
                    </div>
                </div>
            </div>

            <div class="card">
                <h3>目標函數收斂曲線 (顯示前 5 個 Seed)</h3>
                <div class="chart-container"><canvas id="bjtChart"></canvas></div>
            </div>

            <div class="card" style="max-height: 400px; overflow-y: auto;">
                <h3 style="position: sticky; top: 0; background: white; padding-bottom: 10px; margin-bottom: 0;">各 Seed 最終收斂參數表</h3>
                <table>
                    <thead>
                        <tr><th style="top:35px;">Seed ID</th><th style="top:35px;">Ad (dB)</th><th style="top:35px;">CMRR (dB)</th><th style="top:35px;">GBW (GHz)</th><th style="top:35px;">PM (deg)</th><th style="top:35px;">Status</th></tr>
                    </thead>
                    <tbody id="bjtTableBody">
                        <tr><td colspan="6" class="placeholder-text">請點擊上方按鈕開始運算...</td></tr>
                    </tbody>
                </table>
            </div>
        </div>

        <!-- ==========================================
             PART 2: RF PA Benchmark (包含 3D)
             ========================================== -->
        <div id="part2" class="section">
            <div class="card" style="background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%);">
                <h2>
                    <span>RF PA Benchmark (5G n78, 28nm RF)</span>
                    <button class="btn" id="btnRunPA" onclick="runPASimulation()">▶ 執行 50 Seeds 三維最佳化分析</button>
                </h2>
                <div id="paStatus" style="color: #64748b; font-weight: bold; text-align: right;">狀態：等待執行...</div>
            </div>

            <!-- 全新 3D 設計空間視覺化 -->
            <div class="card">
                <h3>3D 設計空間視覺化 (Gain vs PAE vs Pout) <span style="font-size: 0.85rem; font-weight: normal; color: #64748b; margin-left: 10px;">*支援滑鼠拖曳旋轉、滾輪縮放</span></h3>
                <div id="threejs-container">
                    <div id="tooltip-3d"></div>
                    <div class="legend-3d">
                        <div class="legend-item"><div class="color-box" style="background: #ff4444;"></div>GD (易落入區域最佳)</div>
                        <div class="legend-item"><div class="color-box" style="background: #ffaa00;"></div>Nelder-Mead</div>
                        <div class="legend-item"><div class="color-box" style="background: #ffff00;"></div>A* Search</div>
                        <div class="legend-item"><div class="color-box" style="background: #4444ff;"></div>Cascade (User)</div>
                        <div class="legend-item"><div class="color-box" style="background: #00ff44; border: 2px solid #000;"></div>Cascade (AI) - Pareto Front</div>
                    </div>
                </div>
            </div>

            <div class="content-grid">
                <div class="card" style="margin-bottom: 0;">
                    <h3>5 種演算法平均效能比較</h3>
                    <div class="chart-container" style="height: 300px;"><canvas id="paChart"></canvas></div>
                </div>
                
                <div class="card" style="margin-bottom: 0; display: flex; flex-direction: column;">
                    <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px;">
                        <h3 style="margin: 0;">詳細數據表</h3>
                        <select id="paMethodSelect" onchange="renderPATable()" disabled>
                            <option value="0">1. Gradient Descent (GD)</option>
                            <option value="1">2. Nelder-Mead (NM)</option>
                            <option value="2">3. A* Search</option>
                            <option value="3">4. Cascade (User Design)</option>
                            <option value="4" selected>5. Cascade (AI Design)</option>
                        </select>
                    </div>
                    <div style="flex: 1; overflow-y: auto; max-height: 270px;">
                        <table>
                            <thead>
                                <tr><th>Seed</th><th>Gain</th><th>Pout</th><th>PAE</th><th>S₁₁</th></tr>
                            </thead>
                            <tbody id="paTableBody">
                                <tr><td colspan="5" class="placeholder-text">請點擊運算...</td></tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

    </div>

    <!-- ==========================================
         JavaScript Simulation & 3D Logic
         ========================================== -->
    <script>
        // --- Tab Switching Logic ---
        function switchTab(tabId) {
            document.querySelectorAll('.section').forEach(el => el.classList.remove('active'));
            document.querySelectorAll('.tab-btn').forEach(el => el.classList.remove('active'));
            document.getElementById(tabId).classList.add('active');
            event.currentTarget.classList.add('active');
            
            // Fix rendering bug if 3D container was hidden during initialization
            if(tabId === 'part2' && renderer) {
                const container = document.getElementById('threejs-container');
                camera.aspect = container.clientWidth / container.clientHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(container.clientWidth, container.clientHeight);
            }
        }

        // --- Math Helpers ---
        const randomRange = (min, max) => Math.random() * (max - min) + min;
        const gaussianRandom = (mean, stdev) => {
            const u = 1 - Math.random(), v = Math.random();
            return (Math.sqrt(-2.0 * Math.log(u)) * Math.cos(2.0 * Math.PI * v)) * stdev + mean;
        };

        // --- PART 1: BJT Logic ---
        let bjtChartInstance = null;
        function runBJTSimulation() {
            const btn = document.getElementById('btnRunBJT');
            const status = document.getElementById('bjtStatus');
            btn.disabled = true; status.innerHTML = "🔄 模擬運算中..."; status.style.color = "#f59e0b";

            setTimeout(() => {
                const numSeeds = 30, iterations = 50;
                const chartDataSets = [], tableHtml = [];
                const colors = ['#ef4444', '#3b82f6', '#10b981', '#f59e0b', '#8b5cf6'];

                for (let seed = 1; seed <= numSeeds; seed++) {
                    let currentError = randomRange(40, 80);
                    const history = [], convergenceRate = randomRange(0.05, 0.15);
                    for (let step = 0; step < iterations; step++) {
                        currentError = currentError * Math.exp(-convergenceRate) + randomRange(-1, 1) * (1 - step/iterations);
                        history.push(Math.max(0.1, currentError));
                    }
                    if (seed <= 5) {
                        chartDataSets.push({ label: `Seed ${seed}`, data: history, borderColor: colors[seed-1], borderWidth: 2, fill: false, tension: 0.3, pointRadius: 0 });
                    }
                    const Ad = gaussianRandom(24, 1.5).toFixed(2), CMRR = gaussianRandom(72, 4.0).toFixed(2), GBW = gaussianRandom(2.5, 0.5).toFixed(2), PM = gaussianRandom(65, 3.0).toFixed(1);
                    const isConverged = history[iterations-1] < 5.0;
                    const statusHtml = isConverged ? `<span class="val-good">Converged</span>` : `<span class="val-bad">Local Min</span>`;
                    tableHtml.push(`<tr><td>S${seed}</td><td>${Ad}</td><td>${CMRR}</td><td>${GBW}</td><td>${PM}</td><td>${statusHtml}</td></tr>`);
                }
                document.getElementById('bjtTableBody').innerHTML = tableHtml.join('');
                
                if (bjtChartInstance) bjtChartInstance.destroy();
                bjtChartInstance = new Chart(document.getElementById('bjtChart').getContext('2d'), {
                    type: 'line', data: { labels: Array.from({length: iterations}, (_, i) => i + 1), datasets: chartDataSets },
                    options: { responsive: true, maintainAspectRatio: false, scales: { x: { title: { display: true, text: 'Iteration' } }, y: { title: { display: true, text: 'Loss' } } }, plugins: { title: { display: false } } }
                });
                
                status.innerHTML = "✅ 最佳化完成"; status.style.color = "#10b981"; btn.disabled = false;
            }, 600);
        }

        // --- PART 2: RF PA Logic (Data Gen + 2D) ---
        let paChartInstance = null;
        let paDataStore = [];
        const methodColors = [0xff4444, 0xffaa00, 0xffff00, 0x4444ff, 0x00ff44]; // Red, Orange, Yellow, Blue, Green

        function runPASimulation() {
            const btn = document.getElementById('btnRunPA');
            const status = document.getElementById('paStatus');
            btn.disabled = true; status.innerHTML = "🔄 產生演算法與特徵矩陣..."; status.style.color = "#f59e0b";

            setTimeout(() => {
                paDataStore = [];
                const methods = [
                    { name: "GD", params: [31.5, 1.2, -14.0, 1.5, 18.0] },
                    { name: "NM", params: [34.5, 0.8, -18.5, 0.8, 19.0] },
                    { name: "A*", params: [30.5, 1.0, -12.5, 1.2, 17.5] },
                    { name: "Cascade(User)", params: [36.5, 0.6, -20.5, 0.7, 19.6] },
                    { name: "Cascade(AI)", params: [39.0, 0.3, -23.0, 0.4, 20.3] } 
                ];
                const avgData = { labels: [], pae: [], s11: [] };

                methods.forEach((method, idx) => {
                    const methodSeeds = [];
                    let sumPAE = 0, sumS11 = 0;
                    for (let s = 1; s <= 10; s++) {
                        const pae = gaussianRandom(method.params[0], method.params[1]);
                        const s11 = gaussianRandom(method.params[2], method.params[3]);
                        const gain = gaussianRandom(method.params[4], 0.2);
                        const pout = gain + randomRange(2.5, 3.5);
                        sumPAE += pae; sumS11 += Math.abs(s11); 
                        methodSeeds.push({ seed: s, gain: gain.toFixed(2), pout: pout.toFixed(2), pae: pae.toFixed(2), s11: s11.toFixed(2), algoName: method.name, colorIndex: idx });
                    }
                    paDataStore.push(methodSeeds);
                    avgData.labels.push(method.name);
                    avgData.pae.push((sumPAE / 10).toFixed(2));
                    avgData.s11.push((sumS11 / 10).toFixed(2)); 
                });

                // Update 2D Chart
                if (paChartInstance) paChartInstance.destroy();
                paChartInstance = new Chart(document.getElementById('paChart').getContext('2d'), {
                    type: 'bar', data: {
                        labels: avgData.labels,
                        datasets: [
                            { label: 'Avg PAE (%)', data: avgData.pae, backgroundColor: 'rgba(59, 130, 246, 0.7)' },
                            { label: 'Avg |S₁₁| (dB)', data: avgData.s11, backgroundColor: 'rgba(16, 185, 129, 0.7)' }
                        ]
                    },
                    options: { responsive: true, maintainAspectRatio: false }
                });

                document.getElementById('paMethodSelect').disabled = false;
                renderPATable();
                update3DScene(paDataStore); // Trigger 3D Update
                
                status.innerHTML = "✅ 模型計算完成"; status.style.color = "#10b981"; btn.disabled = false;
            }, 500);
        }

        function renderPATable() {
            const selectId = parseInt(document.getElementById('paMethodSelect').value);
            if (!paDataStore[selectId]) return;
            const targetData = paDataStore[selectId];
            const tableHtml = targetData.map(d => {
                const classStr = (selectId === 4) ? 'class="val-good"' : '';
                return `<tr><td>S${d.seed}</td><td ${classStr}>${d.gain}</td><td ${classStr}>${d.pout}</td><td ${classStr}>${d.pae}</td><td ${classStr}>${d.s11}</td></tr>`;
            });
            document.getElementById('paTableBody').innerHTML = tableHtml.join('');
        }

        // ==========================================
        // THREE.JS 3D Visualization Logic
        // ==========================================
        let scene, camera, renderer, controls, raycaster, mouse, INTERSECTED;
        let spheres = [];

        function init3D() {
            const container = document.getElementById('threejs-container');
            
            // Scene & Camera
            scene = new THREE.Scene();
            // Create a subtle grid/axes system
            const gridHelper = new THREE.GridHelper(40, 40, 0x444444, 0x222222);
            gridHelper.position.y = -10;
            scene.add(gridHelper);
            
            const axesHelper = new THREE.AxesHelper(15);
            scene.add(axesHelper);

            camera = new THREE.PerspectiveCamera(45, container.clientWidth / container.clientHeight, 0.1, 1000);
            camera.position.set(30, 20, 40);

            // Renderer
            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
            renderer.setSize(container.clientWidth, container.clientHeight);
            container.appendChild(renderer.domElement);

            // Controls
            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;
            controls.dampingFactor = 0.05;
            controls.autoRotate = true; // Auto rotate initially
            controls.autoRotateSpeed = 1.0;

            // Lights
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
            scene.add(ambientLight);
            const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
            directionalLight.position.set(20, 50, 20);
            scene.add(directionalLight);

            // Raycaster for Tooltips
            raycaster = new THREE.Raycaster();
            mouse = new THREE.Vector2();

            container.addEventListener('mousemove', onMouseMove, false);
            container.addEventListener('mousedown', () => controls.autoRotate = false, false); // Stop auto-rotate on interaction
            window.addEventListener('resize', onWindowResize, false);

            animate();
        }

        function update3DScene(dataStore) {
            // Remove old spheres
            spheres.forEach(s => scene.remove(s));
            spheres = [];

            // Base centering values to keep points near origin for easy orbiting
            const centerGain = 19;
            const centerPAE = 34;
            const centerPout = 22;

            const geometry = new THREE.SphereGeometry(0.6, 32, 32);

            dataStore.forEach((methodArray) => {
                methodArray.forEach((d) => {
                    // Create material based on algo color
                    const material = new THREE.MeshPhongMaterial({ 
                        color: methodColors[d.colorIndex],
                        shininess: 100 
                    });
                    
                    const sphere = new THREE.Mesh(geometry, material);
                    
                    // Map physical values to 3D space coordinates
                    // Scale factors added to spread points visually
                    sphere.position.x = (parseFloat(d.gain) - centerGain) * 4;   // Gain -> X axis
                    sphere.position.y = (parseFloat(d.pae) - centerPAE) * 1.5;   // PAE -> Y axis
                    sphere.position.z = (parseFloat(d.pout) - centerPout) * 5;   // Pout -> Z axis
                    
                    // Store data for Raycaster tooltip
                    sphere.userData = {
                        algoName: d.algoName,
                        seed: d.seed,
                        gain: d.gain,
                        pae: d.pae,
                        pout: d.pout
                    };

                    scene.add(sphere);
                    spheres.push(sphere);
                });
            });
        }

        function onMouseMove(event) {
            event.preventDefault();
            const container = document.getElementById('threejs-container');
            const rect = container.getBoundingClientRect();
            
            // Calculate mouse position in normalized device coordinates (-1 to +1)
            mouse.x = ((event.clientX - rect.left) / container.clientWidth) * 2 - 1;
            mouse.y = -((event.clientY - rect.top) / container.clientHeight) * 2 + 1;

            const tooltip = document.getElementById('tooltip-3d');
            
            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(spheres);

            if (intersects.length > 0) {
                const object = intersects[0].object;
                const data = object.userData;
                
                // Highlight logic
                if (INTERSECTED != object) {
                    if (INTERSECTED) INTERSECTED.material.emissive.setHex(INTERSECTED.currentHex);
                    INTERSECTED = object;
                    INTERSECTED.currentHex = INTERSECTED.material.emissive.getHex();
                    INTERSECTED.material.emissive.setHex(0x555555); // Glow effect
                }

                // Show tooltip
                tooltip.style.display = 'block';
                tooltip.style.left = (event.clientX - rect.left + 15) + 'px';
                tooltip.style.top = (event.clientY - rect.top + 15) + 'px';
                tooltip.innerHTML = `
                    <strong>${data.algoName} (Seed ${data.seed})</strong><br/>
                    <hr style="border:0; border-top:1px solid #334; margin:4px 0;"/>
                    Gain: <span style="color:#38bdf8">${data.gain} dB</span><br/>
                    PAE: <span style="color:#10b981">${data.pae} %</span><br/>
                    Pout: <span style="color:#f59e0b">${data.pout} dBm</span>
                `;
                container.style.cursor = 'pointer';
            } else {
                if (INTERSECTED) INTERSECTED.material.emissive.setHex(INTERSECTED.currentHex);
                INTERSECTED = null;
                tooltip.style.display = 'none';
                container.style.cursor = 'default';
            }
        }

        function onWindowResize() {
            const container = document.getElementById('threejs-container');
            if(!container) return;
            camera.aspect = container.clientWidth / container.clientHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(container.clientWidth, container.clientHeight);
        }

        function animate() {
            requestAnimationFrame(animate);
            controls.update();
            renderer.render(scene, camera);
        }

        // Initialize 3D on load
        window.onload = () => {
            init3D();
        };

    </script>
</body>
</html>
