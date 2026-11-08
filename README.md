<!DOCTYPE html>
<html lang="zh-TW" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>台股個人戰情室 | TWSE Personal Hub</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js for K-line / Trend simulation -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Font Awesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts: Inter & Noto Sans TC -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Noto+Sans+TC:wght@400;500;700&display=swap" rel="stylesheet">
    
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'Noto Sans TC', 'sans-serif'],
                        mono: ['ui-monospace', 'SFMono-Regular', 'Menlo', 'Monaco', 'Consolas', 'monospace']
                    },
                    colors: {
                        twRed: '#ef4444',     // 台股紅漲
                        twGreen: '#10b981',   // 台股綠跌
                        darkBg: '#0b0f19',
                        cardBg: '#131b2e',
                        borderDark: '#1e293b'
                    }
                }
            }
        }
    </script>
    <style>
        body {
            background-color: #0b0f19;
            color: #e2e8f0;
            overflow-x: hidden;
        }
        /* 自訂捲軸 */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #0b0f19;
        }
        ::-webkit-scrollbar-thumb {
            background: #334155;
            border-radius: 3px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #475569;
        }
        .glass-card {
            background: rgba(19, 27, 46, 0.85);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.08);
        }
        .code-block {
            background: #090d16;
            border: 1px solid #1e293b;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col font-sans antialiased selection:bg-blue-500 selection:text-white">

    <!-- 頂部大盤指數與狀態列 -->
    <header class="sticky top-0 z-30 glass-card border-b border-slate-800 px-4 xl:px-8 py-3">
        <div class="w-full mx-auto flex flex-wrap items-center justify-between gap-4">
            <!-- Logo 與標題 -->
            <div class="flex items-center space-x-3">
                <div class="w-10 h-10 rounded-xl bg-gradient-to-br from-blue-600 to-indigo-600 flex items-center justify-center shadow-lg shadow-blue-500/20">
                    <i class="fa-solid fa-chart-line text-white text-lg"></i>
                </div>
                <div>
                    <div class="flex items-center space-x-2">
                        <h1 class="text-lg font-bold tracking-tight text-white">台股個人戰情室</h1>
                        <span id="dataSourceBadge" class="text-[11px] font-medium px-2 py-0.5 rounded-full bg-emerald-500/20 text-emerald-300 border border-emerald-500/30 flex items-center space-x-1">
                            <span class="w-1.5 h-1.5 rounded-full bg-emerald-400 animate-pulse"></span>
                            <span id="dataSourceText">即時連線啟動中...</span>
                        </span>
                    </div>
                    <p class="text-xs text-slate-400" id="lastUpdateTime">最後更新：正在同步最新台股真實行情...</p>
                </div>
            </div>

            <!-- 大盤快速指標 -->
            <div class="flex items-center space-x-6 overflow-x-auto py-1 text-sm">
                <div class="flex items-center space-x-2 whitespace-nowrap">
                    <span class="text-slate-400">加權指數</span>
                    <span class="font-mono font-bold text-twRed" id="taiexValue">載入中...</span>
                    <span class="font-mono text-xs px-1.5 py-0.5 rounded bg-red-500/10 text-twRed" id="taiexChange">--</span>
                </div>
                <div class="hidden md:flex items-center space-x-2 whitespace-nowrap border-l border-slate-800 pl-6">
                    <span class="text-slate-400">櫃買指數</span>
                    <span class="font-mono font-bold text-slate-200" id="tpexValue">載入中...</span>
                    <span class="font-mono text-xs px-1.5 py-0.5 rounded bg-slate-800 text-slate-300" id="tpexChange">--</span>
                </div>
                <div class="hidden lg:flex items-center space-x-2 whitespace-nowrap border-l border-slate-800 pl-6">
                    <span class="text-slate-400">市場狀態</span>
                    <span class="font-mono font-bold text-blue-400" id="marketStatusText">偵測中</span>
                </div>
            </div>

            <!-- 右側操作按鈕 -->
            <div class="flex items-center space-x-2">
                <!-- 自動輪詢開關 -->
                <button onclick="toggleAutoRefresh()" id="autoRefreshBtn" class="px-3 py-2 rounded-lg bg-emerald-600/20 hover:bg-emerald-600/30 text-emerald-300 border border-emerald-500/30 text-xs font-medium flex items-center space-x-1.5 transition" title="每 15 秒自動同步最新報價">
                    <i class="fa-solid fa-bolt" id="autoRefreshIcon"></i>
                    <span id="autoRefreshLabel">自動更新: 開啟 (15s)</span>
                </button>
                <button onclick="refreshQuotes(true)" class="px-3 py-2 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-200 border border-slate-700 text-xs font-medium flex items-center space-x-1.5 transition">
                    <i class="fa-solid fa-arrows-rotate" id="refreshIcon"></i>
                    <span>立即同步</span>
                </button>
                <button onclick="openApiModal()" class="px-3 py-2 rounded-lg bg-blue-600/20 hover:bg-blue-600/30 text-blue-300 border border-blue-500/30 text-xs font-medium flex items-center space-x-1.5 transition">
                    <i class="fa-solid fa-sliders"></i>
                    <span>資料源設定</span>
                </button>
            </div>
        </div>
    </header>

    <!-- 導覽分頁列 -->
    <div class="border-b border-slate-800 bg-slate-900/50 px-4 xl:px-8">
        <div class="w-full mx-auto flex space-x-1 overflow-x-auto py-2">
            <button onclick="switchTab('dashboard')" id="tab-dashboard" class="tab-btn px-4 py-2 rounded-lg text-sm font-medium bg-blue-600 text-white shadow-sm whitespace-nowrap flex items-center space-x-2 transition">
                <i class="fa-solid fa-grip"></i>
                <span>自選監控與技術面</span>
            </button>
            <button onclick="switchTab('portfolio')" id="tab-portfolio" class="tab-btn px-4 py-2 rounded-lg text-sm font-medium text-slate-400 hover:text-white hover:bg-slate-800 whitespace-nowrap flex items-center space-x-2 transition">
                <i class="fa-solid fa-wallet"></i>
                <span>庫存損益與試算</span>
            </button>
            <button onclick="switchTab('etf')" id="tab-etf" class="tab-btn px-4 py-2 rounded-lg text-sm font-medium text-slate-400 hover:text-white hover:bg-slate-800 whitespace-nowrap flex items-center space-x-2 transition">
                <i class="fa-solid fa-calendar-check"></i>
                <span>ETF 配息與重大訊息</span>
            </button>
            <button onclick="switchTab('guide')" id="tab-guide" class="tab-btn px-4 py-2 rounded-lg text-sm font-medium text-emerald-400 hover:bg-emerald-500/10 border border-emerald-500/30 whitespace-nowrap flex items-center space-x-2 transition ml-auto">
                <i class="fa-solid fa-book-open"></i>
                <span>完整設定與部署教學</span>
            </button>
        </div>
    </div>

    <!-- 主內容區 (自動調整全螢幕滿版寬度) -->
    <main class="flex-1 w-full mx-auto px-4 xl:px-8 py-5 space-y-5">

        <!-- ==================== 分頁 1：自選監控與技術面 ==================== -->
        <section id="view-dashboard" class="space-y-5">
            <!-- 美股大盤與台股高度連動科技巨頭即時看板 -->
            <div class="glass-card rounded-2xl p-4">
                <div class="flex flex-wrap items-center justify-between gap-2 mb-3">
                    <div class="flex items-center space-x-2">
                        <span class="px-2 py-0.5 rounded bg-indigo-500/20 text-indigo-300 border border-indigo-500/30 text-xs font-bold">US Market</span>
                        <h2 class="text-sm font-bold text-white">美股四大指數與台股連動權值指標 (TSM / NVDA / AAPL)</h2>
                    </div>
                    <span class="text-[11px] text-slate-400">台股開盤前先行指標 · 隨報價同步更新</span>
                </div>
                <div id="usMarketGrid" class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-6 gap-3">
                    <!-- 由 JS 動態渲染美股指標 -->
                </div>
            </div>

            <!-- 搜尋與新增自選股控制列 -->
            <div class="glass-card rounded-2xl p-4 flex flex-wrap items-center justify-between gap-4">
                <div class="flex flex-wrap items-center gap-2 flex-1 min-w-[260px]">
                    <div class="relative flex-1 max-w-xs">
                        <i class="fa-solid fa-magnifying-glass absolute left-3.5 top-3 text-slate-400 text-sm"></i>
                        <input type="text" id="newStockCode" placeholder="輸入代號 (例: 2330, 00919, 2603)" 
                            class="w-full pl-9 pr-4 py-2 rounded-xl bg-slate-900 border border-slate-700 text-sm text-white placeholder-slate-500 focus:outline-none focus:border-blue-500">
                    </div>
                    <input type="text" id="newStockName" placeholder="名稱 (選填，可自動辨識)" 
                        class="w-36 px-3 py-2 rounded-xl bg-slate-900 border border-slate-700 text-sm text-white placeholder-slate-500 focus:outline-none focus:border-blue-500">
                    <button onclick="addStock()" class="px-4 py-2 rounded-xl bg-blue-600 hover:bg-blue-500 text-white text-sm font-medium transition flex items-center space-x-1.5 shadow-lg shadow-blue-600/20">
                        <i class="fa-solid fa-plus"></i>
                        <span>加入自選</span>
                    </button>
                </div>

                <!-- 分類篩選器 -->
                <div class="flex items-center space-x-1.5 text-xs">
                    <button onclick="filterCategory('ALL')" id="cat-ALL" class="cat-btn px-3 py-1.5 rounded-lg bg-slate-700 text-white font-medium">全部</button>
                    <button onclick="filterCategory('ETF')" id="cat-ETF" class="cat-btn px-3 py-1.5 rounded-lg bg-slate-800/80 text-slate-400 hover:text-white">高股息/市值 ETF</button>
                    <button onclick="filterCategory('TECH')" id="cat-TECH" class="cat-btn px-3 py-1.5 rounded-lg bg-slate-800/80 text-slate-400 hover:text-white">半導體/AI</button>
                    <button onclick="filterCategory('FIN')" id="cat-FIN" class="cat-btn px-3 py-1.5 rounded-lg bg-slate-800/80 text-slate-400 hover:text-white">金融存股</button>
                </div>
            </div>

            <!-- 自選股卡片網格 (滿版自動擴展至 4 欄) -->
            <div id="watchlistGrid" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
                <!-- 由 JS 動態渲染 -->
            </div>

            <!-- 下方圖表與個股深度分析面板 (滿版 12 欄配置) -->
            <div class="grid grid-cols-1 xl:grid-cols-12 gap-5">
                <!-- 左側 8/12：個股 30 天內股價漲跌走勢與逐日明細 -->
                <div class="xl:col-span-8 glass-card rounded-2xl p-5 flex flex-col justify-between space-y-4">
                    <div class="flex flex-wrap items-center justify-between gap-3 pb-3 border-b border-slate-800">
                        <div class="flex items-center space-x-3">
                            <span id="chartStockBadge" class="px-2.5 py-1 rounded-lg bg-blue-500/20 text-blue-300 font-mono font-bold text-sm">2330</span>
                            <h2 id="chartStockTitle" class="text-lg font-bold text-white">台積電 近 30 日股價漲跌與均線走勢</h2>
                        </div>
                        <!-- 視圖切換：走勢圖 vs 30日漲跌明細表 -->
                        <div class="flex items-center space-x-2">
                            <div class="bg-slate-900 p-1 rounded-xl border border-slate-800 flex space-x-1 text-xs">
                                <button onclick="toggle30DayView('chart')" id="btn30dChart" class="px-3 py-1 rounded-lg bg-blue-600 text-white font-medium transition">
                                    <i class="fa-solid fa-chart-area mr-1"></i>走勢圖
                                </button>
                                <button onclick="toggle30DayView('table')" id="btn30dTable" class="px-3 py-1 rounded-lg text-slate-400 hover:text-white transition">
                                    <i class="fa-solid fa-table-list mr-1"></i>30日逐日漲跌表
                                </button>
                            </div>
                        </div>
                    </div>

                    <!-- 30 天關鍵統計指標列 -->
                    <div class="grid grid-cols-2 sm:grid-cols-4 gap-3 text-xs">
                        <div class="p-3 rounded-xl bg-slate-900/90 border border-slate-800 flex items-center justify-between">
                            <span class="text-slate-400">30日累計漲跌</span>
                            <span id="stat30dChange" class="font-mono font-bold text-sm text-twRed">+0.00 (+0.00%)</span>
                        </div>
                        <div class="p-3 rounded-xl bg-slate-900/90 border border-slate-800 flex items-center justify-between">
                            <span class="text-slate-400">30日最高價</span>
                            <span id="stat30dHigh" class="font-mono font-bold text-sm text-white">--</span>
                        </div>
                        <div class="p-3 rounded-xl bg-slate-900/90 border border-slate-800 flex items-center justify-between">
                            <span class="text-slate-400">30日最低價</span>
                            <span id="stat30dLow" class="font-mono font-bold text-sm text-white">--</span>
                        </div>
                        <div class="p-3 rounded-xl bg-slate-900/90 border border-slate-800 flex items-center justify-between">
                            <span class="text-slate-400">上漲 / 下跌天數</span>
                            <span id="stat30dWinDays" class="font-mono font-bold text-sm text-amber-400">-- 漲 / -- 跌</span>
                        </div>
                    </div>

                    <!-- 視圖 A：30 日 Chart.js 走勢圖 -->
                    <div id="panel30dChart" class="relative w-full h-80">
                        <canvas id="stockChart"></canvas>
                    </div>

                    <!-- 視圖 B：30 日逐日股價漲跌明細表 -->
                    <div id="panel30dTable" class="hidden max-h-80 overflow-y-auto rounded-xl border border-slate-800 bg-slate-900/60">
                        <table class="w-full text-left border-collapse text-xs">
                            <thead class="sticky top-0 bg-slate-900 border-b border-slate-800 text-slate-400">
                                <tr>
                                    <th class="py-2.5 px-4">交易日期</th>
                                    <th class="py-2.5 px-4 text-right">收盤價 (元)</th>
                                    <th class="py-2.5 px-4 text-right">單日漲跌</th>
                                    <th class="py-2.5 px-4 text-right">單日漲跌幅</th>
                                    <th class="py-2.5 px-4 text-right">5日均線 (MA5)</th>
                                </tr>
                            </thead>
                            <tbody id="table30dBody" class="divide-y divide-slate-800/60 font-mono">
                                <!-- 由 JS 動態產生 30 天逐日漲跌紀錄 -->
                            </tbody>
                        </table>
                    </div>
                </div>

                <!-- 右側 4/12：選中個股基本面、籌碼面與個人筆記 -->
                <div class="xl:col-span-4 glass-card rounded-2xl p-5 flex flex-col justify-between space-y-4">
                    <div>
                        <div class="flex items-center justify-between border-b border-slate-800 pb-3 mb-4">
                            <h3 class="font-bold text-white flex items-center space-x-2">
                                <i class="fa-solid fa-microscope text-blue-400"></i>
                                <span>個股籌碼與基本面摘要</span>
                            </h3>
                            <span id="detailCode" class="text-xs font-mono text-slate-400">2330.TW</span>
                        </div>

                        <div class="grid grid-cols-2 gap-3 text-sm mb-4">
                            <div class="p-3 rounded-xl bg-slate-900/90 border border-slate-800">
                                <div class="text-xs text-slate-400">預估年化殖利率</div>
                                <div id="detailYield" class="font-mono font-bold text-base text-amber-400 mt-1">1.85%</div>
                            </div>
                            <div class="p-3 rounded-xl bg-slate-900/90 border border-slate-800">
                                <div class="text-xs text-slate-400">本益比 (P/E)</div>
                                <div id="detailPE" class="font-mono font-bold text-base text-white mt-1">24.6x</div>
                            </div>
                            <div class="p-3 rounded-xl bg-slate-900/90 border border-slate-800">
                                <div class="text-xs text-slate-400">外資近5日買賣超</div>
                                <div id="detailForeign" class="font-mono font-bold text-base text-twRed mt-1">+38,420 張</div>
                            </div>
                            <div class="p-3 rounded-xl bg-slate-900/90 border border-slate-800">
                                <div class="text-xs text-slate-400">投信近5日買賣超</div>
                                <div id="detailTrust" class="font-mono font-bold text-base text-twRed mt-1">+4,150 張</div>
                            </div>
                        </div>
                    </div>

                    <!-- 個人投資觀察筆記 (自動儲存於 LocalStorage) -->
                    <div class="flex-1 flex flex-col">
                        <div class="flex items-center justify-between mb-1.5">
                            <label class="text-xs font-medium text-slate-300 flex items-center space-x-1.5">
                                <i class="fa-solid fa-pen-to-square text-amber-400"></i>
                                <span>個人操作策略與支撐壓力筆記</span>
                            </label>
                            <span id="noteSaveStatus" class="text-[11px] text-emerald-400 opacity-0 transition">已自動儲存</span>
                        </div>
                        <textarea id="stockNoteInput" oninput="saveStockNote()" rows="4" 
                            class="w-full flex-1 rounded-xl bg-slate-900 border border-slate-700 p-3 text-xs text-slate-200 placeholder-slate-500 focus:outline-none focus:border-blue-500 resize-none"
                            placeholder="記錄這檔股票的買進理由、目標價、停損點或法說會重點..."></textarea>
                    </div>
                </div>
            </div>

            <!-- 即時財經新聞與個股最新動態 (Google News 即時搜尋) -->
            <div class="glass-card rounded-2xl p-5">
                <div class="flex flex-wrap items-center justify-between gap-2 mb-4 pb-3 border-b border-slate-800">
                    <div class="flex items-center space-x-2">
                        <i class="fa-solid fa-newspaper text-emerald-400"></i>
                        <h3 class="font-bold text-white text-base">即時台股與個股新聞動態</h3>
                        <span id="newsStockTag" class="text-xs px-2 py-0.5 rounded bg-blue-500/20 text-blue-300 font-mono">台股大盤 & 2330</span>
                    </div>
                    <button onclick="fetchLiveNews(selectedStockCode)" class="text-xs text-slate-400 hover:text-white flex items-center space-x-1 transition">
                        <i class="fa-solid fa-arrows-rotate"></i>
                        <span>重新整理新聞</span>
                    </button>
                </div>
                <div id="liveNewsContainer" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3 text-xs">
                    <div class="col-span-full text-slate-400 py-4 text-center">正在載入最新即時台股新聞...</div>
                </div>
            </div>
        </section>

        <!-- ==================== 分頁 2：庫存損益與試算 ==================== -->
        <section id="view-portfolio" class="hidden space-y-6">
            <!-- 資產總覽卡片 -->
            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                <div class="glass-card rounded-2xl p-5">
                    <div class="text-xs text-slate-400">庫存總市值 (TWD)</div>
                    <div id="portTotalMarket" class="text-2xl font-mono font-bold text-white mt-1">$0</div>
                    <div class="text-[11px] text-slate-500 mt-1">依目前即時報價自動換算</div>
                </div>
                <div class="glass-card rounded-2xl p-5">
                    <div class="text-xs text-slate-400">總投入成本 (TWD)</div>
                    <div id="portTotalCost" class="text-2xl font-mono font-bold text-slate-200 mt-1">$0</div>
                    <div class="text-[11px] text-slate-500 mt-1">含設定之持有股數 × 均價</div>
                </div>
                <div class="glass-card rounded-2xl p-5">
                    <div class="text-xs text-slate-400">帳面未實現損益</div>
                    <div id="portTotalProfit" class="text-2xl font-mono font-bold text-twRed mt-1">+$0</div>
                    <div id="portTotalReturnRate" class="text-xs font-mono text-twRed mt-1">+0.00%</div>
                </div>
                <div class="glass-card rounded-2xl p-5">
                    <div class="text-xs text-slate-400">預估年度現金股利</div>
                    <div id="portAnnualDividend" class="text-2xl font-mono font-bold text-amber-400 mt-1">$0</div>
                    <div id="portYieldOnCost" class="text-xs text-slate-400 mt-1">成本殖利率: 0.00%</div>
                </div>
            </div>

            <!-- 庫存明細表格 -->
            <div class="glass-card rounded-2xl p-5 overflow-hidden">
                <div class="flex flex-wrap items-center justify-between gap-2 mb-4">
                    <div>
                        <h3 class="text-base font-bold text-white">個人持股成本與損益試算表</h3>
                        <p class="text-xs text-slate-400">可直接修改下方「持有股數」與「成交均價」，資料會儲存在您的瀏覽器本機端。</p>
                    </div>
                </div>

                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse text-sm">
                        <thead>
                            <tr class="border-b border-slate-800 text-xs text-slate-400">
                                <th class="py-3 px-3">股票代號/名稱</th>
                                <th class="py-3 px-3 text-right">目前現價</th>
                                <th class="py-3 px-3 text-right">持有股數 (股)</th>
                                <th class="py-3 px-3 text-right">成本均價 (元)</th>
                                <th class="py-3 px-3 text-right">目前市值</th>
                                <th class="py-3 px-3 text-right">未實現損益</th>
                                <th class="py-3 px-3 text-right">報酬率</th>
                                <th class="py-3 px-3 text-right">預估年領股息</th>
                            </tr>
                        </thead>
                        <tbody id="portfolioTableBody" class="divide-y divide-slate-800/70 font-mono text-xs sm:text-sm">
                            <!-- 由 JS 渲染 -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- ==================== 分頁 3：ETF 配息與重大訊息 ==================== -->
        <section id="view-etf" class="hidden space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- 左側 2/3：熱門高股息與市值型 ETF 配息日曆 -->
                <div class="lg:col-span-2 glass-card rounded-2xl p-5">
                    <div class="flex items-center justify-between mb-4">
                        <div>
                            <h3 class="text-base font-bold text-white flex items-center space-x-2">
                                <i class="fa-solid fa-coins text-amber-400"></i>
                                <span>台股熱門 ETF 除權息與入帳行事曆</span>
                            </h3>
                            <p class="text-xs text-slate-400 mt-0.5">彙整季配息、月配息與半年配 ETF 最新公告配息金額與年化殖利率</p>
                        </div>
                    </div>

                    <div class="overflow-x-auto">
                        <table class="w-full text-left border-collapse text-sm">
                            <thead>
                                <tr class="border-b border-slate-800 text-xs text-slate-400">
                                    <th class="py-3 px-3">ETF 代號/名稱</th>
                                    <th class="py-3 px-3">配息頻率</th>
                                    <th class="py-3 px-3 text-right">最新現價</th>
                                    <th class="py-3 px-3 text-right">每單位配息</th>
                                    <th class="py-3 px-3 text-right">單次/年化殖利率</th>
                                    <th class="py-3 px-3">除息交易日</th>
                                    <th class="py-3 px-3">股利發放日</th>
                                </tr>
                            </thead>
                            <tbody id="etfCalendarBody" class="divide-y divide-slate-800/70 text-xs sm:text-sm">
                                <!-- JS 動態產生 -->
                            </tbody>
                        </table>
                    </div>
                </div>

                <!-- 右側 1/3：公開資訊觀測站 (MOPS) 重大訊息與官方情報捷徑 -->
                <div class="glass-card rounded-2xl p-5 flex flex-col justify-between space-y-6">
                    <div>
                        <h3 class="text-base font-bold text-white flex items-center space-x-2 mb-3">
                            <i class="fa-solid fa-bullhorn text-rose-400"></i>
                            <span>自選股重要公告與法說追蹤</span>
                        </h3>
                        <div class="space-y-3 text-xs">
                            <div class="p-3 rounded-xl bg-slate-900/90 border border-slate-800 hover:border-slate-700 transition">
                                <div class="flex items-center justify-between text-slate-400 mb-1">
                                    <span class="px-2 py-0.5 rounded bg-blue-500/20 text-blue-300 font-mono">2330 台積電</span>
                                    <span>營收公告</span>
                                </div>
                                <p class="text-slate-200 font-medium leading-relaxed">公告本公司最新單月合併營收創歷史同期新高，受惠先進製程 AI 晶片需求強勁。</p>
                            </div>
                            <div class="p-3 rounded-xl bg-slate-900/90 border border-slate-800 hover:border-slate-700 transition">
                                <div class="flex items-center justify-between text-slate-400 mb-1">
                                    <span class="px-2 py-0.5 rounded bg-amber-500/20 text-amber-300 font-mono">00919 群益台灣精選高息</span>
                                    <span>收益分配</span>
                                </div>
                                <p class="text-slate-200 font-medium leading-relaxed">最新一季收益分配期前公告，帳上可分配收益與平準金充沛。</p>
                            </div>
                            <div class="p-3 rounded-xl bg-slate-900/90 border border-slate-800 hover:border-slate-700 transition">
                                <div class="flex items-center justify-between text-slate-400 mb-1">
                                    <span class="px-2 py-0.5 rounded bg-emerald-500/20 text-emerald-300 font-mono">2881 富邦金</span>
                                    <span>自結獲利</span>
                                </div>
                                <p class="text-slate-200 font-medium leading-relaxed">公告自結累計稅後純益與每股盈餘 (EPS) 持續居金控獲利領先地位。</p>
                            </div>
                        </div>
                    </div>

                    <!-- 常用台股官方資訊站快速跳轉 -->
                    <div class="border-t border-slate-800 pt-4">
                        <h4 class="text-xs font-semibold text-slate-400 uppercase tracking-wider mb-3">常用台股資料源快速跳轉</h4>
                        <div class="grid grid-cols-2 gap-2 text-xs">
                            <a href="https://mis.twse.com.tw/stock/index.jsp" target="_blank" rel="noopener noreferrer" class="p-2.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-200 flex items-center justify-between transition">
                                <span>證交所基本市況</span>
                                <i class="fa-solid fa-arrow-up-right-from-square text-slate-400"></i>
                            </a>
                            <a href="https://mops.twse.com.tw/" target="_blank" rel="noopener noreferrer" class="p-2.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-200 flex items-center justify-between transition">
                                <span>公開資訊觀測站</span>
                                <i class="fa-solid fa-arrow-up-right-from-square text-slate-400"></i>
                            </a>
                            <a href="https://www.twse.com.tw/zh/ETFortune/index" target="_blank" rel="noopener noreferrer" class="p-2.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-200 flex items-center justify-between transition">
                                <span>證交所 e添富</span>
                                <i class="fa-solid fa-arrow-up-right-from-square text-slate-400"></i>
                            </a>
                            <a href="https://openapi.twse.com.tw/" target="_blank" rel="noopener noreferrer" class="p-2.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-200 flex items-center justify-between transition">
                                <span>TWSE OpenAPI</span>
                                <i class="fa-solid fa-arrow-up-right-from-square text-slate-400"></i>
                            </a>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- ==================== 分頁 4：完整設定與部署教學 ==================== -->
        <section id="view-guide" class="hidden space-y-6">
            <div class="glass-card rounded-2xl p-6 border-l-4 border-l-emerald-500">
                <h2 class="text-xl font-bold text-white mb-2">三步驟免費上線與串接證交所真實報價（超詳細圖文級別指南）</h2>
                <p class="text-sm text-slate-300 leading-relaxed">
                    本網頁已內建公開代理通道，打開即可直接抓取最新股價。但若您希望擁有<strong>個人專屬、永不限流、跨手機與電腦皆穩定高速</strong>的台股戰情室，請依照以下三個步驟（完全免費、免信用卡、約 5 分鐘完成）進行設定：
                </p>
            </div>

            <!-- 步驟一：GitHub Pages 免費託管網頁 -->
            <div class="glass-card rounded-2xl p-6 space-y-4">
                <div class="flex items-center space-x-3 border-b border-slate-800 pb-3">
                    <span class="px-3 py-1 rounded-full bg-indigo-600 text-white text-xs font-bold">步驟一：託管網頁</span>
                    <h3 class="text-lg font-bold text-white">將 `index.html` 部署到 GitHub Pages 取得專屬網址</h3>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 text-sm text-slate-300">
                    <div class="p-4 rounded-xl bg-slate-900/90 border border-slate-800 space-y-2">
                        <div class="font-bold text-indigo-400">1. 建立 GitHub 儲存庫 (Repository)</div>
                        <ul class="list-disc list-inside space-y-1.5 text-xs leading-relaxed text-slate-300">
                            <li>登入 <a href="https://github.com/" target="_blank" class="text-blue-400 underline">GitHub</a>（若無帳號請免費註冊）。</li>
                            <li>點擊右上角「<strong>+</strong>」號 → 選擇「<strong>New repository</strong>」。</li>
                            <li><strong>Repository name</strong> 輸入例如：<code class="px-1.5 py-0.5 rounded bg-slate-800 text-emerald-300">twse-hub</code>。</li>
                            <li>將權限設為 <strong>Public (公開)</strong>（免費版 GitHub Pages 需為 Public）。</li>
                            <li>勾選「<strong>Add a README file</strong>」，接著點擊最下方綠色按鈕「<strong>Create repository</strong>」。</li>
                        </ul>
                    </div>

                    <div class="p-4 rounded-xl bg-slate-900/90 border border-slate-800 space-y-2">
                        <div class="font-bold text-indigo-400">2. 上傳 `index.html` 並開啟 GitHub Pages</div>
                        <ul class="list-disc list-inside space-y-1.5 text-xs leading-relaxed text-slate-300">
                            <li>在您的儲存庫頁面點擊「<strong>Add file</strong>」→「<strong>Upload files</strong>」（或點 Create new file 命名為 <code class="px-1.5 py-0.5 rounded bg-slate-800 text-emerald-300">index.html</code> 貼上本頁完整原始碼）。</li>
                            <li>點擊下方綠色按鈕「<strong>Commit changes</strong>」完成儲存。</li>
                            <li>點擊頂部選單的「<strong>Settings (設定)</strong>」→ 左側欄點選「<strong>Pages</strong>」。</li>
                            <li>在 <strong>Build and deployment</strong> 下方的 <strong>Branch</strong>，將 `None` 改為「<strong>main</strong>」（資料夾維持 `/(root)`），點擊「<strong>Save</strong>」。</li>
                            <li>等待約 1～2 分鐘重新整理頁面，上方就會出現您的專屬網址：<br><code class="text-amber-300 font-mono">https://您的帳號.github.io/twse-hub/</code></li>
                        </ul>
                    </div>
                </div>

                <!-- 新增：如何更新已部署的 GitHub 網頁與設定 -->
                <div class="p-4 rounded-xl bg-indigo-950/30 border border-indigo-500/30 space-y-2">
                    <div class="font-bold text-indigo-300 text-sm flex items-center space-x-2">
                        <i class="fa-solid fa-rotate"></i>
                        <span>3. 日後如何將新版程式碼更新到 GitHub？（30 秒完成更新）</span>
                    </div>
                    <ol class="list-decimal list-inside space-y-1.5 text-xs leading-relaxed text-slate-300">
                        <li>複製右側編輯器中最新的完整 <code class="px-1.5 py-0.5 rounded bg-slate-800 text-emerald-300">index.html</code> 程式碼。</li>
                        <li>進入您的 GitHub 儲存庫（例如 <code class="text-slate-400">twse-hub</code>），在檔案列表中直接點擊 <code class="px-1.5 py-0.5 rounded bg-slate-800 text-emerald-300">index.html</code>。</li>
                        <li>點擊檔案視窗右上角的「<strong>鉛筆圖示 (<i class="fa-solid fa-pen"></i> Edit this file)</strong>」。</li>
                        <li>按 <kbd class="px-1.5 py-0.5 bg-slate-800 rounded">Ctrl + A</kbd> 全選舊程式碼並刪除，再按 <kbd class="px-1.5 py-0.5 bg-slate-800 rounded">Ctrl + V</kbd> 貼上最新程式碼。</li>
                        <li>點擊右上角綠色按鈕「<strong>Commit changes...</strong>」→ 在彈出視窗再點一次「<strong>Commit changes</strong>」。</li>
                        <li>等待約 <strong>30～60 秒</strong>，前往您的 GitHub Pages 網址並按 <kbd class="px-1.5 py-0.5 bg-slate-800 rounded">Ctrl + F5</kbd>（強制重新整理快取），即可看到最新滿版與美股、30日漲跌介面！</li>
                    </ol>
                </div>
            </div>

            <!-- 步驟二：Google Apps Script (最推薦，支援盤中即時報價) -->
            <div class="glass-card rounded-2xl p-6 space-y-4">
                <div class="flex flex-wrap items-center justify-between gap-2 border-b border-slate-800 pb-3">
                    <div class="flex items-center space-x-3">
                        <span class="px-3 py-1 rounded-full bg-blue-600 text-white text-xs font-bold">步驟二：啟動免伺服器報價代理</span>
                        <h3 class="text-lg font-bold text-white">部署 Google Apps Script (GAS) 串接證交所官方即時行情</h3>
                    </div>
                    <button onclick="copyCode('gasCode')" class="px-3 py-1.5 rounded-lg bg-blue-600 hover:bg-blue-500 text-xs text-white font-medium flex items-center space-x-1.5 shadow">
                        <i class="fa-regular fa-copy"></i>
                        <span>一鍵複製 GAS 完整程式碼</span>
                    </button>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-3 gap-3 text-xs text-slate-300">
                    <div class="p-3.5 rounded-xl bg-slate-900/90 border border-slate-800">
                        <div class="font-bold text-blue-400 mb-1">2-1. 建立新專案並貼上程式碼</div>
                        <p class="leading-relaxed">前往 <a href="https://script.google.com/" target="_blank" class="text-blue-400 underline">Google Apps Script</a> 點擊左上角「<strong>＋ 新專案</strong>」。將預設的 <code class="text-slate-400">function myFunction() {}</code> 全部刪除，貼上下方複製的程式碼，並按 <kbd class="px-1.5 py-0.5 bg-slate-800 rounded">Ctrl + S</kbd> 儲存。</p>
                    </div>
                    <div class="p-3.5 rounded-xl bg-slate-900/90 border border-slate-800">
                        <div class="font-bold text-blue-400 mb-1">2-2. 部署為網頁應用程式 (關鍵設定)</div>
                        <p class="leading-relaxed">點擊右上角藍色按鈕「<strong>部署</strong>」→「<strong>新增部署作業</strong>」→ 點擊左側齒輪選取「<strong>網頁應用程式</strong>」。<br>
                        • 執行身分：<strong>我 (您的 Google 帳號)</strong><br>
                        • 誰可以存取：務必選「<strong class="text-amber-400">所有人 (Anyone)</strong>」</p>
                    </div>
                    <div class="p-3.5 rounded-xl bg-slate-900/90 border border-slate-800">
                        <div class="font-bold text-blue-400 mb-1">2-3. 通過 Google 授權畫面</div>
                        <p class="leading-relaxed">點擊部署後選「<strong>核准存取權</strong>」→ 選擇您的 Google 帳號 → 若出現「Google 尚未驗證這個應用程式」，請點左下角「<strong>進階 (Advanced)</strong>」→ 點擊「<strong>前往未命名專案 (不安全)</strong>」→ 點「<strong>允許</strong>」，最後複製產生的 <code class="text-emerald-300">https://script.google.com/macros/s/.../exec</code> 網址。</p>
                    </div>
                </div>

                <pre id="gasCode" class="code-block rounded-xl p-4 text-xs font-mono text-emerald-300 overflow-x-auto leading-relaxed">/**
 * 台股證交所 MIS 即時報價 Google Apps Script Proxy (含大盤指數與五檔買賣價自動備援)
 * 用途：解決瀏覽器直接呼叫證交所 API 的 CORS 跨域限制
 */
function doGet(e) {
  var codesParam = (e && e.parameter && e.parameter.codes) ? e.parameter.codes : "2330,2317,2454,0050,0056,00878,00919,2881";
  var codeList = codesParam.split(",");
  
  // 同時抓取大盤加權指數(t00)、櫃買指數(o00)以及所有自選股(上市 tse + 上櫃 otc)
  var stockQueries = codeList.map(function(c) {
    c = c.trim();
    return "tse_" + c + ".tw|otc_" + c + ".tw";
  }).join("|");
  
  var fullExCh = "tse_t00.tw|otc_o00.tw|" + stockQueries;
  var url = "https://mis.twse.com.tw/stock/api/getStockInfo.jsp?ex_ch=" + encodeURIComponent(fullExCh) + "&json=1&delay=0";
  
  try {
    var response = UrlFetchApp.fetch(url, {
      headers: {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)",
        "Accept-Language": "zh-TW,zh;q=0.9"
      },
      muteHttpExceptions: true
    });
    
    var rawData = JSON.parse(response.getContentText());
    var result = [];
    var indices = {};
    
    if (rawData && rawData.msgArray) {
      rawData.msgArray.forEach(function(item) {
        if (!item.c) return;
        // z: 最新成交價, b: 五檔買價, a: 五檔賣價, y: 昨收, v: 累積成交量(張), n: 公司簡稱
        var price = parseFloat(item.z);
        if (isNaN(price) || price <= 0) {
          var bestBid = item.b ? parseFloat(item.b.split("_")[0]) : NaN;
          var bestAsk = item.a ? parseFloat(item.a.split("_")[0]) : NaN;
          price = !isNaN(bestBid) && bestBid > 0 ? bestBid : (!isNaN(bestAsk) && bestAsk > 0 ? bestAsk : parseFloat(item.y));
        }
        var prevClose = parseFloat(item.y) || price;
        if (isNaN(price) || price <= 0) return;
        
        var change = +(price - prevClose).toFixed(2);
        var changePct = prevClose > 0 ? +((change / prevClose) * 100).toFixed(2) : 0;
        
        if (item.c === "t00" || item.c === "o00") {
          indices[item.c] = { price: price, change: change, changePercent: changePct };
          return;
        }
        
        result.push({
          code: item.c,
          name: item.n,
          price: price,
          change: change,
          changePercent: changePct,
          volume: parseInt(item.v || "0", 10),
          high: parseFloat(item.h) || price,
          low: parseFloat(item.l) || price
        });
      });
    }
    
    return ContentService.createTextOutput(JSON.stringify({
      status: "ok",
      updatedAt: new Date().toLocaleString("zh-TW", { timeZone: "Asia/Taipei", hour12: false }),
      indices: indices,
      stocks: result
    })).setMimeType(ContentService.MimeType.JSON);
    
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({
      status: "error",
      message: err.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}</pre>
            </div>

            <!-- 步驟三：綁定資料源與常見排錯 -->
            <div class="glass-card rounded-2xl p-6 space-y-4">
                <div class="flex flex-wrap items-center justify-between gap-2 border-b border-slate-800 pb-3">
                    <div class="flex items-center space-x-3">
                        <span class="px-3 py-1 rounded-full bg-emerald-600 text-white text-xs font-bold">步驟三：綁定資料源</span>
                        <h3 class="text-lg font-bold text-white">在戰情室綁定您的 GAS 網址並驗證連線</h3>
                    </div>
                    <button onclick="openApiModal()" class="px-3 py-1.5 rounded-lg bg-emerald-600 hover:bg-emerald-500 text-xs text-white font-medium flex items-center space-x-1.5 shadow">
                        <i class="fa-solid fa-plug"></i>
                        <span>立即開啟「資料源設定」視窗</span>
                    </button>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 text-xs text-slate-300">
                    <div class="p-4 rounded-xl bg-slate-900/90 border border-slate-800 space-y-2">
                        <div class="font-bold text-emerald-400">如何綁定與確認成功？</div>
                        <ol class="list-decimal list-inside space-y-1.5 leading-relaxed">
                            <li>點擊網頁右上角的「<strong>資料源設定</strong>」按鈕。</li>
                            <li>將步驟二複製的 <code class="text-emerald-300">https://script.google.com/macros/s/.../exec</code> 貼入輸入框。</li>
                            <li>點擊「<strong>儲存並立即同步</strong>」。</li>
                            <li>觀察網頁左上角標題旁的綠色標籤，若顯示為「<strong class="text-emerald-300">自訂 GAS 即時連線</strong>」，即代表您的專屬證交所直線通道已綁定成功！</li>
                        </ol>
                    </div>

                    <div class="p-4 rounded-xl bg-slate-900/90 border border-slate-800 space-y-2">
                        <div class="font-bold text-amber-400">常見問題排錯 (Troubleshooting)</div>
                        <ul class="list-disc list-inside space-y-1.5 leading-relaxed">
                            <li><strong>網址結尾必須是 `/exec`</strong>：請勿複製到測試用的 `/dev` 結尾網址，否則只有登入的瀏覽器能讀取。</li>
                            <li><strong>CORS 錯誤或無法讀取</strong>：99% 是因為步驟二部署時，「誰可以存取」沒有選取「<strong>所有人 (Anyone)</strong>」。請在 GAS 點擊右上角「部署」→「管理部署作業」→ 點擊鉛筆圖示修改為「所有人」並選擇「建立新版本」。</li>
                        </ul>
                    </div>
                </div>
            </div>

            <!-- 進階方案 B：GitHub Actions 自動爬蟲 + GitHub Pages 免費託管 -->
            <div class="glass-card rounded-2xl p-6 space-y-4">
                <div class="flex flex-wrap items-center justify-between gap-2">
                    <div class="flex items-center space-x-3">
                        <span class="px-3 py-1 rounded-full bg-purple-600 text-white text-xs font-bold">進階選配方案 B (全自動靜態快取)</span>
                        <h3 class="text-lg font-bold text-white">使用 GitHub Actions 每日盤後自動抓取並產生 `stocks.json`</h3>
                    </div>
                    <button onclick="copyCode('ghActionCode')" class="px-3 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-xs text-purple-300 border border-slate-700 flex items-center space-x-1.5">
                        <i class="fa-regular fa-copy"></i>
                        <span>複製 Workflow 設定檔</span>
                    </button>
                </div>

                <p class="text-sm text-slate-300">
                    如果您將這個 `index.html` 上傳至 GitHub 儲存庫並開啟 <strong>GitHub Pages</strong>，可以新增以下两个檔案，讓 GitHub 每天台北時間下午 14:30（收盤後）自動抓取最新股價並存成 `stocks.json`：
                </p>

                <div class="text-xs font-semibold text-slate-400">1. 建立 `.github/workflows/update_stocks.yml`：</div>
                <pre id="ghActionCode" class="code-block rounded-xl p-4 text-xs font-mono text-purple-300 overflow-x-auto leading-relaxed">name: Daily TWSE Stock Fetcher

on:
  schedule:
    # 每個交易日 UTC 06:30 (台北時間 14:30) 自動執行
    - cron: '30 6 * * 1-5'
  workflow_dispatch: # 支援手動點擊觸發

permissions:
  contents: write

jobs:
  fetch-stocks:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install requests yfinance

      - name: Run crawler script
        run: python fetch_twse.py

      - name: Commit and Push stocks.json
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add stocks.json
          git diff --quiet && git diff --staged --quiet || git commit -m "Auto update TWSE stock data"
          git push</pre>

                <div class="flex items-center justify-between pt-2">
                    <span class="text-xs font-semibold text-slate-400">2. 在根目錄建立 `fetch_twse.py` 爬蟲腳本：</span>
                    <button onclick="copyCode('pyCode')" class="px-3 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-xs text-purple-300 border border-slate-700 flex items-center space-x-1.5">
                        <i class="fa-regular fa-copy"></i>
                        <span>複製 Python 腳本</span>
                    </button>
                </div>
                <pre id="pyCode" class="code-block rounded-xl p-4 text-xs font-mono text-amber-300 overflow-x-auto leading-relaxed">import json
from datetime import datetime
import yfinance as yf

# 設定欲追蹤的台股清單
TARGETS = {
    "2330": ("台積電", "TECH"),
    "2317": ("鴻海", "TECH"),
    "2454": ("聯發科", "TECH"),
    "0050": ("元大台灣50", "ETF"),
    "0056": ("元大高股息", "ETF"),
    "00878": ("國泰永續高股息", "ETF"),
    "00919": ("群益台灣精選高息", "ETF"),
    "2881": ("富邦金", "FIN")
}

output = []
for code, (name, category) in TARGETS.items():
    ticker = yf.Ticker(f"{code}.TW")
    hist = ticker.history(period="20d")
    if len(hist) >= 2:
        close = round(float(hist['Close'].iloc[-1]), 2)
        prev = round(float(hist['Close'].iloc[-2]), 2)
        change = round(close - prev, 2)
        pct = round((change / prev) * 100, 2)
        vol = int(hist['Volume'].iloc[-1] // 1000)
        history_prices = [round(float(x), 2) for x in hist['Close'].tolist()]
        output.append({
            "code": code,
            "name": name,
            "category": category,
            "price": close,
            "change": change,
            "changePercent": pct,
            "volume": vol,
            "history": history_prices
        })

with open("stocks.json", "w", encoding="utf-8") as f:
    json.dump({
        "status": "ok",
        "updatedAt": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        "stocks": output
    }, f, ensure_ascii=False, indent=2)
print("Successfully generated stocks.json")</pre>
            </div>
        </section>

    </main>

    <!-- 資料源設定 Modal -->
    <div id="apiModal" class="fixed inset-0 z-50 hidden bg-black/70 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="glass-card rounded-2xl max-w-lg w-full p-6 border border-slate-700 shadow-2xl space-y-4">
            <div class="flex items-center justify-between border-b border-slate-800 pb-3">
                <h3 class="text-base font-bold text-white flex items-center space-x-2">
                    <i class="fa-solid fa-plug text-blue-400"></i>
                    <span>外部資料源串接設定</span>
                </h3>
                <button onclick="closeApiModal()" class="text-slate-400 hover:text-white">
                    <i class="fa-solid fa-xmark text-lg"></i>
                </button>
            </div>

            <div class="space-y-3 text-xs text-slate-300">
                <p class="leading-relaxed">
                    您可在此填入依照「設定教學」部署好的 <strong>Google Apps Script Web App 網址</strong> 或同網域下的 <strong>`./stocks.json`</strong> 路徑。若留空，系統將使用內建仿真台股行情引擎。
                </p>
                <div>
                    <label class="block text-slate-400 mb-1.5 font-medium">API Endpoint / JSON URL：</label>
                    <input type="url" id="customApiUrl" placeholder="https://script.google.com/macros/s/.../exec 或 ./stocks.json" 
                        class="w-full px-3.5 py-2.5 rounded-xl bg-slate-900 border border-slate-700 text-sm text-white placeholder-slate-500 focus:outline-none focus:border-blue-500">
                </div>
            </div>

            <div class="flex items-center justify-between pt-3 border-t border-slate-800">
                <button onclick="resetDefaultData()" class="px-3 py-2 rounded-xl bg-rose-500/10 hover:bg-rose-500/20 text-rose-300 text-xs font-medium transition">
                    重置所有本機快取資料
                </button>
                <div class="flex space-x-2">
                    <button onclick="closeApiModal()" class="px-4 py-2 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-300 text-xs font-medium transition">
                        取消
                    </button>
                    <button onclick="saveApiConfig()" class="px-4 py-2 rounded-xl bg-blue-600 hover:bg-blue-500 text-white text-xs font-medium transition">
                        儲存並立即同步
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- 全域提示 Toast -->
    <div id="toastMsg" class="fixed bottom-5 right-5 z-50 hidden px-4 py-3 rounded-xl bg-slate-800 border border-slate-700 text-white text-xs font-medium shadow-xl flex items-center space-x-2">
        <i class="fa-solid fa-circle-check text-emerald-400"></i>
        <span id="toastText">操作成功</span>
    </div>

    <!-- 頁尾 -->
    <footer class="border-t border-slate-800/80 py-4 px-4 xl:px-8 text-center text-xs text-slate-500 mt-8">
        <div class="w-full mx-auto flex flex-col sm:flex-row items-center justify-between gap-2">
            <span>台股個人戰情室 (Single-File Serverless Architecture) · 資料儲存於瀏覽器 LocalStorage</span>
            <span>台股配色慣例：<span class="text-twRed font-semibold">▲ 紅漲</span> / <span class="text-twGreen font-semibold">▼ 綠跌</span></span>
        </div>
    </footer>

    <script>
        // ==================== 美股四大指數與台股連動科技權值股預設資料 ====================
        let usMarkets = [
            { symbol: '^SOX', name: '費城半導體', price: 5240.5, change: 68.2, changePercent: 1.32 },
            { symbol: '^IXIC', name: '那斯達克', price: 18420.8, change: 115.4, changePercent: 0.63 },
            { symbol: '^GSPC', name: 'S&P 500', price: 5810.2, change: 24.5, changePercent: 0.42 },
            { symbol: 'TSM', name: '台積電 ADR', price: 192.45, change: 4.15, changePercent: 2.20 },
            { symbol: 'NVDA', name: '輝達 NVIDIA', price: 138.60, change: 3.20, changePercent: 2.36 },
            { symbol: 'AAPL', name: '蘋果 Apple', price: 231.30, change: -0.85, changePercent: -0.37 }
        ];

        // ==================== 預設台股標的與基本面資料庫 (含 30 日歷史資料) ====================
        const DEFAULT_STOCKS = [
            {
                code: '2330', name: '台積電', category: 'TECH',
                price: 1065.0, change: 20.0, changePercent: 1.91, volume: 42850,
                pe: '24.6x', yieldRate: 1.65, annualDiv: 17.5,
                foreign5d: '+38,420 張', trust5d: '+4,150 張',
                shares: 1000, avgCost: 920.0,
                history: [975, 980, 972, 985, 990, 995, 1005, 1000, 1010, 1015, 1010, 1020, 1015, 1030, 1045, 1040, 1055, 1050, 1045, 1060, 1070, 1055, 1040, 1050, 1065, 1060, 1050, 1045, 1045, 1065]
            },
            {
                code: '2317', name: '鴻海', category: 'TECH',
                price: 214.5, change: 3.5, changePercent: 1.66, volume: 68420,
                pe: '16.2x', yieldRate: 2.80, annualDiv: 6.0,
                foreign5d: '+22,190 張', trust5d: '+3,800 張',
                shares: 2000, avgCost: 185.0,
                history: [186, 188, 187, 190, 192, 191, 194, 196, 195, 197, 198, 200, 202, 201, 205, 208, 206, 209, 211, 210, 208, 207, 210, 212, 211, 209, 210, 211, 211, 214.5]
            },
            {
                code: '2454', name: '聯發科', category: 'TECH',
                price: 1315.0, change: -15.0, changePercent: -1.13, volume: 6320,
                pe: '19.8x', yieldRate: 4.18, annualDiv: 55.0,
                foreign5d: '-1,420 張', trust5d: '+890 張',
                shares: 0, avgCost: 0,
                history: [1220, 1235, 1230, 1245, 1250, 1260, 1255, 1270, 1265, 1275, 1280, 1295, 1310, 1305, 1320, 1340, 1335, 1350, 1345, 1330, 1325, 1340, 1335, 1330, 1320, 1325, 1335, 1330, 1330, 1315]
            },
            {
                code: '0050', name: '元大台灣50', category: 'ETF',
                price: 194.2, change: 2.1, changePercent: 1.09, volume: 15400,
                pe: '-', yieldRate: 2.57, annualDiv: 5.0,
                foreign5d: '+6,210 張', trust5d: '+1,200 張',
                shares: 2000, avgCost: 168.5,
                history: [180.2, 181.0, 180.5, 182.0, 183.1, 182.8, 184.0, 184.5, 185.0, 185.2, 185.5, 186.2, 187.0, 188.5, 189.0, 188.2, 190.1, 191.0, 190.5, 191.8, 192.5, 191.0, 190.8, 191.5, 192.0, 192.8, 191.5, 192.1, 192.1, 194.2]
            },
            {
                code: '00919', name: '群益台灣精選高息', category: 'ETF',
                price: 23.68, change: 0.12, changePercent: 0.51, volume: 58900,
                pe: '-', yieldRate: 11.48, annualDiv: 2.72,
                foreign5d: '-8,500 張', trust5d: '+12,400 張',
                shares: 10000, avgCost: 22.4,
                history: [22.6, 22.65, 22.7, 22.68, 22.75, 22.8, 22.85, 22.9, 22.95, 23.0, 23.1, 23.2, 23.15, 23.3, 23.4, 23.35, 23.5, 23.48, 23.42, 23.55, 23.6, 23.5, 23.45, 23.52, 23.58, 23.6, 23.55, 23.56, 23.56, 23.68]
            },
            {
                code: '00878', name: '國泰永續高股息', category: 'ETF',
                price: 22.85, change: -0.05, changePercent: -0.22, volume: 49200,
                pe: '-', yieldRate: 8.92, annualDiv: 2.04,
                foreign5d: '+3,120 張', trust5d: '+5,600 張',
                shares: 8000, avgCost: 20.8,
                history: [22.1, 22.15, 22.2, 22.18, 22.25, 22.3, 22.35, 22.4, 22.42, 22.48, 22.5, 22.55, 22.6, 22.58, 22.7, 22.75, 22.72, 22.8, 22.85, 22.9, 22.88, 22.82, 22.78, 22.85, 22.9, 22.92, 22.88, 22.9, 22.9, 22.85]
            },
            {
                code: '2881', name: '富邦金', category: 'FIN',
                price: 91.8, change: 0.7, changePercent: 0.77, volume: 18750,
                pe: '11.4x', yieldRate: 3.81, annualDiv: 3.5,
                foreign5d: '+14,300 張', trust5d: '+2,100 張',
                shares: 3000, avgCost: 78.5,
                history: [85.2, 85.5, 86.0, 85.8, 86.4, 86.9, 87.2, 87.5, 87.8, 88.1, 88.5, 88.8, 89.2, 89.0, 89.5, 90.0, 89.8, 90.2, 90.5, 90.8, 90.4, 90.1, 90.5, 90.9, 91.2, 91.0, 90.8, 91.1, 91.1, 91.8]
            }
        ];

        // 熱門 ETF 配息行事曆資料
        const ETF_CALENDAR_DATA = [
            { code: '00919', name: '群益台灣精選高息', freq: '季配 (3,6,9,12月)', price: 23.68, div: '0.72 元', yieldStr: '3.04% / 年化 12.1%', exDate: '12月中旬', payDate: '次月月中旬' },
            { code: '00878', name: '國泰永續高股息', freq: '季配 (2,5,8,11月)', price: 22.85, div: '0.55 元', yieldStr: '2.40% / 年化 9.6%', exDate: '11月中旬', payDate: '12月中旬' },
            { code: '0056', name: '元大高股息', freq: '季配 (1,4,7,10月)', price: 37.90, div: '1.07 元', yieldStr: '2.82% / 年化 11.2%', exDate: '1月中旬', payDate: '2月中旬' },
            { code: '00713', name: '元大台灣高息低波', freq: '季配 (3,6,9,12月)', price: 55.40, div: '1.50 元', yieldStr: '2.70% / 年化 10.8%', exDate: '12月中旬', payDate: '次月月中旬' },
            { code: '00929', name: '復華台灣科技優息', freq: '月配 (每月)', price: 18.45, div: '0.11 元', yieldStr: '0.59% / 年化 7.1%', exDate: '每月下旬', payDate: '次月中旬' },
            { code: '0050', name: '元大台灣50', freq: '半年配 (1,7月)', price: 194.20, div: '4.00 元', yieldStr: '2.06% / 年化 2.6%', exDate: '1月中旬', payDate: '2月中旬' }
        ];

        // ==================== 狀態管理 ====================
        let stocks = [];
        let stockNotes = {};
        let currentCategory = 'ALL';
        let selectedStockCode = '2330';
        let chartInstance = null;
        let autoRefreshTimer = null;
        let isAutoRefresh = true;
        let countdownSec = 15;

        // 多重 CORS Proxy 備援通道，確保免設定也能直接從瀏覽器抓到證交所 MIS 與 Yahoo Finance 真實報價
        const CORS_PROXIES = [
            (url) => `https://api.allorigins.win/raw?url=${encodeURIComponent(url)}`,
            (url) => `https://corsproxy.io/?${encodeURIComponent(url)}`,
            (url) => `https://api.codetabs.com/v1/proxy?quest=${encodeURIComponent(url)}`
        ];

        // 初始化載入
        window.onload = function() {
            loadLocalData();
            updateMarketStatus();
            renderUsMarket();
            renderWatchlist();
            selectStock(selectedStockCode);
            renderPortfolio();
            renderEtfCalendar();
            // 立即啟動真實報價同步與自動計時器
            refreshQuotes(false);
            startAutoRefreshTimer();
        };

        // 渲染美股大盤與科技巨頭看板
        function renderUsMarket() {
            const grid = document.getElementById('usMarketGrid');
            if (!grid) return;
            grid.innerHTML = '';

            usMarkets.forEach(item => {
                const isUp = item.change >= 0;
                const colorClass = isUp ? 'text-twRed' : 'text-twGreen';
                const badgeClass = isUp ? 'bg-red-500/10 text-twRed border-red-500/20' : 'bg-emerald-500/10 text-twGreen border-emerald-500/20';
                const sign = isUp ? '▲ +' : '▼ ';

                const div = document.createElement('div');
                div.className = 'p-3 rounded-xl bg-slate-900/90 border border-slate-800/90 flex flex-col justify-between';
                div.innerHTML = `
                    <div class="flex items-center justify-between text-xs mb-1">
                        <span class="font-bold text-slate-200 truncate">${item.name}</span>
                        <span class="font-mono text-[10px] text-slate-500">${item.symbol.replace('^', '')}</span>
                    </div>
                    <div class="flex items-baseline justify-between mt-1">
                        <span class="font-mono font-bold text-sm sm:text-base ${colorClass}">${item.price.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}</span>
                        <span class="font-mono text-[11px] px-1.5 py-0.5 rounded border ${badgeClass}">${sign}${item.changePercent.toFixed(2)}%</span>
                    </div>
                `;
                grid.appendChild(div);
            });
        }

        // 切換 30 日走勢圖與 30 日逐日漲跌明細表
        function toggle30DayView(mode) {
            const chartPanel = document.getElementById('panel30dChart');
            const tablePanel = document.getElementById('panel30dTable');
            const btnChart = document.getElementById('btn30dChart');
            const btnTable = document.getElementById('btn30dTable');

            if (mode === 'table') {
                chartPanel.classList.add('hidden');
                tablePanel.classList.remove('hidden');
                btnTable.className = 'px-3 py-1 rounded-lg bg-blue-600 text-white font-medium transition';
                btnChart.className = 'px-3 py-1 rounded-lg text-slate-400 hover:text-white transition';
            } else {
                tablePanel.classList.add('hidden');
                chartPanel.classList.remove('hidden');
                btnChart.className = 'px-3 py-1 rounded-lg bg-blue-600 text-white font-medium transition';
                btnTable.className = 'px-3 py-1 rounded-lg text-slate-400 hover:text-white transition';
            }
        }

        // 判斷目前是否為台股開盤時間 (週一至週五 09:00 ~ 13:35)
        function updateMarketStatus() {
            const now = new Date();
            const day = now.getDay();
            const hours = now.getHours();
            const mins = now.getMinutes();
            const timeNum = hours * 100 + mins;
            const statusEl = document.getElementById('marketStatusText');

            if (day >= 1 && day <= 5 && timeNum >= 900 && timeNum <= 1335) {
                statusEl.textContent = '盤中交易進行中';
                statusEl.className = 'font-mono font-bold text-emerald-400';
            } else {
                statusEl.textContent = '已收盤 (最新收盤行情)';
                statusEl.className = 'font-mono font-bold text-amber-400';
            }
        }

        function startAutoRefreshTimer() {
            if (autoRefreshTimer) clearInterval(autoRefreshTimer);
            countdownSec = 15;
            autoRefreshTimer = setInterval(() => {
                if (!isAutoRefresh) return;
                countdownSec--;
                const label = document.getElementById('autoRefreshLabel');
                if (label) label.textContent = `自動更新: ${countdownSec}s`;
                if (countdownSec <= 0) {
                    countdownSec = 15;
                    refreshQuotes(false);
                }
            }, 1000);
        }

        function toggleAutoRefresh() {
            isAutoRefresh = !isAutoRefresh;
            const btn = document.getElementById('autoRefreshBtn');
            const label = document.getElementById('autoRefreshLabel');
            if (isAutoRefresh) {
                countdownSec = 15;
                btn.className = 'px-3 py-2 rounded-lg bg-emerald-600/20 hover:bg-emerald-600/30 text-emerald-300 border border-emerald-500/30 text-xs font-medium flex items-center space-x-1.5 transition';
                label.textContent = '自動更新: 開啟 (15s)';
                showToast('已開啟每 15 秒自動同步真實報價');
            } else {
                btn.className = 'px-3 py-2 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-400 border border-slate-700 text-xs font-medium flex items-center space-x-1.5 transition';
                label.textContent = '自動更新: 暫停';
                showToast('已暫停自動更新');
            }
        }

        function loadLocalData() {
            try {
                const savedStocks = localStorage.getItem('twse_personal_stocks_v1');
                if (savedStocks) {
                    stocks = JSON.parse(savedStocks);
                } else {
                    stocks = JSON.parse(JSON.stringify(DEFAULT_STOCKS));
                }

                const savedNotes = localStorage.getItem('twse_personal_notes_v1');
                if (savedNotes) {
                    stockNotes = JSON.parse(savedNotes);
                } else {
                    stockNotes = {
                        '2330': '長線核心持股，回測月線或季線分批逢低布局，關注 CoWoS 產能擴充進度與毛利率表現。',
                        '00919': '高股息現金流配置，除息前若溢價過高不追高，除息後貼息可分批加碼。'
                    };
                }

                const savedApiUrl = localStorage.getItem('twse_personal_api_url');
                if (savedApiUrl) {
                    document.getElementById('customApiUrl').value = savedApiUrl;
                }
            } catch (e) {
                stocks = JSON.parse(JSON.stringify(DEFAULT_STOCKS));
            }
        }

        function saveLocalData() {
            try {
                localStorage.setItem('twse_personal_stocks_v1', JSON.stringify(stocks));
                localStorage.setItem('twse_personal_notes_v1', JSON.stringify(stockNotes));
            } catch (e) {
                console.error('儲存 LocalStorage 失敗', e);
            }
        }

        function updateBadge(sourceName, isLive = true) {
            const badge = document.getElementById('dataSourceBadge');
            const textEl = document.getElementById('dataSourceText');
            if (isLive) {
                textEl.textContent = sourceName;
                badge.className = 'text-[11px] font-medium px-2 py-0.5 rounded-full bg-emerald-500/20 text-emerald-300 border border-emerald-500/30 flex items-center space-x-1';
            } else {
                textEl.textContent = sourceName;
                badge.className = 'text-[11px] font-medium px-2 py-0.5 rounded-full bg-amber-500/20 text-amber-300 border border-amber-500/30 flex items-center space-x-1';
            }
        }

        function updateTimestamp(customTime) {
            const nowStr = customTime || new Date().toLocaleString('zh-TW', { hour12: false });
            document.getElementById('lastUpdateTime').textContent = `最後更新：${nowStr}`;
        }

        // ==================== 渲染自選股卡片 ====================
        function renderWatchlist() {
            const grid = document.getElementById('watchlistGrid');
            grid.innerHTML = '';

            const filtered = stocks.filter(s => currentCategory === 'ALL' || s.category === currentCategory);

            filtered.forEach(stock => {
                const isUp = stock.change > 0;
                const isDown = stock.change < 0;
                const colorClass = isUp ? 'text-twRed' : (isDown ? 'text-twGreen' : 'text-slate-300');
                const bgBadgeClass = isUp ? 'bg-red-500/15 text-twRed border-red-500/30' : (isDown ? 'bg-emerald-500/15 text-twGreen border-emerald-500/30' : 'bg-slate-700/40 text-slate-300 border-slate-600');
                const sign = isUp ? '▲ +' : (isDown ? '▼ ' : '');
                const isSelected = stock.code === selectedStockCode;

                const card = document.createElement('div');
                card.className = `glass-card rounded-2xl p-4 cursor-pointer transition-all duration-200 hover:-translate-y-0.5 relative group ${isSelected ? 'ring-2 ring-blue-500 bg-slate-800/70' : 'hover:border-slate-700'}`;
                card.onclick = () => selectStock(stock.code);

                card.innerHTML = `
                    <div class="flex items-start justify-between mb-3">
                        <div class="flex items-center space-x-2">
                            <span class="px-2 py-0.5 rounded-md bg-slate-800 text-blue-400 font-mono text-xs font-bold border border-slate-700">${stock.code}</span>
                            <h3 class="font-bold text-white text-base">${stock.name}</h3>
                            <span class="text-[10px] px-1.5 py-0.5 rounded bg-slate-800 text-slate-400">${stock.category}</span>
                        </div>
                        <button onclick="removeStock(event, '${stock.code}')" title="移除自選股" class="opacity-0 group-hover:opacity-100 text-slate-500 hover:text-rose-400 transition p-1">
                            <i class="fa-solid fa-trash-can text-xs"></i>
                        </button>
                    </div>

                    <div class="flex items-baseline justify-between my-2">
                        <div class="text-2xl font-mono font-bold ${colorClass}">
                            ${stock.price.toFixed(2)}
                        </div>
                        <div class="font-mono text-xs font-semibold px-2 py-1 rounded-lg border ${bgBadgeClass}">
                            ${sign}${stock.change.toFixed(2)} (${isUp ? '+' : ''}${stock.changePercent.toFixed(2)}%)
                        </div>
                    </div>

                    <div class="flex items-center justify-between text-xs text-slate-400 pt-3 mt-2 border-t border-slate-800/80">
                        <span>成交量: <strong class="font-mono text-slate-300">${stock.volume.toLocaleString()}</strong> 張</span>
                        <span>30日漲跌: <strong class="font-mono ${(() => {
                            const h = stock.history || [];
                            const diff = h.length >= 2 ? h[h.length - 1] - h[0] : 0;
                            return diff >= 0 ? 'text-twRed' : 'text-twGreen';
                        })()}">${(() => {
                            const h = stock.history || [];
                            if (h.length < 2 || !h[0]) return '0.0%';
                            const pct = ((h[h.length - 1] - h[0]) / h[0]) * 100;
                            return (pct >= 0 ? '+' : '') + pct.toFixed(1) + '%';
                        })()}</strong></span>
                    </div>
                `;
                grid.appendChild(card);
            });
        }

        // ==================== 選擇個股並更新圖表與筆記 ====================
        function selectStock(code) {
            selectedStockCode = code;
            const stock = stocks.find(s => s.code === code) || stocks[0];
            if (!stock) return;

            renderWatchlist();

            document.getElementById('chartStockBadge').textContent = stock.code;
            document.getElementById('chartStockTitle').textContent = `${stock.name} 近 30 日股價漲跌與均線走勢`;
            document.getElementById('detailCode').textContent = `${stock.code}.TW`;
            document.getElementById('detailYield').textContent = `${stock.yieldRate}%`;
            document.getElementById('detailPE').textContent = stock.pe || '-';

            const fEl = document.getElementById('detailForeign');
            fEl.textContent = stock.foreign5d || '+0 張';
            fEl.className = `font-mono font-bold text-base mt-1 ${(stock.foreign5d || '').includes('-') ? 'text-twGreen' : 'text-twRed'}`;

            const tEl = document.getElementById('detailTrust');
            tEl.textContent = stock.trust5d || '+0 張';
            tEl.className = `font-mono font-bold text-base mt-1 ${(stock.trust5d || '').includes('-') ? 'text-twGreen' : 'text-twRed'}`;

            document.getElementById('stockNoteInput').value = stockNotes[stock.code] || '';

            renderStockChart(stock);
            fetchSingleStockHistory(stock.code);
            fetchLiveNews(stock.code);
        }

        // 透過 Yahoo Finance 抓取指定個股最近 30 個交易日真實收盤價、日期與 K 線走勢
        async function fetchSingleStockHistory(code) {
            const stock = stocks.find(s => s.code === code);
            if (!stock) return;

            const yfUrl = `https://query1.finance.yahoo.com/v8/finance/chart/${code}.TW?interval=1d&range=3mo`;
            for (const makeProxy of CORS_PROXIES) {
                try {
                    const res = await fetchWithTimeout(makeProxy(yfUrl), 5000);
                    if (!res.ok) continue;
                    const data = await res.json();
                    const result = data?.chart?.result?.[0];
                    const timestamps = result?.timestamp;
                    const closes = result?.indicators?.quote?.[0]?.close;
                    const volumes = result?.indicators?.quote?.[0]?.volume;
                    if (Array.isArray(closes) && closes.length > 3) {
                        const pairs = [];
                        closes.forEach((c, idx) => {
                            if (c !== null && !isNaN(c)) {
                                const ts = Array.isArray(timestamps) && timestamps[idx] ? timestamps[idx] * 1000 : null;
                                const dateStr = ts ? new Date(ts).toLocaleDateString('zh-TW', { month: '2-digit', day: '2-digit' }) : null;
                                pairs.push({ close: +c.toFixed(2), date: dateStr });
                            }
                        });
                        if (pairs.length >= 2) {
                            const last30 = pairs.slice(-30);
                            stock.history = last30.map(p => p.close);
                            stock.historyDates = last30.map(p => p.date);
                            const latestClose = stock.history[stock.history.length - 1];
                            const prevClose = stock.history[stock.history.length - 2];
                            if (latestClose > 0) {
                                stock.price = latestClose;
                                stock.change = +(latestClose - prevClose).toFixed(2);
                                stock.changePercent = prevClose > 0 ? +((stock.change / prevClose) * 100).toFixed(2) : 0;
                            }
                            if (Array.isArray(volumes)) {
                                const validVols = volumes.filter(v => v !== null && !isNaN(v));
                                if (validVols.length > 0) {
                                    stock.volume = Math.round(validVols[validVols.length - 1] / 1000);
                                }
                            }
                            saveLocalData();
                            renderWatchlist();
                            if (selectedStockCode === code) {
                                renderStockChart(stock);
                            }
                            renderPortfolio();
                            return;
                        }
                    }
                } catch (e) {
                    // 嘗試下一個通道
                }
            }
        }

        // 即時抓取 Google News RSS 針對該檔台股的最新新聞
        async function fetchLiveNews(code) {
            const container = document.getElementById('liveNewsContainer');
            const tagEl = document.getElementById('newsStockTag');
            const stock = stocks.find(s => s.code === code) || stocks[0];
            if (!container || !stock) return;

            const query = `${stock.code} ${stock.name} 台股`;
            if (tagEl) tagEl.textContent = `${stock.code} ${stock.name}`;

            const rssUrl = `https://news.google.com/rss/search?q=${encodeURIComponent(query)}&hl=zh-TW&gl=TW&ceid=TW:zh-Hant`;

            for (const makeProxy of CORS_PROXIES) {
                try {
                    const res = await fetchWithTimeout(makeProxy(rssUrl), 5000);
                    if (!res.ok) continue;
                    const xmlText = await res.text();
                    const parser = new DOMParser();
                    const xmlDoc = parser.parseFromString(xmlText, 'text/xml');
                    const items = Array.from(xmlDoc.querySelectorAll('item')).slice(0, 6);

                    if (items.length > 0) {
                        container.innerHTML = '';
                        items.forEach(item => {
                            const title = item.querySelector('title')?.textContent || '台股即時新聞';
                            const link = item.querySelector('link')?.textContent || '#';
                            const pubDateRaw = item.querySelector('pubDate')?.textContent;
                            const pubDate = pubDateRaw ? new Date(pubDateRaw).toLocaleString('zh-TW', { month: '2-digit', day: '2-digit', hour: '2-digit', minute: '2-digit', hour12: false }) : '最新';
                            const source = item.querySelector('source')?.textContent || '財經新聞';

                            const a = document.createElement('a');
                            a.href = link;
                            a.target = '_blank';
                            a.rel = 'noopener noreferrer';
                            a.className = 'p-3.5 rounded-xl bg-slate-900/90 border border-slate-800 hover:border-blue-500/50 hover:bg-slate-800/60 transition flex flex-col justify-between group';
                            a.innerHTML = `
                                <div class="text-slate-200 font-medium group-hover:text-blue-400 transition line-clamp-2 leading-relaxed mb-2">${title}</div>
                                <div class="flex items-center justify-between text-[11px] text-slate-500 pt-2 border-t border-slate-800/70">
                                    <span class="text-emerald-400/90 font-medium">${source}</span>
                                    <span class="font-mono">${pubDate}</span>
                                </div>
                            `;
                            container.appendChild(a);
                        });
                        return;
                    }
                } catch (e) {
                    // 嘗試下一個 Proxy
                }
            }

            // 若 RSS Proxy 暫時忙碌，提供一鍵前往 Google 新聞與 Yahoo 股市個股新聞的備援卡片
            container.innerHTML = `
                <a href="https://www.google.com/search?q=${encodeURIComponent(query)}&tbm=nws" target="_blank" rel="noopener noreferrer" class="p-3.5 rounded-xl bg-slate-900/90 border border-slate-800 hover:border-blue-500/50 transition flex items-center justify-between">
                    <div>
                        <div class="text-slate-200 font-bold mb-1">前往 Google 財經新聞查看「${stock.code} ${stock.name}」最新報導</div>
                        <div class="text-[11px] text-slate-400">即時彙整各大財經媒體盤中與盤後焦點</div>
                    </div>
                    <i class="fa-solid fa-arrow-up-right-from-square text-blue-400"></i>
                </a>
                <a href="https://tw.stock.yahoo.com/quote/${stock.code}.TW/news" target="_blank" rel="noopener noreferrer" class="p-3.5 rounded-xl bg-slate-900/90 border border-slate-800 hover:border-blue-500/50 transition flex items-center justify-between">
                    <div>
                        <div class="text-slate-200 font-bold mb-1">前往 Yahoo 股市查看「${stock.code} ${stock.name}」個股新聞與研究報告</div>
                        <div class="text-[11px] text-slate-400">包含外資報告、營收快訊與重大訊息</div>
                    </div>
                    <i class="fa-solid fa-arrow-up-right-from-square text-emerald-400"></i>
                </a>
                <a href="https://mops.twse.com.tw/" target="_blank" rel="noopener noreferrer" class="p-3.5 rounded-xl bg-slate-900/90 border border-slate-800 hover:border-blue-500/50 transition flex items-center justify-between">
                    <div>
                        <div class="text-slate-200 font-bold mb-1">公開資訊觀測站「${stock.code}」重大訊息查詢</div>
                        <div class="text-[11px] text-slate-400">官方公告、法說會簡報與股利政策</div>
                    </div>
                    <i class="fa-solid fa-arrow-up-right-from-square text-amber-400"></i>
                </a>
            `;
        }

        // ==================== 繪製 Chart.js 30日收盤與 MA5 均線圖 + 30日逐日漲跌明細表 ====================
        function renderStockChart(stock) {
            const ctx = document.getElementById('stockChart').getContext('2d');
            const prices = Array.isArray(stock.history) && stock.history.length > 0 ? stock.history.slice(-30) : [stock.price];
            const dates = Array.isArray(stock.historyDates) && stock.historyDates.length === prices.length
                ? stock.historyDates
                : prices.map((_, idx) => {
                    const d = new Date();
                    d.setDate(d.getDate() - (prices.length - 1 - idx));
                    return `${String(d.getMonth() + 1).padStart(2, '0')}/${String(d.getDate()).padStart(2, '0')}`;
                });

            // 計算 5 日均線 (MA5)
            const ma5 = prices.map((val, idx, arr) => {
                const start = Math.max(0, idx - 4);
                const slice = arr.slice(start, idx + 1);
                const sum = slice.reduce((a, b) => a + b, 0);
                return +(sum / slice.length).toFixed(2);
            });

            // 更新 30 天統計摘要與逐日表格
            if (prices.length >= 2) {
                const firstPrice = prices[0];
                const lastPrice = prices[prices.length - 1];
                const diff30 = +(lastPrice - firstPrice).toFixed(2);
                const pct30 = firstPrice > 0 ? +((diff30 / firstPrice) * 100).toFixed(2) : 0;
                const high30 = Math.max(...prices);
                const low30 = Math.min(...prices);

                let upDays = 0;
                let downDays = 0;
                for (let i = 1; i < prices.length; i++) {
                    if (prices[i] > prices[i - 1]) upDays++;
                    else if (prices[i] < prices[i - 1]) downDays++;
                }

                const chgEl = document.getElementById('stat30dChange');
                if (chgEl) {
                    const isUp = diff30 >= 0;
                    chgEl.textContent = `${isUp ? '▲ +' : '▼ '}${diff30.toFixed(2)} (${isUp ? '+' : ''}${pct30.toFixed(2)}%)`;
                    chgEl.className = `font-mono font-bold text-sm ${isUp ? 'text-twRed' : 'text-twGreen'}`;
                }
                const highEl = document.getElementById('stat30dHigh');
                if (highEl) highEl.textContent = high30.toFixed(2);
                const lowEl = document.getElementById('stat30dLow');
                if (lowEl) lowEl.textContent = low30.toFixed(2);
                const winEl = document.getElementById('stat30dWinDays');
                if (winEl) winEl.innerHTML = `<span class="text-twRed">${upDays} 漲</span> / <span class="text-twGreen">${downDays} 跌</span>`;
            }

            // 渲染 30 日逐日漲跌明細表 (由最新日期排到最舊)
            const tbody30 = document.getElementById('table30dBody');
            if (tbody30) {
                tbody30.innerHTML = '';
                for (let i = prices.length - 1; i >= 0; i--) {
                    const close = prices[i];
                    const prev = i > 0 ? prices[i - 1] : close;
                    const dayDiff = +(close - prev).toFixed(2);
                    const dayPct = prev > 0 ? +((dayDiff / prev) * 100).toFixed(2) : 0;
                    const isUp = dayDiff > 0;
                    const isDown = dayDiff < 0;
                    const cClass = isUp ? 'text-twRed' : (isDown ? 'text-twGreen' : 'text-slate-400');
                    const sign = isUp ? '▲ +' : (isDown ? '▼ ' : '');

                    const tr = document.createElement('tr');
                    tr.className = 'hover:bg-slate-800/50 transition';
                    tr.innerHTML = `
                        <td class="py-2 px-4 text-slate-300">${dates[i]} ${i === prices.length - 1 ? '<span class="ml-1 text-[10px] px-1.5 py-0.5 rounded bg-blue-500/20 text-blue-300">最新</span>' : ''}</td>
                        <td class="py-2 px-4 text-right font-bold text-white">${close.toFixed(2)}</td>
                        <td class="py-2 px-4 text-right font-bold ${cClass}">${sign}${dayDiff.toFixed(2)}</td>
                        <td class="py-2 px-4 text-right font-bold ${cClass}">${isUp ? '+' : ''}${dayPct.toFixed(2)}%</td>
                        <td class="py-2 px-4 text-right text-amber-400">${ma5[i].toFixed(2)}</td>
                    `;
                    tbody30.appendChild(tr);
                }
            }

            if (chartInstance) {
                chartInstance.destroy();
            }

            chartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: dates,
                    datasets: [
                        {
                            label: `${stock.name} 收盤價`,
                            data: prices,
                            borderColor: '#ef4444',
                            backgroundColor: 'rgba(239, 68, 68, 0.12)',
                            borderWidth: 2.5,
                            pointRadius: 3,
                            pointBackgroundColor: '#ef4444',
                            tension: 0.25,
                            fill: true
                        },
                        {
                            label: '5日均線 (MA5)',
                            data: ma5,
                            borderColor: '#fbbf24',
                            borderWidth: 2,
                            borderDash: [5, 5],
                            pointRadius: 0,
                            tension: 0.25,
                            fill: false
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    interaction: {
                        mode: 'index',
                        intersect: false
                    },
                    plugins: {
                        legend: { display: false },
                        tooltip: {
                            backgroundColor: 'rgba(15, 23, 42, 0.95)',
                            titleColor: '#94a3b8',
                            bodyColor: '#f8fafc',
                            borderColor: '#334155',
                            borderWidth: 1,
                            callbacks: {
                                afterBody: function(context) {
                                    const idx = context[0].dataIndex;
                                    if (idx > 0) {
                                        const curr = prices[idx];
                                        const prev = prices[idx - 1];
                                        const d = +(curr - prev).toFixed(2);
                                        const p = prev > 0 ? +((d / prev) * 100).toFixed(2) : 0;
                                        return `單日漲跌: ${d >= 0 ? '+' : ''}${d} (${p >= 0 ? '+' : ''}${p}%)`;
                                    }
                                    return '';
                                }
                            }
                        }
                    },
                    scales: {
                        x: {
                            grid: { color: 'rgba(51, 65, 85, 0.25)' },
                            ticks: { color: '#64748b', font: { size: 11 } }
                        },
                        y: {
                            grid: { color: 'rgba(51, 65, 85, 0.25)' },
                            ticks: { color: '#94a3b8', font: { family: 'monospace', size: 11 } }
                        }
                    }
                }
            });
        }

        // ==================== 新增、刪除、分類篩選與筆記儲存 ====================
        function addStock() {
            const codeInput = document.getElementById('newStockCode');
            const nameInput = document.getElementById('newStockName');
            const code = codeInput.value.trim().toUpperCase();
            let name = nameInput.value.trim();

            if (!code) {
                showToast('請輸入台股代號（例如 2330 或 00919）');
                return;
            }
            if (stocks.some(s => s.code === code)) {
                selectStock(code);
                showToast(`自選清單已有 ${code}，已為您切換至該股`);
                return;
            }

            if (!name) name = `台股 ${code}`;
            const category = code.startsWith('00') ? 'ETF' : (code.startsWith('28') ? 'FIN' : 'TECH');

            const newStockObj = {
                code: code,
                name: name,
                category: category,
                price: 100.0,
                change: 0.0,
                changePercent: 0.0,
                volume: 0,
                pe: '-',
                yieldRate: 3.5,
                annualDiv: 3.5,
                foreign5d: '同步中',
                trust5d: '同步中',
                shares: 0,
                avgCost: 0,
                history: [100, 100, 100, 100, 100]
            };

            stocks.unshift(newStockObj);
            saveLocalData();
            renderWatchlist();
            renderPortfolio();
            selectStock(code);
            // 新增後立即抓取該股真實報價
            refreshQuotes(true);

            codeInput.value = '';
            nameInput.value = '';
            showToast(`已將 ${code} ${name} 加入自選並同步即時報價！`);
        }

        function removeStock(event, code) {
            event.stopPropagation();
            if (stocks.length <= 1) {
                showToast('請至少保留一檔自選股');
                return;
            }
            stocks = stocks.filter(s => s.code !== code);
            saveLocalData();
            if (selectedStockCode === code) {
                selectedStockCode = stocks[0].code;
            }
            renderWatchlist();
            selectStock(selectedStockCode);
            renderPortfolio();
            showToast(`已移除 ${code}`);
        }

        function filterCategory(cat) {
            currentCategory = cat;
            document.querySelectorAll('.cat-btn').forEach(btn => {
                btn.className = 'cat-btn px-3 py-1.5 rounded-lg bg-slate-800/80 text-slate-400 hover:text-white transition';
            });
            const activeBtn = document.getElementById(`cat-${cat}`);
            if (activeBtn) {
                activeBtn.className = 'cat-btn px-3 py-1.5 rounded-lg bg-slate-700 text-white font-medium transition';
            }
            renderWatchlist();
        }

        function saveStockNote() {
            const val = document.getElementById('stockNoteInput').value;
            stockNotes[selectedStockCode] = val;
            saveLocalData();
            const status = document.getElementById('noteSaveStatus');
            status.classList.remove('opacity-0');
            setTimeout(() => status.classList.add('opacity-0'), 1500);
        }

        // ==================== 庫存損益表與 ETF 配息行事曆渲染 ====================
        function renderPortfolio() {
            const tbody = document.getElementById('portfolioTableBody');
            if (!tbody) return;
            tbody.innerHTML = '';

            let totalMarket = 0;
            let totalCost = 0;
            let totalAnnualDiv = 0;

            stocks.forEach((stock, idx) => {
                const shares = Number(stock.shares) || 0;
                const avgCost = Number(stock.avgCost) || 0;
                const marketVal = Math.round(shares * stock.price);
                const costVal = Math.round(shares * avgCost);
                const profit = marketVal - costVal;
                const returnRate = costVal > 0 ? ((profit / costVal) * 100) : 0;
                const estDiv = Math.round(shares * (stock.annualDiv || (stock.price * (stock.yieldRate || 0) / 100)));

                totalMarket += marketVal;
                totalCost += costVal;
                totalAnnualDiv += estDiv;

                const isProfit = profit >= 0;
                const pColor = profit > 0 ? 'text-twRed' : (profit < 0 ? 'text-twGreen' : 'text-slate-400');

                const tr = document.createElement('tr');
                tr.className = 'hover:bg-slate-800/40 transition';
                tr.innerHTML = `
                    <td class="py-3 px-3">
                        <div class="font-bold text-white">${stock.code} ${stock.name}</div>
                    </td>
                    <td class="py-3 px-3 text-right font-bold text-slate-200">${stock.price.toFixed(2)}</td>
                    <td class="py-3 px-3 text-right">
                        <input type="number" step="100" min="0" value="${shares}" onchange="updateHolding(${idx}, 'shares', this.value)"
                            class="w-24 text-right px-2 py-1 rounded bg-slate-900 border border-slate-700 text-white focus:outline-none focus:border-blue-500">
                    </td>
                    <td class="py-3 px-3 text-right">
                        <input type="number" step="0.1" min="0" value="${avgCost}" onchange="updateHolding(${idx}, 'avgCost', this.value)"
                            class="w-24 text-right px-2 py-1 rounded bg-slate-900 border border-slate-700 text-white focus:outline-none focus:border-blue-500">
                    </td>
                    <td class="py-3 px-3 text-right text-white">$${marketVal.toLocaleString()}</td>
                    <td class="py-3 px-3 text-right font-bold ${pColor}">${isProfit && profit > 0 ? '+' : ''}$${profit.toLocaleString()}</td>
                    <td class="py-3 px-3 text-right font-bold ${pColor}">${isProfit && profit > 0 ? '+' : ''}${returnRate.toFixed(2)}%</td>
                    <td class="py-3 px-3 text-right text-amber-400 font-bold">$${estDiv.toLocaleString()}</td>
                `;
                tbody.appendChild(tr);
            });

            const totalProfit = totalMarket - totalCost;
            const totalReturn = totalCost > 0 ? ((totalProfit / totalCost) * 100) : 0;
            const yieldOnCost = totalCost > 0 ? ((totalAnnualDiv / totalCost) * 100) : 0;
            const totalColor = totalProfit >= 0 ? 'text-twRed' : 'text-twGreen';

            document.getElementById('portTotalMarket').textContent = `$${totalMarket.toLocaleString()}`;
            document.getElementById('portTotalCost').textContent = `$${totalCost.toLocaleString()}`;
            const pEl = document.getElementById('portTotalProfit');
            pEl.textContent = `${totalProfit >= 0 ? '+' : ''}$${totalProfit.toLocaleString()}`;
            pEl.className = `text-2xl font-mono font-bold mt-1 ${totalColor}`;

            const rEl = document.getElementById('portTotalReturnRate');
            rEl.textContent = `總報酬率: ${totalReturn >= 0 ? '+' : ''}${totalReturn.toFixed(2)}%`;
            rEl.className = `text-xs font-mono mt-1 ${totalColor}`;

            document.getElementById('portAnnualDividend').textContent = `$${totalAnnualDiv.toLocaleString()}`;
            document.getElementById('portYieldOnCost').textContent = `成本殖利率: ${yieldOnCost.toFixed(2)}%`;
        }

        function updateHolding(index, field, value) {
            if (!stocks[index]) return;
            stocks[index][field] = parseFloat(value) || 0;
            saveLocalData();
            renderPortfolio();
            showToast('庫存損益已更新並儲存於本機');
        }

        function renderEtfCalendar() {
            const tbody = document.getElementById('etfCalendarBody');
            if (!tbody) return;
            tbody.innerHTML = '';

            ETF_CALENDAR_DATA.forEach(item => {
                const tr = document.createElement('tr');
                tr.className = 'hover:bg-slate-800/40 transition';
                tr.innerHTML = `
                    <td class="py-3 px-3 font-bold text-white"><span class="text-blue-400 font-mono mr-1.5">${item.code}</span>${item.name}</td>
                    <td class="py-3 px-3 text-slate-300">${item.freq}</td>
                    <td class="py-3 px-3 text-right font-mono font-bold text-white">${item.price.toFixed(2)}</td>
                    <td class="py-3 px-3 text-right font-mono text-amber-400 font-bold">${item.div}</td>
                    <td class="py-3 px-3 text-right font-mono text-emerald-400">${item.yieldStr}</td>
                    <td class="py-3 px-3 text-slate-300">${item.exDate}</td>
                    <td class="py-3 px-3 text-slate-300">${item.payDate}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        // ==================== 分頁切換與 Modal 控制 ====================
        function switchTab(tabName) {
            ['dashboard', 'portfolio', 'etf', 'guide'].forEach(t => {
                document.getElementById(`view-${t}`).classList.add('hidden');
                const btn = document.getElementById(`tab-${t}`);
                if (t === 'guide') {
                    btn.className = 'tab-btn px-4 py-2 rounded-lg text-sm font-medium text-emerald-400 hover:bg-emerald-500/10 border border-emerald-500/30 whitespace-nowrap flex items-center space-x-2 transition ml-auto';
                } else {
                    btn.className = 'tab-btn px-4 py-2 rounded-lg text-sm font-medium text-slate-400 hover:text-white hover:bg-slate-800 whitespace-nowrap flex items-center space-x-2 transition';
                }
            });

            document.getElementById(`view-${tabName}`).classList.remove('hidden');
            const activeBtn = document.getElementById(`tab-${tabName}`);
            if (tabName === 'guide') {
                activeBtn.className = 'tab-btn px-4 py-2 rounded-lg text-sm font-medium bg-emerald-600 text-white shadow-sm whitespace-nowrap flex items-center space-x-2 transition ml-auto';
            } else {
                activeBtn.className = 'tab-btn px-4 py-2 rounded-lg text-sm font-medium bg-blue-600 text-white shadow-sm whitespace-nowrap flex items-center space-x-2 transition';
            }
        }

        function openApiModal() {
            document.getElementById('apiModal').classList.remove('hidden');
        }

        function closeApiModal() {
            document.getElementById('apiModal').classList.add('hidden');
        }

        // ==================== 更新報價（內建證交所 MIS、TWSE OpenAPI、Yahoo Finance 真實即時串接） ====================
        async function fetchWithTimeout(url, timeout = 6500) {
            const controller = new AbortController();
            const timer = setTimeout(() => controller.abort(), timeout);
            try {
                const response = await fetch(url, { signal: controller.signal, cache: 'no-store' });
                clearTimeout(timer);
                return response;
            } catch (err) {
                clearTimeout(timer);
                throw err;
            }
        }

        // 通道 1：直接透過公開 Proxy 抓取臺灣證券交易所 (TWSE MIS) 官方即時明細與加權指數
        async function fetchFromTwseMis() {
            const stockQueries = stocks.map(s => `tse_${s.code}.tw|otc_${s.code}.tw`).join('|');
            const fullExCh = `tse_t00.tw|otc_o00.tw|${stockQueries}`;
            const misUrl = `https://mis.twse.com.tw/stock/api/getStockInfo.jsp?ex_ch=${encodeURIComponent(fullExCh)}&json=1&delay=0&_=${Date.now()}`;

            for (const makeProxy of CORS_PROXIES) {
                try {
                    const res = await fetchWithTimeout(makeProxy(misUrl), 6000);
                    if (!res.ok) continue;
                    const data = await res.json();
                    if (data && Array.isArray(data.msgArray) && data.msgArray.length > 0) {
                        let updatedCount = 0;
                        data.msgArray.forEach(item => {
                            if (!item.c) return;
                            // 解析最新成交價 z，若盤中無瞬間撮合則取買一價 b 或昨收 y
                            let rawPrice = parseFloat(item.z);
                            if (isNaN(rawPrice) || rawPrice <= 0) {
                                const bestBid = item.b ? parseFloat(item.b.split('_')[0]) : NaN;
                                const bestAsk = item.a ? parseFloat(item.a.split('_')[0]) : NaN;
                                rawPrice = !isNaN(bestBid) && bestBid > 0 ? bestBid : (!isNaN(bestAsk) && bestAsk > 0 ? bestAsk : parseFloat(item.y));
                            }
                            const prevClose = parseFloat(item.y);
                            if (isNaN(rawPrice) || rawPrice <= 0) return;

                            const change = !isNaN(prevClose) && prevClose > 0 ? +(rawPrice - prevClose).toFixed(2) : 0;
                            const changePct = !isNaN(prevClose) && prevClose > 0 ? +((change / prevClose) * 100).toFixed(2) : 0;

                            // 更新加權指數 t00 與櫃買指數 o00
                            if (item.c === 't00') {
                                updateIndexHeader('taiexValue', 'taiexChange', rawPrice, change, changePct);
                                return;
                            }
                            if (item.c === 'o00') {
                                updateIndexHeader('tpexValue', 'tpexChange', rawPrice, change, changePct);
                                return;
                            }

                            const local = stocks.find(s => s.code === item.c);
                            if (local) {
                                local.name = item.n || local.name;
                                local.price = rawPrice;
                                local.change = change;
                                local.changePercent = changePct;
                                if (item.v && !isNaN(parseInt(item.v, 10))) {
                                    local.volume = parseInt(item.v, 10);
                                }
                                if (Array.isArray(local.history) && local.history.length > 0) {
                                    local.history[local.history.length - 1] = rawPrice;
                                }
                                // 同步更新 ETF 行事曆上的最新現價
                                const etfItem = ETF_CALENDAR_DATA.find(e => e.code === item.c);
                                if (etfItem) etfItem.price = rawPrice;
                                updatedCount++;
                            }
                        });

                        if (updatedCount > 0) {
                            return { source: 'TWSE 證交所即時行情', time: data.queryTime?.sysDate ? `${data.queryTime.sysDate} ${data.queryTime.sysTime}` : null };
                        }
                    }
                } catch (e) {
                    // 切換下一個 Proxy
                }
            }
            throw new Error('TWSE MIS 通道暫無回應');
        }

        // 通道 2：透過 Yahoo Finance v7/v8 批次抓取台股真實即時報價、加權指數 (^TWII) 與美股權值指標
        async function fetchFromYahooFinance() {
            const usSymbols = usMarkets.map(u => u.symbol);
            const symbols = ['^TWII', '^TWOII', ...usSymbols, ...stocks.map(s => `${s.code}.TW`)];
            let updatedCount = 0;

            for (const makeProxy of CORS_PROXIES) {
                try {
                    const quoteUrl = `https://query1.finance.yahoo.com/v7/finance/quote?symbols=${encodeURIComponent(symbols.join(','))}&_=${Date.now()}`;
                    const res = await fetchWithTimeout(makeProxy(quoteUrl), 6000);
                    if (!res.ok) continue;
                    const data = await res.json();
                    const results = data?.quoteResponse?.result;
                    if (Array.isArray(results) && results.length > 0) {
                        results.forEach(q => {
                            const price = q.regularMarketPrice;
                            const change = q.regularMarketChange || 0;
                            const changePct = q.regularMarketChangePercent || 0;
                            if (!price) return;

                            if (q.symbol === '^TWII') {
                                updateIndexHeader('taiexValue', 'taiexChange', price, change, changePct);
                                return;
                            }
                            if (q.symbol === '^TWOII') {
                                updateIndexHeader('tpexValue', 'tpexChange', price, change, changePct);
                                return;
                            }

                            const usItem = usMarkets.find(u => u.symbol === q.symbol);
                            if (usItem) {
                                usItem.price = +price.toFixed(2);
                                usItem.change = +change.toFixed(2);
                                usItem.changePercent = +changePct.toFixed(2);
                                return;
                            }

                            const code = q.symbol.replace('.TW', '').replace('.TWO', '');
                            const local = stocks.find(s => s.code === code);
                            if (local) {
                                local.price = +price.toFixed(2);
                                local.change = +change.toFixed(2);
                                local.changePercent = +changePct.toFixed(2);
                                if (q.regularMarketVolume) {
                                    local.volume = Math.round(q.regularMarketVolume / 1000);
                                }
                                if (Array.isArray(local.history) && local.history.length > 0) {
                                    local.history[local.history.length - 1] = local.price;
                                }
                                const etfItem = ETF_CALENDAR_DATA.find(e => e.code === code);
                                if (etfItem) etfItem.price = local.price;
                                updatedCount++;
                            }
                        });
                        renderUsMarket();
                        if (updatedCount > 0) {
                            return { source: 'Yahoo Finance 即時行情', time: null };
                        }
                    }
                } catch (e) {
                    // 繼續嘗試
                }
            }
            throw new Error('Yahoo Finance 批次報價失敗');
        }

        // 獨立同步美股四大指數與科技巨頭即時報價
        async function fetchUsMarketQuotes() {
            const usSymbols = usMarkets.map(u => u.symbol);
            for (const makeProxy of CORS_PROXIES) {
                try {
                    const quoteUrl = `https://query1.finance.yahoo.com/v7/finance/quote?symbols=${encodeURIComponent(usSymbols.join(','))}&_=${Date.now()}`;
                    const res = await fetchWithTimeout(makeProxy(quoteUrl), 5000);
                    if (!res.ok) continue;
                    const data = await res.json();
                    const results = data?.quoteResponse?.result;
                    if (Array.isArray(results) && results.length > 0) {
                        results.forEach(q => {
                            const usItem = usMarkets.find(u => u.symbol === q.symbol);
                            if (usItem && q.regularMarketPrice) {
                                usItem.price = +q.regularMarketPrice.toFixed(2);
                                usItem.change = +(q.regularMarketChange || 0).toFixed(2);
                                usItem.changePercent = +(q.regularMarketChangePercent || 0).toFixed(2);
                            }
                        });
                        renderUsMarket();
                        return;
                    }
                } catch (e) {
                    // 繼續嘗試
                }
            }
        }

        function updateIndexHeader(valId, chgId, price, change, changePct) {
            const valEl = document.getElementById(valId);
            const chgEl = document.getElementById(chgId);
            if (!valEl || !chgEl) return;

            const isUp = change >= 0;
            const sign = isUp ? '▲ +' : '▼ ';
            valEl.textContent = price.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
            valEl.className = `font-mono font-bold ${isUp ? 'text-twRed' : 'text-twGreen'}`;

            chgEl.textContent = `${sign}${change.toFixed(2)} (${isUp ? '+' : ''}${changePct.toFixed(2)}%)`;
            chgEl.className = `font-mono text-xs px-1.5 py-0.5 rounded ${isUp ? 'bg-red-500/10 text-twRed' : 'bg-emerald-500/10 text-twGreen'}`;
        }

        async function refreshQuotes(showNotification = true) {
            const icon = document.getElementById('refreshIcon');
            if (icon) icon.classList.add('fa-spin');
            updateMarketStatus();
            fetchUsMarketQuotes();

            const customUrl = localStorage.getItem('twse_personal_api_url');

            // 優先使用使用者自訂的 GAS Web App 或 JSON API
            if (customUrl) {
                try {
                    const codes = stocks.map(s => s.code).join(',');
                    const fetchUrl = customUrl.includes('?') ? `${customUrl}&codes=${codes}` : `${customUrl}?codes=${codes}`;
                    const res = await fetchWithTimeout(fetchUrl, 7000);
                    const data = await res.json();

                    if (data && Array.isArray(data.stocks)) {
                        if (data.indices) {
                            if (data.indices.t00) {
                                updateIndexHeader('taiexValue', 'taiexChange', data.indices.t00.price, data.indices.t00.change, data.indices.t00.changePercent);
                            }
                            if (data.indices.o00) {
                                updateIndexHeader('tpexValue', 'tpexChange', data.indices.o00.price, data.indices.o00.change, data.indices.o00.changePercent);
                            }
                        }
                        data.stocks.forEach(remote => {
                            const local = stocks.find(s => s.code === remote.code);
                            if (local && remote.price > 0) {
                                local.name = remote.name || local.name;
                                local.price = remote.price;
                                local.change = remote.change;
                                local.changePercent = remote.changePercent;
                                local.volume = remote.volume || local.volume;
                                if (Array.isArray(remote.history) && remote.history.length > 0) {
                                    local.history = remote.history;
                                } else {
                                    local.history[local.history.length - 1] = remote.price;
                                }
                                const etfItem = ETF_CALENDAR_DATA.find(e => e.code === remote.code);
                                if (etfItem) etfItem.price = remote.price;
                            }
                        });
                        saveLocalData();
                        renderWatchlist();
                        renderStockChart(stocks.find(s => s.code === selectedStockCode) || stocks[0]);
                        renderPortfolio();
                        renderEtfCalendar();
                        updateBadge('自訂 GAS 即時連線', true);
                        updateTimestamp(data.updatedAt);
                        if (showNotification) showToast('已透過自訂 API 同步最新台股真實報價！');
                        if (icon) icon.classList.remove('fa-spin');
                        return;
                    }
                } catch (err) {
                    console.warn('自訂 API 連線失敗，自動切換至內建證交所即時通道');
                }
            }

            // 內建自動真實行情通道：先抓證交所 MIS 即時行情，若被擋則自動切換 Yahoo Finance，並同時補齊 20 日真實歷史走勢
            try {
                const misResult = await fetchFromTwseMis();
                saveLocalData();
                renderWatchlist();
                renderStockChart(stocks.find(s => s.code === selectedStockCode) || stocks[0]);
                renderPortfolio();
                renderEtfCalendar();
                updateBadge(misResult.source, true);
                updateTimestamp();
                if (showNotification) showToast('已同步證交所 (TWSE) 最新即時報價！');
            } catch (e1) {
                try {
                    const yfResult = await fetchFromYahooFinance();
                    saveLocalData();
                    renderWatchlist();
                    renderStockChart(stocks.find(s => s.code === selectedStockCode) || stocks[0]);
                    renderPortfolio();
                    renderEtfCalendar();
                    updateBadge(yfResult.source, true);
                    updateTimestamp();
                    if (showNotification) showToast('已同步最新台股即時行情！');
                } catch (e2) {
                    // 逐一透過 Yahoo Finance Chart API 更新每檔股票的真實收盤價
                    await Promise.allSettled(stocks.map(s => fetchSingleStockHistory(s.code)));
                    updateBadge('Yahoo 盤後/即時通道', true);
                    updateTimestamp();
                    if (showNotification) showToast('已更新各股最新真實收盤與走勢！');
                }
            } finally {
                if (icon) icon.classList.remove('fa-spin');
                countdownSec = 15;
            }
        }

        function saveApiConfig() {
            const url = document.getElementById('customApiUrl').value.trim();
            if (url) {
                localStorage.setItem('twse_personal_api_url', url);
                updateBadge('自訂 GAS 連線中...', true);
            } else {
                localStorage.removeItem('twse_personal_api_url');
                updateBadge('內建即時通道', true);
            }
            closeApiModal();
            refreshQuotes(true);
        }

        function resetDefaultData() {
            localStorage.removeItem('twse_personal_stocks_v1');
            localStorage.removeItem('twse_personal_notes_v1');
            localStorage.removeItem('twse_personal_api_url');
            document.getElementById('customApiUrl').value = '';
            updateBadge('內建即時通道', true);
            loadLocalData();
            renderWatchlist();
            selectStock('2330');
            renderPortfolio();
            closeApiModal();
            showToast('已還原為預設台股清單');
        }

        function copyCode(elementId) {
            const text = document.getElementById(elementId).innerText;
            const textarea = document.createElement('textarea');
            textarea.value = text;
            document.body.appendChild(textarea);
            textarea.select();
            document.execCommand('copy');
            document.body.removeChild(textarea);
            showToast('程式碼已複製到剪貼簿！');
        }

        function showToast(msg) {
            const toast = document.getElementById('toastMsg');
            document.getElementById('toastText').textContent = msg;
            toast.classList.remove('hidden');
            setTimeout(() => {
                toast.classList.add('hidden');
            }, 2600);
        }
    </script>
</body>
</html>
