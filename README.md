<!DOCTYPE html>
<html lang="zh-HK">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>時序小火車：尋日、今日、聽日</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    /* 專為4歲幼童及特教設計的字體、防誤觸及低刺激視覺規範 */
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "PingFang HK", "Noto Sans HK", "Microsoft JhengHei", sans-serif;
      touch-action: manipulation;
      user-select: none;
      -webkit-user-select: none;
      -webkit-touch-callout: none;
      background-color: #F8FAFC;
    }

    /* 溫和的高亮微動動畫 (避免過度刺激與快速閃爍) */
    @keyframes gentlePulse {
      0%, 100% {
        transform: scale(1);
        box-shadow: 0 4px 20px rgba(245, 158, 11, 0.25);
      }
      50% {
        transform: scale(1.025);
        box-shadow: 0 8px 30px rgba(245, 158, 11, 0.45);
      }
    }

    .active-carriage {
      animation: gentlePulse 2.8s ease-in-out infinite;
      border-color: #F59E0B !important;
      background-color: #FFFBEB !important;
    }

    /* 拖曳中的手勢反饋 */
    .dragging-card {
      opacity: 0.6;
      transform: scale(1.08);
      cursor: grabbing;
      box-shadow: 0 16px 32px rgba(0, 0, 0, 0.15);
      z-index: 50;
    }

    .touch-proxy {
      position: fixed;
      pointer-events: none;
      z-index: 9999;
      transform: translate(-50%, -50%) scale(1.08);
      filter: drop-shadow(0 14px 24px rgba(0, 0, 0, 0.2));
      will-change: transform, left, top;
    }

    /* 柔和輕彈回位動畫 */
    .bounce-back {
      transition: transform 0.35s cubic-bezier(0.34, 1.56, 0.64, 1);
    }

    /* 輕微晃動表示重試 (溫和不具處罰感) */
    @keyframes softNudge {
      0%, 100% { transform: translateX(0); }
      30% { transform: translateX(-6px); }
      70% { transform: translateX(6px); }
    }
    .nudge-retry {
      animation: softNudge 0.32s ease-in-out;
    }

    /* 車廂軌道與連接口 */
    .train-connector::after {
      content: '';
      position: absolute;
      right: -18px;
      top: 50%;
      width: 18px;
      height: 8px;
      background-color: #94A3B8;
      border-radius: 4px;
      transform: translateY(-50%);
    }

    /* 綠色大打勾成功標記 */
    .checkmark-pop {
      animation: popIn 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
    }
    @keyframes popIn {
      0% { transform: scale(0.4) rotate(-15deg); opacity: 0; }
      100% { transform: scale(1) rotate(0); opacity: 1; }
    }
  </style>
</head>
<body class="min-h-screen text-slate-800 flex flex-col justify-between overflow-x-hidden p-3 md:p-6 select-none">

  <!-- 頂部：導師工具列與仿說指引 -->
  <header class="w-full max-w-6xl mx-auto flex flex-col md:flex-row items-center justify-between gap-2.5 pb-2 border-b border-slate-200">
    <div class="flex items-center gap-3 w-full md:w-auto justify-between md:justify-start">
      <div class="flex items-center gap-2">
        <span class="text-2xl" role="img" aria-label="train">🚂</span>
        <div>
          <h1 class="text-xl md:text-2xl font-black text-slate-800 tracking-wide">時序小火車</h1>
          <p class="text-xs text-slate-500 font-medium">幼兒個別化訓練教材（4歲時序概念）</p>
        </div>
      </div>
      
      <!-- 重新發音與重試控制按鈕 -->
      <div class="flex items-center gap-2">
        <button id="btn-replay-audio" class="p-2.5 px-3.5 bg-amber-100 hover:bg-amber-200 active:scale-95 text-amber-900 rounded-2xl flex items-center gap-1.5 font-bold text-sm transition-all shadow-sm border border-amber-200">
          <svg class="w-5 h-5 text-amber-700" fill="currentColor" viewBox="0 0 20 20">
            <path fill-rule="evenodd" d="M9.383 3.076A1 1 0 0110 4v12a1 1 0 01-1.617.794L4.82 13H2a1 1 0 01-1-1V8a1 1 0 011-1h2.82l3.564-3.794A1 1 0 019.383 3.076zm5.825 2.131a1 1 0 011.414 0 9.972 9.972 0 010 14.108 1 1 0 01-1.414-1.414 7.972 7.972 0 000-11.28 1 1 0 010-1.414zm-2.828 2.828a1 1 0 011.414 0 5.986 5.986 0 010 8.465 1 1 0 01-1.414-1.414 3.986 3.986 0 000-5.637 1 1 0 010-1.414z" clip-rule="evenodd"></path>
          </svg>
          <span>讀多次</span>
        </button>
        <button id="btn-reset-all" class="p-2.5 px-3 bg-slate-100 hover:bg-slate-200 active:scale-95 text-slate-600 rounded-2xl text-xs font-semibold transition-all border border-slate-200">
          重新開始
        </button>
      </div>
    </div>

    <!-- 導師個別訓練口語鷹架提點板 (低刺激、專業指引) -->
    <div id="tutor-prompt-card" class="w-full md:w-auto bg-blue-50/90 border border-blue-200 rounded-2xl px-4 py-2 flex items-center gap-2 text-blue-900 shadow-sm">
      <span class="inline-block px-2 py-0.5 text-xs font-black bg-blue-600 text-white rounded-md tracking-wider">導師指引</span>
      <span id="tutor-speech-prompt" class="text-sm md:text-base font-bold">請按「讀多次」或等待學生操作...</span>
    </div>
  </header>

  <!-- 中間主要工作區：時間軸三節車廂 -->
  <main class="w-full max-w-6xl mx-auto flex-1 flex flex-col justify-around py-3">
    
    <!-- 語音文字即時看板 (超大字型、嚴格3-5字) -->
    <div class="text-center my-2">
      <div id="current-spoken-phrase" class="inline-flex items-center gap-2.5 px-6 py-2.5 bg-amber-50 rounded-full border-2 border-amber-300 shadow-sm transition-all duration-300">
        <span class="w-3.5 h-3.5 rounded-full bg-amber-500 animate-ping"></span>
        <span id="instruction-text" class="text-2xl md:text-3xl font-black text-amber-950 tracking-wider">今日玩積木</span>
      </div>
    </div>

    <!-- 軌道與車廂區域 -->
    <div class="relative w-full">
      <!-- 鐵路軌道底紋 -->
      <div class="absolute left-4 right-4 top-1/2 -translate-y-1/2 h-4 bg-slate-300 rounded-full flex justify-between px-8 items-center -z-0">
        <div class="w-full h-1 bg-slate-400"></div>
      </div>

      <!-- 三節火車車廂 (尋日 -> 今日 -> 聽日) -->
      <div class="grid grid-cols-3 gap-2.5 sm:gap-4 md:gap-6 relative z-10">
        
        <!-- 左車廂：尋日 (上一站) -->
        <div id="carriage-yesterday" data-slot="yesterday" class="carriage relative flex flex-col items-center bg-white border-4 border-slate-300 rounded-3xl p-2.5 sm:p-4 shadow-md transition-all duration-300 min-h-[220px] sm:min-h-[260px] md:min-h-[280px]">
          <!-- 站點標籤 -->
          <div class="w-full flex items-center justify-between border-b border-slate-150 pb-1.5 mb-2">
            <span class="px-2 py-0.5 text-xs sm:text-sm font-bold bg-slate-100 text-slate-500 rounded-lg">上一站</span>
            <span class="text-lg sm:text-2xl font-black text-slate-700">尋日</span>
          </div>

          <!-- 放置區 -->
          <div class="drop-zone w-full flex-1 flex flex-col items-center justify-center border-2 border-dashed border-slate-300 rounded-2xl bg-slate-50/70 p-2 transition-colors relative">
            <div class="placeholder-hint flex flex-col items-center text-slate-400 text-center pointer-events-none">
              <svg class="w-10 h-10 mb-1 opacity-40 text-slate-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v3m0 0v3m0-3h3m-3 0H9m12 0a9 9 0 11-18 0 9 9 0 0118 0z"></path>
              </svg>
              <span class="text-xs sm:text-sm font-bold">放入尋日事件</span>
            </div>
            <!-- 正確放置後掛載的卡片容器 -->
            <div class="placed-card-container hidden w-full h-full flex flex-col items-center justify-center"></div>
          </div>

          <!-- 輪子裝飾 -->
          <div class="w-full flex justify-around mt-2">
            <div class="w-7 h-7 sm:w-9 sm:h-9 bg-slate-700 rounded-full border-4 border-slate-300 flex items-center justify-center shadow-inner"><div class="w-2.5 h-2.5 bg-slate-200 rounded-full"></div></div>
            <div class="w-7 h-7 sm:w-9 sm:h-9 bg-slate-700 rounded-full border-4 border-slate-300 flex items-center justify-center shadow-inner"><div class="w-2.5 h-2.5 bg-slate-200 rounded-full"></div></div>
          </div>
        </div>

        <!-- 中間車廂：今日 (現在站 - 粗黃色外框醒目標示) -->
        <div id="carriage-today" data-slot="today" class="carriage relative flex flex-col items-center bg-amber-50/50 border-4 border-amber-400 active-carriage rounded-3xl p-2.5 sm:p-4 shadow-lg transition-all duration-300 min-h-[220px] sm:min-h-[260px] md:min-h-[280px]">
          <!-- 站點標籤 -->
          <div class="w-full flex items-center justify-between border-b border-amber-200 pb-1.5 mb-2">
            <span class="px-2 py-0.5 text-xs sm:text-sm font-black bg-amber-400 text-amber-950 rounded-lg shadow-sm">★ 現在站</span>
            <span class="text-xl sm:text-3xl font-black text-amber-900">今日</span>
          </div>

          <!-- 放置區 -->
          <div class="drop-zone w-full flex-1 flex flex-col items-center justify-center border-2 border-dashed border-amber-400 rounded-2xl bg-amber-100/50 p-2 transition-colors relative">
            <div class="placeholder-hint flex flex-col items-center text-amber-700 text-center pointer-events-none">
              <svg class="w-10 h-10 mb-1 opacity-70 text-amber-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 9v3m0 0v3m0-3h3m-3 0H9m12 0a9 9 0 11-18 0 9 9 0 0118 0z"></path>
              </svg>
              <span class="text-xs sm:text-sm font-bold text-amber-800">放入今日事件</span>
            </div>
            <!-- 正確放置後掛載的卡片容器 -->
            <div class="placed-card-container hidden w-full h-full flex flex-col items-center justify-center"></div>
          </div>

          <!-- 輪子裝飾 -->
          <div class="w-full flex justify-around mt-2">
            <div class="w-7 h-7 sm:w-9 sm:h-9 bg-amber-800 rounded-full border-4 border-amber-300 flex items-center justify-center shadow-inner"><div class="w-2.5 h-2.5 bg-amber-100 rounded-full"></div></div>
            <div class="w-7 h-7 sm:w-9 sm:h-9 bg-amber-800 rounded-full border-4 border-amber-300 flex items-center justify-center shadow-inner"><div class="w-2.5 h-2.5 bg-amber-100 rounded-full"></div></div>
          </div>
        </div>

        <!-- 右車廂：聽日 (下一站) -->
        <div id="carriage-tomorrow" data-slot="tomorrow" class="carriage relative flex flex-col items-center bg-white border-4 border-slate-300 rounded-3xl p-2.5 sm:p-4 shadow-md transition-all duration-300 min-h-[220px] sm:min-h-[260px] md:min-h-[280px]">
          <!-- 站點標籤 -->
          <div class="w-full flex items-center justify-between border-b border-slate-150 pb-1.5 mb-2">
            <span class="px-2 py-0.5 text-xs sm:text-sm font-bold bg-slate-100 text-slate-500 rounded-lg">下一站</span>
            <span class="text-lg sm:text-2xl font-black text-slate-700">聽日</span>
          </div>

          <!-- 放置區 -->
          <div class="drop-zone w-full flex-1 flex flex-col items-center justify-center border-2 border-dashed border-slate-300 rounded-2xl bg-slate-50/70 p-2 transition-colors relative">
            <div class="placeholder-hint flex flex-col items-center text-slate-400 text-center pointer-events-none">
              <svg class="w-10 h-10 mb-1 opacity-40 text-slate-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v3m0 0v3m0-3h3m-3 0H9m12 0a9 9 0 11-18 0 9 9 0 0118 0z"></path>
              </svg>
              <span class="text-xs sm:text-sm font-bold">放入聽日事件</span>
            </div>
            <!-- 正確放置後掛載的卡片容器 -->
            <div class="placed-card-container hidden w-full h-full flex flex-col items-center justify-center"></div>
          </div>

          <!-- 輪子裝飾 -->
          <div class="w-full flex justify-around mt-2">
            <div class="w-7 h-7 sm:w-9 sm:h-9 bg-slate-700 rounded-full border-4 border-slate-300 flex items-center justify-center shadow-inner"><div class="w-2.5 h-2.5 bg-slate-200 rounded-full"></div></div>
            <div class="w-7 h-7 sm:w-9 sm:h-9 bg-slate-700 rounded-full border-4 border-slate-300 flex items-center justify-center shadow-inner"><div class="w-2.5 h-2.5 bg-slate-200 rounded-full"></div></div>
          </div>
        </div>

      </div>
    </div>

    <!-- 底部操作區：三大高對比去背實物事件圖卡 (支援點擊選取或拖曳) -->
    <div class="mt-4 pt-3 border-t-2 border-slate-100">
      <div class="flex items-center justify-between mb-2">
        <p class="text-xs md:text-sm font-bold text-slate-500">
          💡 操作方式：可以<span class="text-amber-600 font-extrabold">按住拖上去</span>，或者<span class="text-blue-600 font-extrabold">點一下卡片再點車廂</span>
        </p>
        <span id="selected-status-badge" class="hidden text-xs font-bold text-blue-700 bg-blue-100 px-3 py-1 rounded-full animate-pulse">
          已選取卡片，請點目標車廂
        </span>
      </div>

      <div id="cards-container" class="grid grid-cols-3 gap-3 sm:gap-6 max-w-4xl mx-auto">
        
        <!-- 卡片 1：雨傘 (尋日落大雨) -->
        <div id="card-umbrella" 
             data-type="yesterday" 
             data-name="雨傘" 
             data-voice="尋日落大雨"
             class="item-card bg-white border-2 border-slate-200 hover:border-blue-400 rounded-3xl p-3 sm:p-5 flex flex-col items-center justify-center shadow-md cursor-grab active:cursor-grabbing transform transition-all duration-200 hover:shadow-lg touch-none">
          <div class="w-20 h-20 sm:w-28 sm:h-28 md:w-32 md:h-32 flex items-center justify-center pointer-events-none">
            <!-- 清楚去背風格高對比 SVG：雨傘與雨滴 -->
            <svg class="w-full h-full drop-shadow-md" viewBox="0 0 120 120" fill="none">
              <!-- 雨滴 -->
              <circle cx="28" cy="22" r="3.5" fill="#38BDF8"/>
              <circle cx="92" cy="26" r="3.5" fill="#38BDF8"/>
              <circle cx="60" cy="12" r="3.5" fill="#38BDF8"/>
              <!-- 傘頂天線 -->
              <path d="M60 22V16" stroke="#475569" stroke-width="4" stroke-linecap="round"/>
              <!-- 傘面 -->
              <path d="M18 64C18 40.804 36.804 22 60 22C83.196 22 102 40.804 102 64C102 65 101.2 66 100 66C88 66 84 59 74 59C64 59 60 66 60 66C60 66 56 59 46 59C36 59 32 66 20 66C18.8 66 18 65 18 64Z" fill="#0284C7"/>
              <path d="M38 27C44.5 37 47 50 46 59C36 59 32 66 20 66C18.8 66 18 65 18 64C18 40.8 36.8 22 60 22C51.5 22 43.8 24 38 27Z" fill="#38BDF8"/>
              <!-- 傘柄 -->
              <path d="M60 64V94C60 100 54 103 49 99" stroke="#E2E8F0" stroke-width="5" stroke-linecap="round"/>
              <path d="M60 64V94C60 100 54 103 49 99" stroke="#334155" stroke-width="4" stroke-linecap="round"/>
            </svg>
          </div>
          <span class="mt-2 text-base sm:text-xl font-black text-slate-800 tracking-wide pointer-events-none">雨傘</span>
          <span class="text-xs font-semibold text-slate-400 pointer-events-none">落大雨</span>
        </div>

        <!-- 卡片 2：積木 (今日玩積木) -->
        <div id="card-blocks" 
             data-type="today" 
             data-name="積木" 
             data-voice="今日玩積木"
             class="item-card bg-white border-2 border-slate-200 hover:border-amber-400 rounded-3xl p-3 sm:p-5 flex flex-col items-center justify-center shadow-md cursor-grab active:cursor-grabbing transform transition-all duration-200 hover:shadow-lg touch-none">
          <div class="w-20 h-20 sm:w-28 sm:h-28 md:w-32 md:h-32 flex items-center justify-center pointer-events-none">
            <!-- 清楚去背風格高對比 SVG：幼兒積木 -->
            <svg class="w-full h-full drop-shadow-md" viewBox="0 0 120 120" fill="none">
              <!-- 底層紅色積木 -->
              <rect x="18" y="66" width="38" height="34" rx="4" fill="#EF4444"/>
              <rect x="22" y="70" width="30" height="26" rx="2" fill="#F87171"/>
              <circle cx="28" cy="76" r="3" fill="#FEE2E2"/>
              <circle cx="46" cy="76" r="3" fill="#FEE2E2"/>

              <!-- 底層黃色長積木 -->
              <rect x="62" y="66" width="40" height="34" rx="4" fill="#F59E0B"/>
              <rect x="66" y="70" width="32" height="26" rx="2" fill="#FBBF24"/>
              <circle cx="74" cy="76" r="3" fill="#FEF3C7"/>
              <circle cx="90" cy="76" r="3" fill="#FEF3C7"/>

              <!-- 上層藍色拱頂/三角積木 -->
              <path d="M38 64H82L60 26L38 64Z" fill="#3B82F6"/>
              <path d="M44 60H76L60 32L44 60Z" fill="#60A5FA"/>
              <circle cx="60" cy="48" r="4" fill="#EFF6FF"/>
            </svg>
          </div>
          <span class="mt-2 text-base sm:text-xl font-black text-slate-800 tracking-wide pointer-events-none">積木</span>
          <span class="text-xs font-semibold text-slate-400 pointer-events-none">玩積木</span>
        </div>

        <!-- 卡片 3：生日蛋糕 (聽日食蛋糕) -->
        <div id="card-cake" 
             data-type="tomorrow" 
             data-name="生日蛋糕" 
             data-voice="聽日食蛋糕"
             class="item-card bg-white border-2 border-slate-200 hover:border-pink-400 rounded-3xl p-3 sm:p-5 flex flex-col items-center justify-center shadow-md cursor-grab active:cursor-grabbing transform transition-all duration-200 hover:shadow-lg touch-none">
          <div class="w-20 h-20 sm:w-28 sm:h-28 md:w-32 md:h-32 flex items-center justify-center pointer-events-none">
            <!-- 清楚去背風格高對比 SVG：生日蛋糕與蠟燭 -->
            <svg class="w-full h-full drop-shadow-md" viewBox="0 0 120 120" fill="none">
              <!-- 盤子 -->
              <ellipse cx="60" cy="98" rx="44" ry="8" fill="#CBD5E1"/>
              
              <!-- 蛋糕底層 -->
              <rect x="24" y="65" width="72" height="30" rx="6" fill="#F472B6"/>
              <!-- 奶油裝飾邊緣 -->
              <path d="M24 74C28 78 32 78 36 74C40 70 44 70 48 74C52 78 56 78 60 74C64 70 68 70 72 74C76 78 80 78 84 74C88 70 92 70 96 74V65H24V74Z" fill="#FFF1F2"/>
              
              <!-- 草莓裝飾 -->
              <circle cx="36" cy="62" r="5" fill="#E11D48"/>
              <circle cx="60" cy="62" r="5" fill="#E11D48"/>
              <circle cx="84" cy="62" r="5" fill="#E11D48"/>

              <!-- 蠟燭 -->
              <rect x="57" y="38" width="6" height="22" rx="2" fill="#38BDF8"/>
              <!-- 火焰 -->
              <path d="M60 22C64 27 65 32 60 36C55 32 56 27 60 22Z" fill="#F59E0B"/>
              <circle cx="60" cy="31" r="2" fill="#FEF08A"/>
            </svg>
          </div>
          <span class="mt-2 text-base sm:text-xl font-black text-slate-800 tracking-wide pointer-events-none">蛋糕</span>
          <span class="text-xs font-semibold text-slate-400 pointer-events-none">食蛋糕</span>
        </div>

      </div>
    </div>
  </main>

  <!-- 完成全關卡後之成就展示畫面 (低感官負荷、溫和自信激勵) -->
  <div id="completion-modal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm hidden flex items-center justify-center p-4 z-50">
    <div class="bg-white rounded-3xl p-6 sm:p-8 max-w-lg w-full text-center shadow-2xl border-4 border-amber-300 transform transition-all scale-95 opacity-0 duration-300" id="completion-modal-box">
      <div class="text-5xl sm:text-6xl mb-2 animate-bounce">🎉</div>
      <h2 class="text-2xl sm:text-3xl font-black text-slate-800 mb-2">好叻呀！完成喇！</h2>
      <p class="text-slate-600 text-base sm:text-lg font-bold mb-4">尋日、今日、聽日，時序火車出發囉！</p>
      
      <!-- 溫和回顧小火車示意 -->
      <div class="bg-amber-50 rounded-2xl p-4 border border-amber-200 mb-6 flex items-center justify-around">
        <div class="flex flex-col items-center">
          <span class="text-xs font-bold text-slate-500">尋日</span>
          <span class="text-2xl">☂️</span>
          <span class="text-xs font-semibold text-slate-700">落大雨</span>
        </div>
        <span class="text-slate-300 font-black">➔</span>
        <div class="flex flex-col items-center bg-white px-3 py-1 rounded-xl shadow-sm border border-amber-300">
          <span class="text-xs font-black text-amber-600">今日</span>
          <span class="text-2xl">🧱</span>
          <span class="text-xs font-bold text-amber-900">玩積木</span>
        </div>
        <span class="text-slate-300 font-black">➔</span>
        <div class="flex flex-col items-center">
          <span class="text-xs font-bold text-slate-500">聽日</span>
          <span class="text-2xl">🎂</span>
          <span class="text-xs font-semibold text-slate-700">食蛋糕</span>
        </div>
      </div>

      <div class="flex gap-3 justify-center">
        <button id="modal-replay-btn" class="w-full py-3.5 bg-amber-500 hover:bg-amber-600 active:scale-95 text-white font-black text-lg rounded-2xl shadow-lg transition-transform">
          再玩一次 🔄
        </button>
      </div>
    </div>
  </div>

  <script>
    /* ==========================================================================
       1. 溫和音效合成器 (Web Audio API - 無需外部音檔，溫暖正向五聲音階)
       ========================================================================== */
    class CalmSoundController {
      constructor() {
        this.ctx = null;
      }

      initCtx() {
        if (!this.ctx) {
          const AudioContext = window.AudioContext || window.webkitAudioContext;
          this.ctx = new AudioContext();
        }
        if (this.ctx.state === 'suspended') {
          this.ctx.resume();
        }
      }

      // 溫暖的成功柔和雙音（杜絕尖銳刺耳或過大音量）
      playSoftSuccess() {
        this.initCtx();
        const now = this.ctx.currentTime;
        
        // 音符 1: C5 (523.25Hz)
        const osc1 = this.ctx.createOscillator();
        const gain1 = this.ctx.createGain();
        osc1.type = 'sine';
        osc1.frequency.setValueAtTime(523.25, now);
        gain1.gain.setValueAtTime(0.001, now);
        gain1.gain.exponentialRampToValueAtTime(0.2, now + 0.05);
        gain1.gain.exponentialRampToValueAtTime(0.0001, now + 0.45);
        osc1.connect(gain1);
        gain1.connect(this.ctx.destination);
        osc1.start(now);
        osc1.stop(now + 0.45);

        // 音符 2: G5 (783.99Hz)
        const osc2 = this.ctx.createOscillator();
        const gain2 = this.ctx.createGain();
        osc2.type = 'sine';
        osc2.frequency.setValueAtTime(783.99, now + 0.12);
        gain2.gain.setValueAtTime(0.001, now + 0.12);
        gain2.gain.exponentialRampToValueAtTime(0.25, now + 0.17);
        gain2.gain.exponentialRampToValueAtTime(0.0001, now + 0.7);
        osc2.connect(gain2);
        gain2.connect(this.ctx.destination);
        osc2.start(now + 0.12);
        osc2.stop(now + 0.7);
      }

      // 完成三部曲之歡樂柔和小調
      playCelebrationFanfare() {
        this.initCtx();
        const notes = [523.25, 659.25, 783.99, 1046.50]; // C - E - G - High C
        const start = this.ctx.currentTime;
        notes.forEach((freq, idx) => {
          const osc = this.ctx.createOscillator();
          const gain = this.ctx.createGain();
          osc.type = 'triangle';
          osc.frequency.setValueAtTime(freq, start + (idx * 0.14));
          gain.gain.setValueAtTime(0.001, start + (idx * 0.14));
          gain.gain.exponentialRampToValueAtTime(0.18, start + (idx * 0.14) + 0.04);
          gain.gain.exponentialRampToValueAtTime(0.0001, start + (idx * 0.14) + 0.5);
          osc.connect(gain);
          gain.connect(this.ctx.destination);
          osc.start(start + (idx * 0.14));
          osc.stop(start + (idx * 0.14) + 0.55);
        });
      }
    }

    const soundManager = new CalmSoundController();

    /* ==========================================================================
       2. 廣東話精準口語發音模組 (嚴格3-5字限制，支援zh-HK)
       ========================================================================== */
    function speakCantonese(phrase, onEndCallback) {
      if (!('speechSynthesis' in window)) {
        if (onEndCallback) onEndCallback();
        return;
      }

      window.speechSynthesis.cancel(); // 終止前置積壓語音
      const utterance = new SpeechSynthesisUtterance(phrase);
      utterance.lang = 'zh-HK'; // 指定廣東話
      utterance.rate = 0.85;    // 專為4歲幼童稍微調緩語速，增強咬字辨析度
      utterance.pitch = 1.1;    // 柔和親和聲調

      // 檢索最佳廣東話或中文語音引擎
      const voices = window.speechSynthesis.getVoices();
      const hkVoice = voices.find(v => v.lang === 'zh-HK' || v.lang.includes('yue') || v.lang === 'zh_HK');
      if (hkVoice) {
        utterance.voice = hkVoice;
      }

      if (onEndCallback) {
        utterance.onend = onEndCallback;
      }
      window.speechSynthesis.speak(utterance);
    }

    // 確保部分瀏覽器異步加載語音清單
    if ('speechSynthesis' in window) {
      window.speechSynthesis.onvoiceschanged = () => {
        // ready voices
      };
    }

    /* ==========================================================================
       3. 步驟狀態管理與教案時序配置
       ========================================================================== */
    const GAME_STEPS = [
      {
        stepIndex: 1,
        targetKey: 'today',
        phrase: '今日玩積木',          // 嚴格5字廣東話
        tutorHint: '導師引導：指住中間車廂「今日」，問幼兒「今日做咩呀？」',
        affirmation: '係今日！',
        activeCarriageId: 'carriage-today'
      },
      {
        stepIndex: 2,
        targetKey: 'yesterday',
        phrase: '尋日落大雨',          // 嚴格5字廣東話
        tutorHint: '導師引導：指住左邊「尋日」，提示幼兒「尋日過去咗落雨」',
        affirmation: '係尋日！',
        activeCarriageId: 'carriage-yesterday'
      },
      {
        stepIndex: 3,
        targetKey: 'tomorrow',
        phrase: '聽日食蛋糕',          // 嚴格5字廣東話
        tutorHint: '導師引導：指住右邊「聽日」，引導幼兒「聽日未到去食蛋糕」',
        affirmation: '係聽日！',
        activeCarriageId: 'carriage-tomorrow'
      }
    ];

    let currentStepIdx = 0;
    let selectedItemCard = null; // 支援點選放置模式
    const completedItems = new Set();

    // DOM 元素引用
    const instructionText = document.getElementById('instruction-text');
    const tutorSpeechPrompt = document.getElementById('tutor-speech-prompt');
    const selectedStatusBadge = document.getElementById('selected-status-badge');
    const completionModal = document.getElementById('completion-modal');
    const completionModalBox = document.getElementById('completion-modal-box');
    const btnReplayAudio = document.getElementById('btn-replay-audio');
    const btnResetAll = document.getElementById('btn-reset-all');
    const modalReplayBtn = document.getElementById('modal-replay-btn');

    /* ==========================================================================
       4. 介面與鷹架狀態推進邏輯
       ========================================================================== */
    function updateGameStepVisuals(playVoice = true) {
      if (currentStepIdx >= GAME_STEPS.length) {
        showCelebration();
        return;
      }

      const currentConfig = GAME_STEPS[currentStepIdx];

      // 更新畫面文字
      instructionText.textContent = currentConfig.phrase;
      tutorSpeechPrompt.textContent = currentConfig.tutorHint;

      // 更新車廂聚焦點
      document.querySelectorAll('.carriage').forEach(el => {
        el.classList.remove('active-carriage');
      });

      const currentCarriageEl = document.getElementById(currentConfig.activeCarriageId);
      if (currentCarriageEl) {
        currentCarriageEl.classList.add('active-carriage');
      }

      // 播放廣東話指令
      if (playVoice) {
        speakCantonese(currentConfig.phrase);
      }
    }

    /* ==========================================================================
       5. 配對成功與錯誤反饋處置
       ========================================================================== */
    function handleCorrectPlacement(cardType, cardElement, targetCarriageElement) {
      const currentConfig = GAME_STEPS[currentStepIdx];
      
      // 播放輕柔成功音效
      soundManager.playSoftSuccess();

      // 移動並固化圖卡於車廂內
      const dropZone = targetCarriageElement.querySelector('.drop-zone');
      const placeholderHint = targetCarriageElement.querySelector('.placeholder-hint');
      const placedContainer = targetCarriageElement.querySelector('.placed-card-container');

      placeholderHint.classList.add('hidden');
      placedContainer.classList.remove('hidden');
      
      // 複製卡片內容至車廂內，同時添加成功勾號
      const cardClone = cardElement.cloneNode(true);
      cardClone.id = `${cardClone.id}-placed`;
      cardClone.className = "w-full h-full flex flex-col items-center justify-center p-1 pointer-events-none relative";
      
      // 加入綠色勾號徽章
      const checkBadge = document.createElement('div');
      checkBadge.className = 'checkmark-pop absolute top-1 right-2 bg-emerald-500 text-white rounded-full p-1 shadow-md';
      checkBadge.innerHTML = `<svg class="w-5 h-5 sm:w-6 sm:h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"></path></svg>`;
      cardClone.appendChild(checkBadge);

      placedContainer.innerHTML = '';
      placedContainer.appendChild(cardClone);

      // 原卡片半透明並停用
      cardElement.classList.add('opacity-20', 'pointer-events-none', 'scale-90');
      completedItems.add(cardType);

      // 清理選取狀態
      clearSelection();

      // 導師與幼兒仿說鷹架回饋（例如：「係今日！」）
      instructionText.textContent = currentConfig.affirmation;
      tutorSpeechPrompt.textContent = `仿說提點：「${currentConfig.affirmation} 一齊講一次。」`;
      
      speakCantonese(currentConfig.affirmation, () => {
        // 短暫停留後推進到下一步
        setTimeout(() => {
          currentStepIdx++;
          updateGameStepVisuals(true);
        }, 900);
      });
    }

    function handleIncorrectPlacement(cardElement) {
      // 溫和回彈與微幅搖晃，無負向刺耳聲音
      cardElement.classList.add('nudge-retry');
      setTimeout(() => {
        cardElement.classList.remove('nudge-retry');
      }, 350);

      // 觸發輔助提醒，溫和重複指令
      const currentConfig = GAME_STEPS[currentStepIdx];
      speakCantonese(currentConfig.phrase);
    }

    /* ==========================================================================
       6. 雙模式互動：支援觸控拖曳 (Touch Drag) 與 點擊選取 (Tap-to-Place)
       ========================================================================== */
    // 點擊選取邏輯 (特別適用於精細小肌肉發展中或手震幼童)
    function clearSelection() {
      if (selectedItemCard) {
        selectedItemCard.classList.remove('ring-4', 'ring-blue-400', 'bg-blue-50');
      }
      selectedItemCard = null;
      selectedStatusBadge.classList.add('hidden');
    }

    function handleCardClick(cardEl) {
      const type = cardEl.getAttribute('data-type');
      if (completedItems.has(type)) return;

      soundManager.initCtx();

      if (selectedItemCard === cardEl) {
        clearSelection();
        return;
      }

      clearSelection();
      selectedItemCard = cardEl;
      cardEl.classList.add('ring-4', 'ring-blue-400', 'bg-blue-50');
      selectedStatusBadge.classList.remove('hidden');

      // 播放卡片語音輔助確認
      const voice = cardEl.getAttribute('data-voice');
      speakCantonese(voice);
    }

    function handleCarriageClick(carriageEl) {
      if (!selectedItemCard) return;

      const currentConfig = GAME_STEPS[currentStepIdx];
      const targetSlot = carriageEl.getAttribute('data-slot');
      const cardType = selectedItemCard.getAttribute('data-type');

      if (targetSlot === currentConfig.targetKey && cardType === currentConfig.targetKey) {
        handleCorrectPlacement(cardType, selectedItemCard, carriageEl);
      } else {
        handleIncorrectPlacement(selectedItemCard);
        clearSelection();
      }
    }

    // 觸控與滑鼠拖曳邏輯 (Touch & Pointer Events with smooth Touch Proxy)
    let touchDragProxy = null;
    let activeDraggedCard = null;

    function initTouchAndDrag() {
      const cards = document.querySelectorAll('.item-card');
      const carriages = document.querySelectorAll('.carriage');

      // 點擊卡片與車廂綁定
      cards.forEach(card => {
        card.addEventListener('click', (e) => {
          handleCardClick(card);
        });

        // Pointerdown for dragging
        card.addEventListener('pointerdown', (e) => {
          const type = card.getAttribute('data-type');
          if (completedItems.has(type)) return;

          soundManager.initCtx();
          activeDraggedCard = card;

          const rect = card.getBoundingClientRect();
          touchDragProxy = card.cloneNode(true);
          touchDragProxy.id = "active-touch-proxy";
          touchDragProxy.classList.add('touch-proxy');
          touchDragProxy.style.width = `${rect.width}px`;
          touchDragProxy.style.height = `${rect.height}px`;
          touchDragProxy.style.left = `${e.clientX}px`;
          touchDragProxy.style.top = `${e.clientY}px`;
          
          document.body.appendChild(touchDragProxy);
          card.classList.add('dragging-card');
        });
      });

      window.addEventListener('pointermove', (e) => {
        if (!touchDragProxy || !activeDraggedCard) return;
        e.preventDefault();
        touchDragProxy.style.left = `${e.clientX}px`;
        touchDragProxy.style.top = `${e.clientY}px`;
      });

      window.addEventListener('pointerup', (e) => {
        if (!activeDraggedCard) return;

        const currentConfig = GAME_STEPS[currentStepIdx];
        const cardType = activeDraggedCard.getAttribute('data-type');

        // 偵測釋放點所在元素
        if (touchDragProxy) {
          touchDragProxy.remove();
          touchDragProxy = null;
        }
        activeDraggedCard.classList.remove('dragging-card');

        // 查找落在下方的車廂
        const elemBelow = document.elementFromPoint(e.clientX, e.clientY);
        const targetCarriage = elemBelow ? elemBelow.closest('.carriage') : null;

        if (targetCarriage) {
          const slot = targetCarriage.getAttribute('data-slot');
          if (slot === currentConfig.targetKey && cardType === currentConfig.targetKey) {
            handleCorrectPlacement(cardType, activeDraggedCard, targetCarriage);
          } else {
            handleIncorrectPlacement(activeDraggedCard);
          }
        }

        activeDraggedCard = null;
      });

      // 點擊車廂對應放置
      carriages.forEach(carriage => {
        carriage.addEventListener('click', () => {
          handleCarriageClick(carriage);
        });
      });
    }

    /* ==========================================================================
       7. 完成畫面展示與重新開始
       ========================================================================== */
    function showCelebration() {
      soundManager.playCelebrationFanfare();
      completionModal.classList.remove('hidden');
      setTimeout(() => {
        completionModalBox.classList.remove('scale-95', 'opacity-0');
        completionModalBox.classList.add('scale-100', 'opacity-100');
      }, 50);

      speakCantonese('好叻呀！尋日、今日、聽日！');
    }

    function resetGame() {
      currentStepIdx = 0;
      completedItems.clear();
      clearSelection();

      // 還原所有車廂
      document.querySelectorAll('.carriage').forEach(carriage => {
        const placeholderHint = carriage.querySelector('.placeholder-hint');
        const placedContainer = carriage.querySelector('.placed-card-container');
        placeholderHint.classList.remove('hidden');
        placedContainer.classList.add('hidden');
        placedContainer.innerHTML = '';
      });

      // 還原所有卡片
      document.querySelectorAll('.item-card').forEach(card => {
        card.className = "item-card bg-white border-2 border-slate-200 hover:border-amber-400 rounded-3xl p-3 sm:p-5 flex flex-col items-center justify-center shadow-md cursor-grab active:cursor-grabbing transform transition-all duration-200 hover:shadow-lg touch-none";
      });

      // 關閉完成視窗
      completionModalBox.classList.add('scale-95', 'opacity-0');
      completionModalBox.classList.remove('scale-100', 'opacity-100');
      setTimeout(() => {
        completionModal.classList.add('hidden');
      }, 250);

      // 啟動首步
      updateGameStepVisuals(true);
    }

    /* ==========================================================================
       8. 初始化與事件監聽
       ========================================================================== */
    window.addEventListener('DOMContentLoaded', () => {
      initTouchAndDrag();
      updateGameStepVisuals(false); // 初始靜態渲染

      btnReplayAudio.addEventListener('click', () => {
        soundManager.initCtx();
        if (currentStepIdx < GAME_STEPS.length) {
          speakCantonese(GAME_STEPS[currentStepIdx].phrase);
        } else {
          speakCantonese('好叻呀！尋日、今日、聽日！');
        }
      });

      btnResetAll.addEventListener('click', resetGame);
      modalReplayBtn.addEventListener('click', resetGame);

      // 初次用戶點擊時喚醒 Web Audio
      window.addEventListener('click', () => {
        soundManager.initCtx();
      }, { once: true });
    });
  </script>
</body>
</html>
