<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HTMLDrop — 靜態網頁即時託管與預覽中心</title>
    <!-- 引入 Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 引入 Google Fonts: Inter 與 JetBrains Mono -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
    <!-- 引入 Font Awesome 圖示庫 -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                        mono: ['JetBrains Mono', 'monospace'],
                    }
                }
            }
        }
    </script>

    <style>
        /* 自訂捲軸 */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        ::-webkit-scrollbar-track {
            background: transparent;
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(148, 163, 184, 0.35);
            border-radius: 9999px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: rgba(99, 102, 241, 0.6);
        }

        /* 拖曳區呼吸發光與動態虛線效果 */
        .drop-zone-active {
            border-color: #6366f1 !important;
            background-color: rgba(99, 102, 241, 0.08) !important;
            transform: scale(1.01);
        }

        /* 卡片內嵌迷你 iframe 縮放容器 */
        .thumbnail-wrapper {
            position: relative;
            width: 100%;
            height: 176px;
            overflow: hidden;
            background: #ffffff;
        }
        .thumbnail-iframe {
            width: 1280px;
            height: 800px;
            border: 0;
            transform: scale(0.28);
            transform-origin: 0 0;
            pointer-events: none;
            user-select: none;
        }
        @media (max-width: 640px) {
            .thumbnail-iframe {
                transform: scale(0.26);
            }
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 dark:bg-slate-950 dark:text-slate-100 font-sans antialiased transition-colors duration-200 min-h-screen flex flex-col">

    <!-- 1. 頂部導覽列 -->
    <nav class="sticky top-0 z-30 bg-white/80 dark:bg-slate-900/80 backdrop-blur-md border-b border-slate-200 dark:border-slate-800">
        <div class="max-w-6xl mx-auto px-4 sm:px-6 h-16 flex items-center justify-between gap-4">
            <!-- 品牌 Logo 與名稱 -->
            <a href="#" class="flex items-center gap-3 group">
                <div class="w-10 h-10 rounded-xl bg-gradient-to-br from-indigo-600 to-violet-600 text-white flex items-center justify-center shadow-md shadow-indigo-500/20 group-hover:scale-105 transition-transform">
                    <i class="fa-solid fa-code text-lg"></i>
                </div>
                <div>
                    <div class="flex items-center gap-2">
                        <span class="font-extrabold text-xl tracking-tight text-slate-900 dark:text-white">HTML<span class="text-indigo-600 dark:text-indigo-400">Drop</span></span>
                        <span class="text-[10px] font-mono font-semibold uppercase px-1.5 py-0.5 rounded bg-emerald-500/10 text-emerald-600 dark:text-emerald-400 border border-emerald-500/20">Beta</span>
                    </div>
                    <p class="text-[11px] text-slate-500 dark:text-slate-400 hidden sm:block">Instant Static Hosting & Sandbox Preview</p>
                </div>
            </a>

            <!-- 右側導覽與本地儲存狀態控制 -->
            <div class="flex items-center gap-2 sm:gap-3">
                <!-- 本地儲存空間指示器 -->
                <div class="hidden md:flex items-center gap-2 px-3 py-1.5 rounded-xl bg-slate-100 dark:bg-slate-800/80 border border-slate-200 dark:border-slate-700 text-xs font-mono">
                    <span class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span>
                    <span class="text-slate-500 dark:text-slate-400">LocalStorage:</span>
                    <span id="storage-usage" class="font-semibold text-slate-700 dark:text-slate-200">0 KB</span>
                </div>

                <!-- 深色/淺色主題切換 -->
                <button id="theme-toggle" type="button" onclick="toggleTheme()"
                        class="w-9 h-9 rounded-xl border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-800 text-slate-600 dark:text-slate-300 hover:text-indigo-600 dark:hover:text-indigo-400 flex items-center justify-center transition cursor-pointer"
                        title="切換深色 / 淺色主題">
                    <i id="theme-icon" class="fa-solid fa-moon"></i>
                </button>

                <!-- 清除全部按鈕 -->
                <button type="button" onclick="requestClearAll()"
                        class="px-3.5 py-2 rounded-xl border border-rose-200 dark:border-rose-900/60 bg-rose-50/70 dark:bg-rose-950/40 hover:bg-rose-600 hover:text-white dark:hover:bg-rose-600 text-rose-600 dark:text-rose-400 text-xs sm:text-sm font-medium flex items-center gap-1.5 transition cursor-pointer">
                    <i class="fa-regular fa-trash-can"></i>
                    <span>清除全部</span>
                </button>
            </div>
        </div>
    </nav>

    <!-- 主內容區 -->
    <main class="flex-1 max-w-6xl w-full mx-auto px-4 sm:px-6 py-8 sm:py-12 space-y-12">

        <!-- 2. 核心英雄區 (Hero Section) 與上傳控制台 -->
        <section class="max-w-3xl mx-auto text-center space-y-6">
            <div class="space-y-3">
                <div class="inline-flex items-center gap-2 px-3.5 py-1 rounded-full bg-indigo-50 dark:bg-indigo-950/60 border border-indigo-200 dark:border-indigo-800/60 text-indigo-600 dark:text-indigo-300 text-xs font-medium">
                    <i class="fa-solid fa-bolt text-amber-500"></i>
                    <span>免後端部署・瀏覽器沙盒即時渲染</span>
                </div>
                <h1 class="text-3xl sm:text-5xl font-extrabold tracking-tight text-slate-900 dark:text-white leading-tight">
                    拖曳上傳你的 <span class="text-transparent bg-clip-text bg-gradient-to-r from-indigo-600 to-emerald-500">HTML 網頁</span><br class="hidden sm:inline"> 秒速生成可預覽的雲端頁面
                </h1>
                <p class="text-slate-500 dark:text-slate-400 text-sm sm:text-base max-w-xl mx-auto">
                    支援多檔拖曳上傳、原始碼貼上部署與多裝置視角即時測試。所有檔案皆安全儲存於您的本機瀏覽器中。
                </p>
            </div>

            <!-- 上傳模式切換分頁 -->
            <div class="bg-white dark:bg-slate-900 rounded-2xl shadow-xl shadow-slate-200/60 dark:shadow-none border border-slate-200 dark:border-slate-800 overflow-hidden text-left">
                <!-- 分頁按鈕列 -->
                <div class="flex border-b border-slate-200 dark:border-slate-800 bg-slate-50/80 dark:bg-slate-900/90 p-1.5 gap-1.5">
                    <button type="button" id="tab-file" onclick="switchMode('file')"
                            class="flex-1 py-2.5 px-4 rounded-xl text-xs sm:text-sm font-semibold flex items-center justify-center gap-2 bg-white dark:bg-slate-800 text-indigo-600 dark:text-indigo-400 shadow-sm transition cursor-pointer">
                        <i class="fa-solid fa-cloud-arrow-up"></i>
                        <span>拖曳 / 選擇 HTML 檔案</span>
                    </button>
                    <button type="button" id="tab-code" onclick="switchMode('code')"
                            class="flex-1 py-2.5 px-4 rounded-xl text-xs sm:text-sm font-semibold flex items-center justify-center gap-2 text-slate-500 dark:text-slate-400 hover:text-slate-800 dark:hover:text-slate-200 transition cursor-pointer">
                        <i class="fa-solid fa-file-code"></i>
                        <span>貼上 HTML 程式碼快速建立</span>
                    </button>
                </div>

                <!-- 模式 A：檔案拖曳與選擇區 -->
                <div id="panel-file" class="p-6 sm:p-8">
                    <div id="drop-zone"
                         class="border-2 border-dashed border-slate-300 dark:border-slate-700 hover:border-indigo-500 dark:hover:border-indigo-400 rounded-2xl p-8 sm:p-12 text-center transition-all duration-200 cursor-pointer bg-slate-50/50 dark:bg-slate-950/40 group">
                        <input type="file" id="file-input" accept=".html,.htm,text/html" multiple class="hidden">
                        
                        <div class="w-16 h-16 rounded-2xl bg-indigo-50 dark:bg-indigo-950/70 text-indigo-600 dark:text-indigo-400 flex items-center justify-center mx-auto mb-4 group-hover:scale-110 transition-transform">
                            <i class="fa-solid fa-file-arrow-up text-3xl"></i>
                        </div>
                        <h3 class="text-base sm:text-lg font-bold text-slate-800 dark:text-slate-100 mb-1">
                            將 <code class="font-mono text-indigo-600 dark:text-indigo-400 bg-indigo-50 dark:bg-indigo-950/80 px-1.5 py-0.5 rounded">.html</code> 或 <code class="font-mono text-indigo-600 dark:text-indigo-400 bg-indigo-50 dark:bg-indigo-950/80 px-1.5 py-0.5 rounded">.htm</code> 檔案拖曳至此處
                        </h3>
                        <p class="text-xs sm:text-sm text-slate-500 dark:text-slate-400 mb-5">
                            或者點擊此區域瀏覽本機檔案（支援同時選取多個網頁檔案）
                        </p>
                        <button type="button" onclick="event.stopPropagation(); document.getElementById('file-input').click();"
                                class="px-5 py-2.5 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-semibold shadow-md shadow-indigo-500/25 transition inline-flex items-center gap-2 cursor-pointer">
                            <i class="fa-regular fa-folder-open"></i>
                            <span>選擇本機檔案</span>
                        </button>
                    </div>
                </div>

                <!-- 模式 B：貼上 HTML 程式碼區 -->
                <div id="panel-code" class="hidden p-6 sm:p-8 space-y-4">
                    <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-2">
                        <label for="raw-filename" class="text-xs font-mono font-semibold text-slate-600 dark:text-slate-300">
                            檔案名稱 (Filename)
                        </label>
                        <input type="text" id="raw-filename" value="landing-page.html" placeholder="example.html"
                               class="w-full sm:w-64 px-3 py-1.5 rounded-lg bg-slate-50 dark:bg-slate-950 border border-slate-300 dark:border-slate-700 text-xs font-mono focus:outline-none focus:border-indigo-500">
                    </div>
                    <div>
                        <textarea id="raw-html-input" rows="8"
                                  placeholder="在此貼上完整的 <!DOCTYPE html> ... </html> 程式碼..."
                                  class="w-full p-4 rounded-xl bg-slate-950 text-emerald-400 font-mono text-xs sm:text-sm border border-slate-800 focus:outline-none focus:border-indigo-500 leading-relaxed"></textarea>
                    </div>
                    <div class="flex flex-wrap items-center justify-between gap-3">
                        <span class="text-xs text-slate-400">系統將自動擷取 <code class="font-mono text-indigo-400">&lt;title&gt;</code> 作為網頁標題</span>
                        <button type="button" onclick="deployRawHtml()"
                                class="px-5 py-2.5 rounded-xl bg-emerald-600 hover:bg-emerald-700 text-white text-sm font-semibold shadow-md shadow-emerald-500/20 flex items-center gap-2 transition cursor-pointer">
                            <i class="fa-solid fa-rocket"></i>
                            <span>立即生成並部署頁面</span>
                        </button>
                    </div>
                </div>

                <!-- 3. 上傳進度與即時分析反饋條 -->
                <div id="progress-container" class="hidden px-6 sm:px-8 pb-6">
                    <div class="p-4 rounded-xl bg-indigo-50/80 dark:bg-indigo-950/40 border border-indigo-200 dark:border-indigo-800/60 space-y-2.5">
                        <div class="flex items-center justify-between text-xs font-mono">
                            <span id="progress-status" class="font-semibold text-indigo-700 dark:text-indigo-300 flex items-center gap-2">
                                <i class="fa-solid fa-spinner fa-spin"></i> 正在解析 HTML 結構與標籤...
                            </span>
                            <span id="progress-percent" class="font-bold text-indigo-600 dark:text-indigo-400">0%</span>
                        </div>
                        <div class="w-full h-2 bg-slate-200 dark:bg-slate-800 rounded-full overflow-hidden">
                            <div id="progress-bar" class="h-full bg-gradient-to-r from-indigo-600 to-emerald-500 rounded-full transition-all duration-150" style="width: 0%"></div>
                        </div>
                        <div id="analysis-summary" class="hidden pt-1 text-xs text-slate-600 dark:text-slate-300 flex flex-wrap gap-3 font-mono">
                            <!-- 動態填入解析結果 -->
                        </div>
                    </div>
                </div>

                <!-- 內建一鍵載入範例模板列 -->
                <div class="bg-slate-50 dark:bg-slate-900/60 border-t border-slate-200 dark:border-slate-800 px-6 py-3.5 flex flex-wrap items-center justify-between gap-3">
                    <span class="text-xs font-medium text-slate-500 dark:text-slate-400 flex items-center gap-1.5">
                        <i class="fa-solid fa-wand-magic-sparkles text-indigo-500"></i>
                        手邊沒有檔案？一鍵載入範例模板測試：
                    </span>
                    <div class="flex flex-wrap gap-2">
                        <button type="button" onclick="loadSampleTemplate('profile')"
                                class="px-3 py-1.5 rounded-lg bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 hover:border-indigo-500 dark:hover:border-indigo-400 text-xs font-medium text-slate-700 dark:text-slate-200 flex items-center gap-1.5 transition cursor-pointer shadow-xs">
                            <span>🪪 極簡個人名片頁</span>
                        </button>
                        <button type="button" onclick="loadSampleTemplate('todo')"
                                class="px-3 py-1.5 rounded-lg bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 hover:border-emerald-500 dark:hover:border-emerald-400 text-xs font-medium text-slate-700 dark:text-slate-200 flex items-center gap-1.5 transition cursor-pointer shadow-xs">
                            <span>✅ 互動式待辦清單</span>
                        </button>
                    </div>
                </div>
            </div>
        </section>

        <!-- 4. 已上傳 HTML 檔案清單區 -->
        <section class="space-y-5">
            <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3 border-b border-slate-200 dark:border-slate-800 pb-4">
                <div class="flex items-center gap-3">
                    <h2 class="text-xl sm:text-2xl font-bold text-slate-900 dark:text-white">
                        已託管的 HTML 頁面
                    </h2>
                    <span id="file-count-badge" class="px-2.5 py-0.5 rounded-full text-xs font-mono font-bold bg-indigo-100 dark:bg-indigo-950 text-indigo-700 dark:text-indigo-300">
                        0 個檔案
                    </span>
                </div>

                <!-- 搜尋過濾框 -->
                <div class="relative w-full sm:w-64">
                    <i class="fa-solid fa-magnifying-glass absolute left-3.5 top-1/2 -translate-y-1/2 text-slate-400 text-xs"></i>
                    <input type="text" id="search-input" oninput="renderList()" placeholder="搜尋標題或檔名..."
                           class="w-full pl-9 pr-3.5 py-2 rounded-xl bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 text-xs sm:text-sm focus:outline-none focus:border-indigo-500">
                </div>
            </div>

            <!-- 卡片網格佈局 -->
            <div id="pages-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                <!-- 動態渲染卡片 -->
            </div>

            <!-- 空狀態提示 -->
            <div id="empty-state" class="hidden rounded-2xl border border-dashed border-slate-300 dark:border-slate-800 bg-white/50 dark:bg-slate-900/30 p-12 text-center space-y-3">
                <div class="w-14 h-14 rounded-2xl bg-slate-100 dark:bg-slate-800 text-slate-400 flex items-center justify-center mx-auto text-2xl">
                    <i class="fa-regular fa-folder-open"></i>
                </div>
                <h3 class="text-base font-bold text-slate-700 dark:text-slate-200">目前尚未上傳任何 HTML 頁面</h3>
                <p class="text-xs sm:text-sm text-slate-500 dark:text-slate-400 max-w-md mx-auto">
                    將本機的 `.html` 檔案拖曳到上方區塊，或直接點擊「極簡個人名片頁」、「互動式待辦清單」立即體驗即時託管預覽！
                </p>
            </div>
        </section>

    </main>

    <!-- 頁尾 -->
    <footer class="border-t border-slate-200 dark:border-slate-800/80 py-6 text-center text-xs text-slate-500 dark:text-slate-400 font-mono">
        HTMLDrop Static Hosting Hub • Powered by Browser Sandbox & LocalStorage
    </footer>

    <!-- 全螢幕即時預覽 Modal (支援切換桌面 / 手機視角) -->
    <div id="preview-modal" class="fixed inset-0 z-50 hidden flex-col bg-slate-950/90 backdrop-blur-md">
        <!-- Modal 頂部控制列 -->
        <div class="h-14 px-4 sm:px-6 bg-slate-900 border-b border-slate-800 flex items-center justify-between gap-4 shrink-0">
            <div class="flex items-center gap-3 min-w-0">
                <span class="w-2.5 h-2.5 rounded-full bg-emerald-400 shrink-0"></span>
                <h3 id="preview-modal-title" class="text-sm sm:text-base font-bold text-white truncate">網頁標題</h3>
                <span id="preview-modal-filename" class="hidden md:inline-block text-xs font-mono text-slate-400 bg-slate-800 px-2 py-0.5 rounded">index.html</span>
            </div>

            <!-- 裝置視角切換器 -->
            <div class="flex items-center bg-slate-800 p-1 rounded-xl border border-slate-700">
                <button type="button" id="view-desktop-btn" onclick="setViewport('desktop')"
                        class="px-3 py-1 rounded-lg text-xs font-mono font-semibold bg-indigo-600 text-white flex items-center gap-1.5 transition cursor-pointer">
                    <i class="fa-solid fa-desktop"></i>
                    <span class="hidden sm:inline">桌面視角</span>
                </button>
                <button type="button" id="view-mobile-btn" onclick="setViewport('mobile')"
                        class="px-3 py-1 rounded-lg text-xs font-mono font-semibold text-slate-300 hover:text-white flex items-center gap-1.5 transition cursor-pointer">
                    <i class="fa-solid fa-mobile-screen-button"></i>
                    <span class="hidden sm:inline">手機視角 (375px)</span>
                </button>
            </div>

            <!-- 右側動作與關閉 -->
            <div class="flex items-center gap-2">
                <button type="button" onclick="downloadCurrentPreview()"
                        class="px-3 py-1.5 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-200 text-xs font-mono flex items-center gap-1.5 border border-slate-700 transition cursor-pointer"
                        title="下載此 HTML 檔案">
                    <i class="fa-solid fa-download"></i>
                    <span class="hidden sm:inline">下載</span>
                </button>
                <button type="button" onclick="closePreviewModal()"
                        class="px-3 py-1.5 rounded-xl bg-rose-600 hover:bg-rose-700 text-white text-xs font-mono font-bold flex items-center gap-1.5 transition cursor-pointer">
                    <i class="fa-solid fa-xmark"></i>
                    <span>關閉</span>
                </button>
            </div>
        </div>

        <!-- iframe 沙盒渲染舞台 -->
        <div class="flex-1 overflow-auto p-3 sm:p-6 flex items-center justify-center bg-slate-950/60">
            <div id="iframe-frame-container" class="w-full h-full bg-white rounded-xl shadow-2xl overflow-hidden transition-all duration-300 border border-slate-700">
                <iframe id="preview-iframe" sandbox="allow-scripts allow-forms allow-modals" class="w-full h-full border-0 bg-white"></iframe>
            </div>
        </div>
    </div>

    <!-- 自訂確認刪除 / 清除全部 Modal (取代原生 confirm) -->
    <div id="confirm-modal" class="fixed inset-0 z-50 hidden items-center justify-center p-4 bg-slate-950/75 backdrop-blur-xs">
        <div class="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-2xl max-w-md w-full p-6 shadow-2xl space-y-4">
            <div class="flex items-center gap-3 text-rose-600 dark:text-rose-400">
                <div class="w-10 h-10 rounded-xl bg-rose-100 dark:bg-rose-950/70 flex items-center justify-center text-lg">
                    <i class="fa-solid fa-triangle-exclamation"></i>
                </div>
                <h3 id="confirm-title" class="text-lg font-bold text-slate-900 dark:text-white">確認刪除</h3>
            </div>
            <p id="confirm-message" class="text-sm text-slate-600 dark:text-slate-300 leading-relaxed">
                確定要執行此操作嗎？此動作無法復原。
            </p>
            <div class="flex justify-end gap-2.5 pt-2">
                <button type="button" onclick="closeConfirmModal()"
                        class="px-4 py-2 rounded-xl border border-slate-200 dark:border-slate-700 text-slate-600 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-800 text-sm font-medium transition cursor-pointer">
                    取消
                </button>
                <button type="button" id="confirm-action-btn"
                        class="px-4 py-2 rounded-xl bg-rose-600 hover:bg-rose-700 text-white text-sm font-semibold shadow-md shadow-rose-500/20 transition cursor-pointer">
                    確定刪除
                </button>
            </div>
        </div>
    </div>

    <!-- Toast 通知提示 -->
    <div id="toast" class="fixed bottom-6 right-6 z-50 transform translate-y-24 opacity-0 transition-all duration-300 pointer-events-none">
        <div class="bg-slate-900 dark:bg-slate-800 border border-indigo-500/40 text-white px-4 py-3 rounded-xl shadow-2xl flex items-center gap-3 text-xs sm:text-sm">
            <span id="toast-icon" class="w-7 h-7 rounded-lg bg-emerald-500/20 text-emerald-400 flex items-center justify-center shrink-0">
                <i class="fa-solid fa-check"></i>
            </span>
            <div>
                <div id="toast-title" class="font-bold text-white">操作成功</div>
                <div id="toast-desc" class="text-slate-300 text-xs">已完成處理</div>
            </div>
        </div>
    </div>

    <script>
        // ==========================================
        // 狀態管理與初始化
        // ==========================================
        const STORAGE_KEY = 'htmldrop_hosted_pages_v1';
        let hostedPages = [];
        let activePreviewId = null;

        // 內建範例模板資料庫
        const SAMPLE_TEMPLATES = {
            profile: {
                filename: 'minimal-profile-card.html',
                code: `<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>林子軒 Alex Lin | 前端工程師名片</title>
<style>
  body { margin:0; font-family: 'Inter', system-ui, sans-serif; background: linear-gradient(135deg, #eef2ff 0%, #e0e7ff 100%); display:flex; align-items:center; justify-content:center; min-height:100vh; color:#1e293b; }
  .card { background:#fff; padding:40px; border-radius:24px; box-shadow:0 20px 40px rgba(79,70,229,0.12); max-width:380px; width:90%; text-align:center; }
  .avatar { width:88px; height:88px; border-radius:50%; background:linear-gradient(135deg,#4f46e5,#10b981); color:#fff; font-size:36px; display:flex; align-items:center; justify-content:center; margin:0 auto 20px; font-weight:bold; }
  h1 { margin:0 0 6px; font-size:24px; color:#0f172a; }
  .role { color:#4f46e5; font-weight:600; font-size:14px; margin-bottom:16px; }
  p { color:#64748b; font-size:14px; line-height:1.6; margin-bottom:24px; }
  .tags { display:flex; flex-wrap:wrap; justify-content:center; gap:8px; margin-bottom:24px; }
  .tag { background:#f1f5f9; color:#334155; padding:6px 12px; border-radius:999px; font-size:12px; font-weight:600; }
  .btn { display:inline-block; background:#4f46e5; color:#fff; text-decoration:none; padding:12px 28px; border-radius:12px; font-weight:600; font-size:14px; transition:0.2s; }
  .btn:hover { background:#4338ca; transform:translateY(-2px); }
</style>
</head>
<body>
  <div class="card">
    <div class="avatar">AL</div>
    <h1>林子軒 Alex Lin</h1>
    <div class="role">UI/UX 設計師 & 前端架構師</div>
    <p>熱愛打造極簡、流暢且具備高度互動性的現代化網頁應用，專注於設計系統與效能最佳化。</p>
    <div class="tags">
      <span class="tag">TypeScript</span>
      <span class="tag">Tailwind CSS</span>
      <span class="tag">Design System</span>
    </div>
    <a href="#contact" class="btn" onclick="this.textContent='感謝您的聯繫！'; return false;">與我聯繫</a>
  </div>
</body>
</html>`
            },
            todo: {
                filename: 'interactive-todo-app.html',
                code: `<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>FocusList — 極簡互動式待辦清單</title>
<style>
  body { margin:0; font-family: system-ui, sans-serif; background:#0f172a; color:#f8fafc; display:flex; justify-content:center; padding:48px 16px; }
  .app { background:#1e293b; width:100%; max-width:440px; border-radius:20px; padding:28px; box-shadow:0 20px 35px rgba(0,0,0,0.4); border:1px solid #334155; }
  h2 { margin:0 0 20px; display:flex; align-items:center; justify-content:space-between; font-size:20px; }
  .badge { background:#10b98122; color:#34d399; font-size:12px; padding:4px 10px; border-radius:99px; }
  .input-row { display:flex; gap:8px; margin-bottom:20px; }
  input { flex:1; background:#0f172a; border:1px solid #334155; color:#fff; padding:10px 14px; border-radius:10px; outline:none; }
  input:focus { border-color:#6366f1; }
  button { background:#6366f1; color:#fff; border:none; padding:10px 16px; border-radius:10px; font-weight:600; cursor:pointer; }
  ul { list-style:none; padding:0; margin:0; display:flex; flex-direction:column; gap:10px; }
  li { background:#0f172a; padding:12px 14px; border-radius:12px; display:flex; align-items:center; justify-content:space-between; cursor:pointer; border:1px solid #1e293b; }
  li.done span { text-decoration:line-through; color:#64748b; }
</style>
</head>
<body>
  <div class="app">
    <h2><span>✅ FocusList 任務清單</span><span class="badge" id="count">3 項待辦</span></h2>
    <div class="input-row">
      <input id="inp" placeholder="輸入新任務並按新增..." onkeydown="if(event.key==='Enter')add()">
      <button onclick="add()">新增</button>
    </div>
    <ul id="list">
      <li onclick="toggle(this)"><span>完成首頁介面設計稿</span><span>○</span></li>
      <li onclick="toggle(this)"><span>測試 HTMLDrop 沙盒預覽功能</span><span>○</span></li>
      <li class="done" onclick="toggle(this)"><span>部署靜態網站原型</span><span>✓</span></li>
    </ul>
  </div>
  <script>
    function updateCount() {
      const left = document.querySelectorAll('#list li:not(.done)').length;
      document.getElementById('count').textContent = left + ' 項待辦';
    }
    function toggle(el) {
      el.classList.toggle('done');
      el.lastElementChild.textContent = el.classList.contains('done') ? '✓' : '○';
      updateCount();
    }
    function add() {
      const inp = document.getElementById('inp');
      if (!inp.value.trim()) return;
      const li = document.createElement('li');
      li.onclick = function() { toggle(this); };
      li.innerHTML = '<span>' + inp.value.replace(/</g,'&lt;') + '</span><span>○</span>';
      document.getElementById('list').prepend(li);
      inp.value = '';
      updateCount();
    }
    updateCount();
  <\/script>
</body>
</html>`
            }
        };

        // 頁面載入初始化
        window.onload = function () {
            initTheme();
            loadFromStorage();
            setupDragAndDrop();
            renderList();
        };

        // ==========================================
        // 主題切換 (深色 / 淺色)
        // ==========================================
        function initTheme() {
            const savedTheme = localStorage.getItem('htmldrop_theme');
            const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
            if (savedTheme === 'dark' || (!savedTheme && prefersDark)) {
                document.documentElement.classList.add('dark');
                document.getElementById('theme-icon').className = 'fa-solid fa-sun text-amber-400';
            } else {
                document.documentElement.classList.remove('dark');
                document.getElementById('theme-icon').className = 'fa-solid fa-moon';
            }
        }

        function toggleTheme() {
            const html = document.documentElement;
            const icon = document.getElementById('theme-icon');
            if (html.classList.contains('dark')) {
                html.classList.remove('dark');
                localStorage.setItem('htmldrop_theme', 'light');
                icon.className = 'fa-solid fa-moon';
            } else {
                html.classList.add('dark');
                localStorage.setItem('htmldrop_theme', 'dark');
                icon.className = 'fa-solid fa-sun text-amber-400';
            }
        }

        // ==========================================
        // 上傳模式切換 (檔案上傳 vs 貼上程式碼)
        // ==========================================
        function switchMode(mode) {
            const tabFile = document.getElementById('tab-file');
            const tabCode = document.getElementById('tab-code');
            const panelFile = document.getElementById('panel-file');
            const panelCode = document.getElementById('panel-code');

            const activeClasses = ['bg-white', 'dark:bg-slate-800', 'text-indigo-600', 'dark:text-indigo-400', 'shadow-sm'];
            const inactiveClasses = ['text-slate-500', 'dark:text-slate-400'];

            if (mode === 'file') {
                tabFile.classList.add(...activeClasses);
                tabFile.classList.remove(...inactiveClasses);
                tabCode.classList.remove(...activeClasses);
                tabCode.classList.add(...inactiveClasses);
                panelFile.classList.remove('hidden');
                panelCode.classList.add('hidden');
            } else {
                tabCode.classList.add(...activeClasses);
                tabCode.classList.remove(...inactiveClasses);
                tabFile.classList.remove(...activeClasses);
                tabFile.classList.add(...inactiveClasses);
                panelCode.classList.remove('hidden');
                panelFile.classList.add('hidden');
            }
        }

        // ==========================================
        // 拖曳上傳與檔案讀取邏輯
        // ==========================================
        function setupDragAndDrop() {
            const dropZone = document.getElementById('drop-zone');
            const fileInput = document.getElementById('file-input');

            dropZone.addEventListener('click', () => fileInput.click());

            ['dragenter', 'dragover'].forEach(eventName => {
                dropZone.addEventListener(eventName, (e) => {
                    e.preventDefault();
                    e.stopPropagation();
                    dropZone.classList.add('drop-zone-active');
                });
            });

            ['dragleave', 'drop'].forEach(eventName => {
                dropZone.addEventListener(eventName, (e) => {
                    e.preventDefault();
                    e.stopPropagation();
                    dropZone.classList.remove('drop-zone-active');
                });
            });

            dropZone.addEventListener('drop', (e) => {
                const files = e.dataTransfer.files;
                handleFiles(files);
            });

            fileInput.addEventListener('change', (e) => {
                handleFiles(e.target.files);
                fileInput.value = ''; // 重置以允許重複上傳同檔名
            });
        }

        function handleFiles(fileList) {
            if (!fileList || fileList.length === 0) return;

            const validFiles = Array.from(fileList).filter(file => {
                const name = file.name.toLowerCase();
                return name.endsWith('.html') || name.endsWith('.htm') || file.type === 'text/html';
            });

            if (validFiles.length === 0) {
                showToast('檔案格式不符', '請選擇 .html 或 .htm 格式的網頁檔案', 'error');
                return;
            }

            validFiles.forEach(file => {
                const reader = new FileReader();
                reader.onload = function (e) {
                    processAndSaveHtml(file.name, e.target.result);
                };
                reader.readAsText(file, 'UTF-8');
            });
        }

        // 貼上 HTML 程式碼部署
        function deployRawHtml() {
            const rawCode = document.getElementById('raw-html-input').value.trim();
            let filename = document.getElementById('raw-filename').value.trim() || 'untitled.html';

            if (!rawCode) {
                showToast('內容為空', '請先貼上或輸入 HTML 程式碼', 'error');
                return;
            }
            if (!filename.toLowerCase().endsWith('.html') && !filename.toLowerCase().endsWith('.htm')) {
                filename += '.html';
            }

            processAndSaveHtml(filename, rawCode);
            document.getElementById('raw-html-input').value = '';
        }

        // 載入內建範例模板
        function loadSampleTemplate(key) {
            const sample = SAMPLE_TEMPLATES[key];
            if (!sample) return;
            processAndSaveHtml(sample.filename, sample.code);
        }

        // ==========================================
        // HTML 解析、進度條動畫與存入 LocalStorage
        // ==========================================
        function processAndSaveHtml(filename, htmlContent) {
            const progressContainer = document.getElementById('progress-container');
            const progressBar = document.getElementById('progress-bar');
            const progressPercent = document.getElementById('progress-percent');
            const progressStatus = document.getElementById('progress-status');
            const analysisSummary = document.getElementById('analysis-summary');

            progressContainer.classList.remove('hidden');
            analysisSummary.classList.add('hidden');
            progressBar.style.width = '15%';
            progressPercent.textContent = '15%';
            progressStatus.innerHTML = '<i class="fa-solid fa-spinner fa-spin"></i> 正在解析 DOM 結構與詮釋資料...';

            // 解析 HTML 標題與標籤數量
            const parser = new DOMParser();
            const doc = parser.parseFromString(htmlContent, 'text/html');
            const parsedTitle = (doc.querySelector('title')?.textContent || '').trim() || filename.replace(/\.(html|htm)$/i, '');
            const tagCount = doc.getElementsByTagName('*').length;
            const byteSize = new Blob([htmlContent]).size;
            const sizeKB = (byteSize / 1024).toFixed(2);
            const nowStr = formatDateTime(new Date());

            // 模擬平滑進度反饋
            setTimeout(() => {
                progressBar.style.width = '65%';
                progressPercent.textContent = '65%';
                progressStatus.innerHTML = '<i class="fa-solid fa-shield-halved"></i> 正在建立沙盒預覽環境...';
            }, 180);

            setTimeout(() => {
                progressBar.style.width = '100%';
                progressPercent.textContent = '100%';
                progressStatus.innerHTML = '<i class="fa-solid fa-circle-check text-emerald-500"></i> 部署完成！已儲存至本地託管清單';

                analysisSummary.innerHTML = `
                    <span class="px-2 py-0.5 rounded bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-700">標題: <strong>${escapeHtml(parsedTitle)}</strong></span>
                    <span class="px-2 py-0.5 rounded bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-700">大小: <strong>${sizeKB} KB</strong></span>
                    <span class="px-2 py-0.5 rounded bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-700">HTML 標籤數: <strong>${tagCount} 個</strong></span>
                    <span class="px-2 py-0.5 rounded bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-700">時間: <strong>${nowStr}</strong></span>
                `;
                analysisSummary.classList.remove('hidden');

                const newItem = {
                    id: 'page_' + Date.now() + '_' + Math.random().toString(36).substring(2, 6),
                    title: parsedTitle,
                    filename: filename,
                    sizeKB: sizeKB,
                    tagCount: tagCount,
                    uploadedAt: nowStr,
                    content: htmlContent
                };

                hostedPages.unshift(newItem);
                saveToStorage();
                renderList();
                showToast('上傳與部署成功', `「${parsedTitle}」已準備好進行即時預覽`);
            }, 420);
        }

        // ==========================================
        // LocalStorage 讀寫與容量計算
        // ==========================================
        function loadFromStorage() {
            try {
                const raw = localStorage.getItem(STORAGE_KEY);
                hostedPages = raw ? JSON.parse(raw) : [];
            } catch (e) {
                console.error('讀取 LocalStorage 失敗:', e);
                hostedPages = [];
            }
            updateStorageIndicator();
        }

        function saveToStorage() {
            try {
                localStorage.setItem(STORAGE_KEY, JSON.stringify(hostedPages));
            } catch (e) {
                showToast('儲存空間已滿', '瀏覽器 LocalStorage 容量不足，請刪除部分舊檔案', 'error');
            }
            updateStorageIndicator();
        }

        function updateStorageIndicator() {
            const raw = localStorage.getItem(STORAGE_KEY) || '';
            const kb = (new Blob([raw]).size / 1024).toFixed(1);
            document.getElementById('storage-usage').textContent = `${kb} KB`;
        }

        // ==========================================
        // 渲染已上傳 HTML 卡片網格
        // ==========================================
        function renderList() {
            const grid = document.getElementById('pages-grid');
            const emptyState = document.getElementById('empty-state');
            const badge = document.getElementById('file-count-badge');
            const keyword = document.getElementById('search-input').value.trim().toLowerCase();

            const filtered = hostedPages.filter(item =>
                item.title.toLowerCase().includes(keyword) ||
                item.filename.toLowerCase().includes(keyword)
            );

            badge.textContent = `${hostedPages.length} 個檔案`;

            if (filtered.length === 0) {
                grid.innerHTML = '';
                emptyState.classList.remove('hidden');
                return;
            }

            emptyState.classList.add('hidden');
            grid.innerHTML = filtered.map(item => `
                <article class="group bg-white dark:bg-slate-900 rounded-2xl border border-slate-200 dark:border-slate-800 hover:border-indigo-500/60 dark:hover:border-indigo-400/60 shadow-sm hover:shadow-xl hover:shadow-indigo-500/5 transition-all duration-200 overflow-hidden flex flex-col">
                    <!-- 迷你沙盒縮圖預覽區 (點擊亦可開啟全螢幕預覽) -->
                    <div onclick="openPreviewModal('${item.id}')" class="thumbnail-wrapper border-b border-slate-200 dark:border-slate-800 cursor-pointer relative">
                        <iframe sandbox="allow-scripts" srcdoc="${escapeAttr(item.content)}" class="thumbnail-iframe" loading="lazy" title="${escapeAttr(item.title)}"></iframe>
                        <!-- 懸停遮罩提示 -->
                        <div class="absolute inset-0 bg-slate-950/0 group-hover:bg-slate-950/40 transition-colors flex items-center justify-center">
                            <span class="opacity-0 group-hover:opacity-100 transform translate-y-2 group-hover:translate-y-0 transition-all px-3.5 py-1.5 rounded-xl bg-indigo-600 text-white text-xs font-semibold shadow-lg flex items-center gap-1.5">
                                <i class="fa-solid fa-expand"></i> 點擊全螢幕預覽
                            </span>
                        </div>
                        <span class="absolute top-2.5 right-2.5 px-2 py-0.5 rounded-md bg-slate-900/80 backdrop-blur-xs text-white font-mono text-[10px]">
                            ${item.sizeKB} KB
                        </span>
                    </div>

                    <!-- 卡片資訊內容 -->
                    <div class="p-5 flex-1 flex flex-col justify-between space-y-4">
                        <div class="space-y-1.5">
                            <div class="flex items-center justify-between gap-2 text-[11px] font-mono text-slate-400">
                                <span class="truncate text-indigo-600 dark:text-indigo-400 font-medium">
                                    <i class="fa-regular fa-file-code mr-1"></i>${escapeHtml(item.filename)}
                                </span>
                                <span class="shrink-0">${item.tagCount} tags</span>
                            </div>
                            <h3 class="font-bold text-base text-slate-900 dark:text-white line-clamp-1" title="${escapeAttr(item.title)}">
                                ${escapeHtml(item.title)}
                            </h3>
                            <p class="text-xs text-slate-400 font-mono">
                                <i class="fa-regular fa-clock mr-1"></i>${item.uploadedAt}
                            </p>
                        </div>

                        <!-- 三個操作按鈕：全螢幕預覽 / 複製連結或下載 / 刪除 -->
                        <div class="pt-3 border-t border-slate-100 dark:border-slate-800 flex items-center gap-2">
                            <button type="button" onclick="openPreviewModal('${item.id}')"
                                    class="flex-1 py-2 px-3 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white text-xs font-semibold flex items-center justify-center gap-1.5 transition cursor-pointer">
                                <i class="fa-solid fa-eye"></i>
                                <span>即時預覽</span>
                            </button>

                            <button type="button" onclick="copyShareDataUrl('${item.id}')"
                                    class="p-2 rounded-xl bg-slate-100 dark:bg-slate-800 hover:bg-emerald-500/15 text-slate-600 dark:text-slate-300 hover:text-emerald-600 dark:hover:text-emerald-400 border border-slate-200 dark:border-slate-700 transition cursor-pointer"
                                    title="複製 Data URL 分享連結">
                                <i class="fa-solid fa-link text-xs"></i>
                            </button>

                            <button type="button" onclick="downloadHtmlFile('${item.id}')"
                                    class="p-2 rounded-xl bg-slate-100 dark:bg-slate-800 hover:bg-indigo-500/15 text-slate-600 dark:text-slate-300 hover:text-indigo-600 dark:hover:text-indigo-400 border border-slate-200 dark:border-slate-700 transition cursor-pointer"
                                    title="下載 HTML 檔案">
                                <i class="fa-solid fa-download text-xs"></i>
                            </button>

                            <button type="button" onclick="requestDeleteItem('${item.id}')"
                                    class="p-2 rounded-xl bg-slate-100 dark:bg-slate-800 hover:bg-rose-500/15 text-slate-600 dark:text-slate-300 hover:text-rose-600 dark:hover:text-rose-400 border border-slate-200 dark:border-slate-700 transition cursor-pointer"
                                    title="刪除此頁面">
                                <i class="fa-regular fa-trash-can text-xs"></i>
                            </button>
                        </div>
                    </div>
                </article>
            `).join('');
        }

        // ==========================================
        // 全螢幕即時預覽 Modal 與視角切換
        // ==========================================
        function openPreviewModal(id) {
            const item = hostedPages.find(p => p.id === id);
            if (!item) return;

            activePreviewId = id;
            document.getElementById('preview-modal-title').textContent = item.title;
            document.getElementById('preview-modal-filename').textContent = item.filename;

            const iframe = document.getElementById('preview-iframe');
            iframe.srcdoc = item.content;

            setViewport('desktop');
            const modal = document.getElementById('preview-modal');
            modal.classList.remove('hidden');
            modal.classList.add('flex');
        }

        function closePreviewModal() {
            const modal = document.getElementById('preview-modal');
            modal.classList.add('hidden');
            modal.classList.remove('flex');
            document.getElementById('preview-iframe').srcdoc = '';
            activePreviewId = null;
        }

        function setViewport(mode) {
            const container = document.getElementById('iframe-frame-container');
            const deskBtn = document.getElementById('view-desktop-btn');
            const mobBtn = document.getElementById('view-mobile-btn');

            if (mode === 'mobile') {
                container.style.width = '375px';
                container.style.maxWidth = '100%';
                mobBtn.classList.add('bg-indigo-600', 'text-white');
                mobBtn.classList.remove('text-slate-300');
                deskBtn.classList.remove('bg-indigo-600', 'text-white');
                deskBtn.classList.add('text-slate-300');
            } else {
                container.style.width = '100%';
                container.style.maxWidth = '100%';
                deskBtn.classList.add('bg-indigo-600', 'text-white');
                deskBtn.classList.remove('text-slate-300');
                mobBtn.classList.remove('bg-indigo-600', 'text-white');
                mobBtn.classList.add('text-slate-300');
            }
        }

        function downloadCurrentPreview() {
            if (activePreviewId) downloadHtmlFile(activePreviewId);
        }

        // ==========================================
        // 複製分享連結、下載與刪除操作
        // ==========================================
        function copyShareDataUrl(id) {
            const item = hostedPages.find(p => p.id === id);
            if (!item) return;

            const dataUrl = 'data:text/html;charset=utf-8,' + encodeURIComponent(item.content);
            const textarea = document.createElement('textarea');
            textarea.value = dataUrl;
            document.body.appendChild(textarea);
            textarea.select();
            try {
                document.execCommand('copy');
                showToast('已複製分享連結', '可貼至瀏覽器網址列直接開啟的 Data URL');
            } catch (err) {
                showToast('複製失敗', '無法存取剪貼簿', 'error');
            }
            document.body.removeChild(textarea);
        }

        function downloadHtmlFile(id) {
            const item = hostedPages.find(p => p.id === id);
            if (!item) return;

            const blob = new Blob([item.content], { type: 'text/html;charset=utf-8' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = item.filename;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
            showToast('開始下載', `正在下載 ${item.filename}`);
        }

        // 自訂確認視窗邏輯
        function requestDeleteItem(id) {
            const item = hostedPages.find(p => p.id === id);
            if (!item) return;

            openConfirmModal(
                '刪除託管頁面',
                `確定要將「${item.title}」（${item.filename}）從本地託管清單中移除嗎？`,
                () => {
                    hostedPages = hostedPages.filter(p => p.id !== id);
                    saveToStorage();
                    renderList();
                    showToast('已刪除頁面', `已移除 ${item.filename}`);
                }
            );
        }

        function requestClearAll() {
            if (hostedPages.length === 0) {
                showToast('清單已是空的', '目前沒有任何需要清除的檔案');
                return;
            }
            openConfirmModal(
                '清除全部託管檔案',
                `確定要清空所有 ${hostedPages.length} 個已上傳的 HTML 檔案嗎？此操作將清除所有 LocalStorage 資料且無法復原。`,
                () => {
                    hostedPages = [];
                    saveToStorage();
                    renderList();
                    document.getElementById('progress-container').classList.add('hidden');
                    showToast('已清空所有資料', '本地儲存空間已釋放');
                }
            );
        }

        function openConfirmModal(title, message, onConfirm) {
            document.getElementById('confirm-title').textContent = title;
            document.getElementById('confirm-message').textContent = message;
            const modal = document.getElementById('confirm-modal');
            const actionBtn = document.getElementById('confirm-action-btn');

            actionBtn.onclick = () => {
                onConfirm();
                closeConfirmModal();
            };

            modal.classList.remove('hidden');
            modal.classList.add('flex');
        }

        function closeConfirmModal() {
            const modal = document.getElementById('confirm-modal');
            modal.classList.add('hidden');
            modal.classList.remove('flex');
        }

        // ESC 鍵關閉彈窗
        document.addEventListener('keydown', (e) => {
            if (e.key === 'Escape') {
                closePreviewModal();
                closeConfirmModal();
            }
        });

        // ==========================================
        // 工具函式 (Toast、日期格式、跳脫字元)
        // ==========================================
        let toastTimer = null;
        function showToast(title, desc, type = 'success') {
            const toast = document.getElementById('toast');
            const iconWrap = document.getElementById('toast-icon');
            document.getElementById('toast-title').textContent = title;
            document.getElementById('toast-desc').textContent = desc;

            if (type === 'error') {
                iconWrap.className = 'w-7 h-7 rounded-lg bg-rose-500/20 text-rose-400 flex items-center justify-center shrink-0';
                iconWrap.innerHTML = '<i class="fa-solid fa-circle-exclamation"></i>';
            } else {
                iconWrap.className = 'w-7 h-7 rounded-lg bg-emerald-500/20 text-emerald-400 flex items-center justify-center shrink-0';
                iconWrap.innerHTML = '<i class="fa-solid fa-check"></i>';
            }

            toast.classList.remove('translate-y-24', 'opacity-0');
            clearTimeout(toastTimer);
            toastTimer = setTimeout(() => {
                toast.classList.add('translate-y-24', 'opacity-0');
            }, 3000);
        }

        function formatDateTime(date) {
            const pad = n => String(n).padStart(2, '0');
            return `${date.getFullYear()}-${pad(date.getMonth() + 1)}-${pad(date.getDate())} ${pad(date.getHours())}:${pad(date.getMinutes())}`;
        }

        function escapeHtml(str) {
            return String(str)
                .replace(/&/g, '&amp;')
                .replace(/</g, '&lt;')
                .replace(/>/g, '&gt;')
                .replace(/"/g, '&quot;');
        }

        function escapeAttr(str) {
            return String(str)
                .replace(/&/g, '&amp;')
                .replace(/"/g, '&quot;');
        }
    </script>
</body>
</html>
