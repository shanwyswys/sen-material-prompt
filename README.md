<!DOCTYPE html>
<html lang="zh-HK">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>快樂獨體字探險記 - 平板互動配對小遊戲</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Canvas Confetti for celebratory positive reinforcement -->
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
  <!-- Google Fonts: Zen Maru Gothic / Noto Serif HK / Educational style -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Chiron+Hei+HK:wght@500;700;900&family=Noto+Sans+TC:wght@500;700;900&family=Noto+Serif+HK:wght@700;900&display=swap" rel="stylesheet">

  <style>
    body {
      font-family: 'Chiron Hei HK', 'Noto Sans TC', system-ui, -apple-system, sans-serif;
      touch-action: manipulation;
      user-select: none;
      -webkit-user-select: none;
      -webkit-tap-highlight-color: transparent;
      background-color: #FFFDF0;
    }

    /* Traditional Chinese Educational / Stroke clarity font */
    .char-font {
      font-family: 'Noto Serif HK', 'DFKai-SB', 'BiauKai', 'KaiTi', serif;
      font-weight: 900;
    }

    /* Gentle pulsing effect for current active element */
    @keyframes gentlePulse {
      0%, 100% { transform: scale(1); box-shadow: 0 4px 15px rgba(251, 146, 60, 0.25); }
      50% { transform: scale(1.05); box-shadow: 0 10px 25px rgba(251, 146, 60, 0.45); }
    }
    .pulse-selected {
      animation: gentlePulse 1.2s infinite ease-in-out;
    }

    /* Soft encouraging shake for non-match (no harsh penalties) */
    @keyframes gentleWiggle {
      0%, 100% { transform: translateX(0); }
      20% { transform: translateX(-6px) rotate(-1.5deg); }
      40% { transform: translateX(6px) rotate(1.5deg); }
      60% { transform: translateX(-4px) rotate(-1deg); }
      80% { transform: translateX(4px) rotate(1deg); }
    }
    .wiggle-hint {
      animation: gentleWiggle 0.5s ease-in-out;
    }

    /* Correct bounce */
    @keyframes happyBounce {
      0%, 100% { transform: scale(1); }
      40% { transform: scale(1.12); }
      70% { transform: scale(0.95); }
    }
    .matched-bounce {
      animation: happyBounce 0.6s cubic-bezier(0.34, 1.56, 0.64, 1);
    }

    /* Custom scrollbars */
    ::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    ::-webkit-scrollbar-thumb {
      background: #fed7aa;
      border-radius: 9999px;
    }
  </style>
</head>
<body class="min-h-screen text-slate-800 flex flex-col justify-between overflow-x-hidden">

  <header class="w-full bg-amber-100/80 backdrop-blur-md border-b-4 border-amber-300 px-4 py-2 sm:py-3 shadow-sm sticky top-0 z-30">
    <div class="max-w-6xl mx-auto flex items-center justify-between gap-2">
      <!-- Title & Mascot -->
      <div class="flex items-center gap-2 sm:gap-3">
        <span class="text-3xl sm:text-4xl filter drop-shadow">🌱</span>
        <div>
          <h1 class="text-xl sm:text-2xl font-black text-amber-900 tracking-wide flex items-center gap-2">
            快樂獨體字探險記
          </h1>
          <p class="text-xs sm:text-sm font-bold text-amber-700 hidden sm:block">
            適合 K3 幼兒及學習支援的小朋友・認形音義
          </p>
        </div>
      </div>

      <!-- Level Stars Progress Indicator -->
      <div class="flex items-center gap-2 sm:gap-4 bg-white/90 px-3 py-1.5 sm:px-4 sm:py-2 rounded-2xl shadow-inner border-2 border-amber-200">
        <div class="flex flex-col items-center">
          <span class="text-xs font-bold text-amber-700">關卡進度</span>
          <span id="level-display" class="text-base sm:text-lg font-black text-orange-600">第 1 / 8 關</span>
        </div>
        <!-- 8 stars dots -->
        <div id="stars-container" class="grid grid-cols-4 gap-1 sm:flex sm:gap-1.5">
          <!-- Populated by JS -->
        </div>
      </div>

      <!-- Controls: Fullscreen & Teacher/Parent settings -->
      <div class="flex items-center gap-2">
        <button id="btn-sound-replay" title="再聽一次廣東話提示" class="p-2 sm:px-3 sm:py-2 bg-emerald-500 hover:bg-emerald-600 active:scale-95 text-white font-bold rounded-xl shadow-md border-b-4 border-emerald-700 flex items-center gap-1.5 transition">
          <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 20 20">
            <path fill-rule="evenodd" d="M9.383 3.076A1 1 0 0110 4v12a1 1 0 01-1.617.774l-4.14-3.313H2a1 1 0 01-1-1V7.539a1 1 0 011-1h2.243l4.14-3.313a1 1 0 011-.15zm5.088 2.095a1 1 0 011.414 0 7.002 7.002 0 010 9.898 1 1 0 11-1.414-1.414 5.002 5.002 0 000-7.07 1 1 0 010-1.414zm-2.828 2.828a1 1 0 011.414 0 3 3 0 010 4.242 1 1 0 11-1.414-1.414 1 1 0 000-1.414 1 1 0 010-1.414z" clip-rule="evenodd"/>
          </svg>
          <span class="text-xs sm:text-sm hidden md:inline">廣東話提示</span>
        </button>

        <button id="btn-teacher-menu" title="教師/家長模式" class="p-2 bg-amber-400 hover:bg-amber-500 active:scale-95 text-amber-950 font-bold rounded-xl shadow-md border-b-4 border-amber-600 transition">
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"/>
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/>
          </svg>
        </button>
      </div>
    </div>
  </header>

  <main class="flex-grow max-w-6xl w-full mx-auto p-3 sm:p-5 flex flex-col justify-center">

    <!-- Top Mode Tab Bar (Step 1: 認一認 vs Step 2: 配一配) -->
    <div class="flex items-center justify-between mb-3 sm:mb-4 bg-orange-100/90 p-2 rounded-2xl border-2 border-orange-200">
      <div class="flex items-center gap-2">
        <button id="tab-step-preview" class="px-4 py-2 rounded-xl font-black text-sm sm:text-base transition bg-orange-500 text-white shadow-md flex items-center gap-1.5">
          <span>👀 步驟一：認一認・聽一聽</span>
        </button>
        <button id="tab-step-match" class="px-4 py-2 rounded-xl font-black text-sm sm:text-base transition bg-white/70 text-orange-900 hover:bg-white flex items-center gap-1.5">
          <span>🧩 步驟二：連一連・配對大挑戰</span>
        </button>
      </div>
      <div id="instruction-pill" class="text-xs sm:text-sm font-bold bg-amber-50 border border-amber-300 text-amber-900 px-3 py-1 rounded-full hidden sm:block">
        💡 點按卡片聽廣東話發音同認識字意
      </div>
    </div>

    <!-- VIEW A: Preview & Read (認一認・聽一聽) -->
    <section id="view-preview" class="w-full">
      <div class="text-center mb-4">
        <span class="inline-block bg-amber-200 text-amber-900 font-extrabold text-sm sm:text-base px-4 py-1 rounded-full shadow-sm mb-1">
          點按喇叭仔或字卡，一齊聽廣東話點讀啦！
        </span>
      </div>
      <!-- 5 Character Cards horizontally/grid wrapped -->
      <div id="preview-cards-container" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-5 gap-3 sm:gap-4">
        <!-- Rendered by JS -->
      </div>
      <!-- Quick Go To Game Button -->
      <div class="mt-6 text-center">
        <button id="btn-start-matching" class="px-8 py-3.5 bg-gradient-to-r from-orange-500 to-amber-500 hover:from-orange-600 hover:to-amber-600 active:scale-95 text-white text-lg sm:text-xl font-black rounded-2xl shadow-lg border-b-4 border-orange-700 transition flex items-center justify-center gap-3 mx-auto">
          <span>我準備好喇！開始配對 🚀</span>
        </button>
      </div>
    </section>

    <!-- VIEW B: Matching Game (連一連・配對大挑戰) -->
    <section id="view-match" class="w-full hidden">
      <!-- Status Prompt -->
      <div class="text-center mb-3">
        <div id="match-hint-banner" class="inline-flex items-center gap-2 bg-amber-100 text-amber-900 border-2 border-amber-300 px-5 py-2 rounded-2xl text-base sm:text-lg font-bold shadow-sm">
          <span>👇 請點選左邊嘅【中文字】，再點選右邊對應嘅【圖案】！</span>
        </div>
      </div>

      <!-- Split Layout for Tablet Matching: Left (5 Characters) & Right (5 Pictograms) -->
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4 sm:gap-6 bg-white/70 p-4 sm:p-6 rounded-3xl border-4 border-amber-200 shadow-xl backdrop-blur-sm">
        
        <!-- Left: Word Cards -->
        <div class="flex flex-col gap-3">
          <div class="flex items-center justify-between pb-1 border-b-2 border-amber-200">
            <span class="text-sm sm:text-base font-black text-amber-800 flex items-center gap-1.5">
              <span>✍️</span> 獨體字卡片
            </span>
            <span class="text-xs font-bold text-amber-600">點按聽字音</span>
          </div>
          <div id="char-column" class="flex flex-col gap-2.5 sm:gap-3">
            <!-- Populated by JS -->
          </div>
        </div>

        <!-- Right: Pictogram Cards -->
        <div class="flex flex-col gap-3">
          <div class="flex items-center justify-between pb-1 border-b-2 border-amber-200">
            <span class="text-sm sm:text-base font-black text-amber-800 flex items-center gap-1.5">
              <span>🎨</span> 意思與象形圖案
            </span>
            <span class="text-xs font-bold text-amber-600">點按尋找配對</span>
          </div>
          <div id="pic-column" class="flex flex-col gap-2.5 sm:gap-3">
            <!-- Populated by JS -->
          </div>
        </div>

      </div>

      <!-- Bottom controls inside match view -->
      <div class="mt-4 flex items-center justify-between">
        <button id="btn-back-to-preview" class="px-4 py-2 bg-slate-200 hover:bg-slate-300 text-slate-800 font-bold rounded-xl active:scale-95 text-xs sm:text-sm transition flex items-center gap-1">
          <span>⬅️ 返去認一認</span>
        </button>
        <div class="text-amber-800 font-black text-sm sm:text-base">
          已成功配對：<span id="matched-count" class="text-xl text-emerald-600 font-black">0</span> / 5
        </div>
        <button id="btn-reset-current-level" class="px-4 py-2 bg-amber-200 hover:bg-amber-300 text-amber-900 font-bold rounded-xl active:scale-95 text-xs sm:text-sm transition flex items-center gap-1">
          <span>🔄 重新配對</span>
        </button>
      </div>
    </section>

  </main>

  <footer class="w-full bg-amber-100/70 py-2 px-4 text-center text-xs font-bold text-amber-800 border-t border-amber-200">
    <p>幼稚園高班校本教材・自然象形獨體字認讀訓練・支援觸控平板與廣東話發音</p>
  </footer>

  <!-- Level Success Celebratory Modal -->
  <div id="modal-level-success" class="fixed inset-0 bg-black/60 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
    <div class="bg-gradient-to-b from-amber-50 to-orange-50 border-4 border-amber-400 rounded-3xl max-w-md w-full p-6 text-center shadow-2xl transform transition-all scale-100 animate-[happyBounce_0.5s]">
      <div class="text-6xl mb-2">🎉</div>
      <h2 class="text-2xl sm:text-3xl font-black text-amber-950 mb-2">太好喇！全部答啱！</h2>
      <p id="modal-success-msg" class="text-base text-amber-800 font-bold mb-5">
        你好有耐性，順利完成呢關嘅獨體字配對！
      </p>

      <!-- Earned Stamp Badge -->
      <div class="bg-white p-4 rounded-2xl border-2 border-dashed border-amber-300 mb-6 flex items-center justify-center gap-3">
        <span class="text-4xl">🌟</span>
        <div class="text-left">
          <p class="text-xs font-bold text-slate-500">獲得探索小印章</p>
          <p id="stamp-badge-title" class="text-lg font-black text-orange-600">獨體字小學士</p>
        </div>
      </div>

      <div class="flex flex-col gap-2.5">
        <button id="btn-next-level" class="w-full py-3.5 bg-emerald-500 hover:bg-emerald-600 active:scale-95 text-white text-lg font-black rounded-2xl shadow-lg border-b-4 border-emerald-700 transition">
          進入下一關 ➡️
        </button>
        <button id="btn-replay-modal" class="w-full py-2.5 bg-amber-200 hover:bg-amber-300 active:scale-95 text-amber-900 font-bold rounded-xl transition">
          重玩呢一關 🔄
        </button>
      </div>
    </div>
  </div>

  <!-- Game Complete Modal (Finished all 8 levels) -->
  <div id="modal-game-complete" class="fixed inset-0 bg-black/60 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
    <div class="bg-gradient-to-b from-yellow-50 to-amber-100 border-4 border-yellow-400 rounded-3xl max-w-lg w-full p-6 sm:p-8 text-center shadow-2xl">
      <div class="text-7xl mb-3">🏆</div>
      <h2 class="text-2xl sm:text-3xl font-black text-amber-950 mb-2">恭喜你！完成全部 8 關！</h2>
      <p class="text-base font-bold text-amber-800 mb-5">
        你已經認識哂 15 個常見獨體字嘅字形、發音同意思喇！你真係太叻喇！
      </p>
      <div class="flex justify-center gap-2 mb-6 text-3xl">
        ⭐ ⭐ ⭐ ⭐ ⭐ ⭐ ⭐ ⭐
      </div>
      <button id="btn-restart-game" class="w-full py-4 bg-orange-500 hover:bg-orange-600 active:scale-95 text-white text-xl font-black rounded-2xl shadow-xl border-b-4 border-orange-700 transition">
        由第 1 關再玩一次 🚀
      </button>
    </div>
  </div>

  <!-- Teacher / Parent Setting Drawer / Modal -->
  <div id="modal-teacher" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
    <div class="bg-white border-4 border-amber-300 rounded-3xl max-w-md w-full p-5 sm:p-6 shadow-2xl">
      <div class="flex items-center justify-between pb-3 border-b-2 border-slate-100">
        <h3 class="text-lg font-black text-amber-900 flex items-center gap-2">
          <span>⚙️</span> 教師與家長選單
        </h3>
        <button id="btn-close-teacher" class="text-slate-400 hover:text-slate-700 text-2xl font-bold p-1">&times;</button>
      </div>

      <div class="mt-4 space-y-4">
        <!-- Direct Level Jumper -->
        <div>
          <label class="block text-xs font-bold text-slate-600 mb-1.5">跳轉關卡 (共 8 關)：</label>
          <div class="grid grid-cols-4 gap-2" id="level-jumper-buttons">
            <!-- 1 to 8 buttons -->
          </div>
        </div>

        <!-- Speech Rate Controller -->
        <div>
          <label class="block text-xs font-bold text-slate-600 mb-1">廣東話語速調整：</label>
          <div class="flex items-center gap-3">
            <input id="speech-rate-slider" type="range" min="0.6" max="1.0" step="0.1" value="0.75" class="w-full accent-amber-500 cursor-pointer">
            <span id="speech-rate-label" class="text-sm font-black text-amber-800 w-12">慢速</span>
          </div>
          <p class="text-[11px] text-slate-400 mt-0.5">預設已調校為慢速發音，有助認知困難學童清晰聆聽字音。</p>
        </div>

        <!-- Fullscreen Button -->
        <div>
          <button id="btn-toggle-fullscreen" class="w-full py-2.5 bg-slate-100 hover:bg-slate-200 text-slate-800 font-bold rounded-xl text-sm flex items-center justify-center gap-2 border border-slate-300">
            <span>📺</span> 切換全螢幕模式 (平板體驗更佳)
          </button>
        </div>
      </div>

      <div class="mt-6 pt-3 border-t border-slate-100 text-right">
        <button id="btn-save-teacher" class="px-5 py-2 bg-amber-500 hover:bg-amber-600 text-white font-bold rounded-xl text-sm">
          確定完成
        </button>
      </div>
    </div>
  </div>

  <script>
    /*
     * TARGET 15 SINGLE-COMPONENT CHARACTERS (獨體字):
     * 日、人、田、口、水、火、石、木、女、山、心、中、大、十、土
     * Detailed with Cantonese phonetic cues, meaning explanation, and child-friendly SVGs.
     */
    const CHARACTERS_DB = [
      {
        char: "日",
        jyutping: "jat6",
        pronunciationText: "日。太陽個日。圓圓嘅太陽喺天上面照發光。",
        meaning: "太陽 / 白天",
        clue: "太陽個日",
        color: "#F59E0B", // amber
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <circle cx="50" cy="50" r="24" fill="#F59E0B" />
          <circle cx="50" cy="50" r="16" fill="#FDE047" />
          <!-- Ray lines -->
          <line x1="50" y1="12" x2="50" y2="4" stroke="#F59E0B" stroke-width="5" stroke-linecap="round"/>
          <line x1="50" y1="88" x2="50" y2="96" stroke="#F59E0B" stroke-width="5" stroke-linecap="round"/>
          <line x1="12" y1="50" x2="4" y2="50" stroke="#F59E0B" stroke-width="5" stroke-linecap="round"/>
          <line x1="88" y1="50" x2="96" y2="50" stroke="#F59E0B" stroke-width="5" stroke-linecap="round"/>
          <line x1="23" y1="23" x2="17" y2="17" stroke="#F59E0B" stroke-width="5" stroke-linecap="round"/>
          <line x1="77" y1="77" x2="83" y2="83" stroke="#F59E0B" stroke-width="5" stroke-linecap="round"/>
          <line x1="23" y1="77" x2="17" y2="83" stroke="#F59E0B" stroke-width="5" stroke-linecap="round"/>
          <line x1="77" y1="23" x2="83" y2="17" stroke="#F59E0B" stroke-width="5" stroke-linecap="round"/>
          <!-- Smile face for kid warmth -->
          <circle cx="44" cy="47" r="2.5" fill="#78350F"/>
          <circle cx="56" cy="47" r="2.5" fill="#78350F"/>
          <path d="M44 54 Q50 58 56 54" stroke="#78350F" stroke-width="2.5" fill="none" stroke-linecap="round"/>
        </svg>`
      },
      {
        char: "人",
        jyutping: "jan4",
        pronunciationText: "人。兩條腿站立行路嘅人。",
        meaning: "人 / 小朋友",
        clue: "人類個人",
        color: "#3B82F6", // blue
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <!-- Head -->
          <circle cx="50" cy="28" r="14" fill="#FDBA74"/>
          <!-- Hair -->
          <path d="M38 24 Q50 14 62 24" fill="#334155"/>
          <!-- Happy face -->
          <circle cx="45" cy="28" r="2" fill="#1E293B"/>
          <circle cx="55" cy="28" r="2" fill="#1E293B"/>
          <path d="M46 34 Q50 38 54 34" stroke="#1E293B" stroke-width="2" fill="none" stroke-linecap="round"/>
          <!-- Body and legs showing 人 stroke shape -->
          <path d="M50 42 Q40 60 25 85" stroke="#2563EB" stroke-width="8" stroke-linecap="round" fill="none"/>
          <path d="M45 56 Q55 70 75 85" stroke="#2563EB" stroke-width="8" stroke-linecap="round" fill="none"/>
        </svg>`
      },
      {
        char: "田",
        jyutping: "tin4",
        pronunciationText: "田。農田個田。一格格種植蔬菜嘅稻田。",
        meaning: "農田 / 田地",
        clue: "農田個田",
        color: "#10B981", // green
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <rect x="20" y="20" width="60" height="60" rx="10" fill="#BBF7D0" stroke="#059669" stroke-width="6"/>
          <line x1="50" y1="20" x2="50" y2="80" stroke="#059669" stroke-width="6"/>
          <line x1="20" y1="50" x2="80" y2="50" stroke="#059669" stroke-width="6"/>
          <!-- Little sprouts inside squares -->
          <circle cx="35" cy="35" r="4" fill="#10B981"/>
          <circle cx="65" cy="35" r="4" fill="#10B981"/>
          <circle cx="35" cy="65" r="4" fill="#10B981"/>
          <circle cx="65" cy="65" r="4" fill="#10B981"/>
        </svg>`
      },
      {
        char: "口",
        jyutping: "hau2",
        pronunciationText: "口。嘴巴個口。可以笑同講嘢嘅嘴仔。",
        meaning: "嘴巴 / 說話",
        clue: "嘴巴個口",
        color: "#F43F5E", // rose
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <!-- Big open smiling mouth -->
          <rect x="22" y="25" width="56" height="50" rx="14" fill="#FFE4E6" stroke="#E11D48" stroke-width="7"/>
          <path d="M30 45 Q50 65 70 45" fill="#E11D48"/>
          <!-- Little teeth & tongue -->
          <rect x="42" y="44" width="16" height="6" rx="2" fill="#FFFFFF"/>
          <circle cx="50" cy="57" r="7" fill="#FDA4AF"/>
        </svg>`
      },
      {
        char: "水",
        jyutping: "seoi2",
        pronunciationText: "水。飲水個水。清涼流動嘅水滴同河流。",
        meaning: "流動水滴 / 河流",
        clue: "喝水個水",
        color: "#06B6D4", // cyan
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <!-- Water drop -->
          <path d="M50 16 C50 16 26 48 26 64 C26 77 37 87 50 87 C63 87 74 77 74 64 C74 48 50 16 50 16 Z" fill="#67E8F9" stroke="#0891B2" stroke-width="5"/>
          <path d="M42 46 C42 46 34 56 34 64 C34 70 38 74 44 75" fill="none" stroke="#FFFFFF" stroke-width="4" stroke-linecap="round"/>
        </svg>`
      },
      {
        char: "火",
        jyutping: "fo2",
        pronunciationText: "火。火焰個火。溫暖明亮嘅火苗。",
        meaning: "燃燒火焰 / 火苗",
        clue: "火焰個火",
        color: "#EF4444", // red
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <!-- Fire shape -->
          <path d="M50 14 C56 28 66 38 72 50 C80 64 72 82 56 86 C70 74 65 58 56 50 C54 62 46 70 38 72 C30 65 28 50 36 38 C42 42 45 36 44 30 C44 22 50 14 50 14 Z" fill="#F87171" stroke="#DC2626" stroke-width="4"/>
          <!-- Inner yellow flame -->
          <path d="M50 42 C53 50 60 58 60 66 C60 76 52 82 48 83 C55 76 53 66 48 62 C46 68 42 72 38 70 C36 62 42 54 48 48 Z" fill="#FDE047"/>
        </svg>`
      },
      {
        char: "石",
        jyutping: "sek6",
        pronunciationText: "石。石頭個石。硬朗圓碌碌嘅小石頭。",
        meaning: "堅硬石頭",
        clue: "石頭個石",
        color: "#64748B", // slate
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <!-- Big rock shape -->
          <polygon points="30,30 68,22 84,52 68,82 24,78 16,50" fill="#CBD5E1" stroke="#475569" stroke-width="6" stroke-linejoin="round"/>
          <line x1="38" y1="36" x2="52" y2="60" stroke="#94A3B8" stroke-width="4" stroke-linecap="round"/>
          <line x1="52" y1="60" x2="72" y2="52" stroke="#94A3B8" stroke-width="4" stroke-linecap="round"/>
          <circle cx="34" cy="62" r="3" fill="#64748B"/>
          <circle cx="62" cy="38" r="2.5" fill="#64748B"/>
        </svg>`
      },
      {
        char: "木",
        jyutping: "muk6",
        pronunciationText: "木。樹木個木。有樹幹同綠色樹葉嘅大樹。",
        meaning: "樹木 / 木頭",
        clue: "樹木個木",
        color: "#16A34A", // green
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <!-- Tree trunk -->
          <rect x="44" y="45" width="12" height="42" rx="3" fill="#92400E"/>
          <!-- Tree leaves canopy mimicking tree structure -->
          <circle cx="50" cy="35" r="24" fill="#4ADE80" stroke="#16A34A" stroke-width="5"/>
          <circle cx="36" cy="42" r="14" fill="#22C55E"/>
          <circle cx="64" cy="42" r="14" fill="#22C55E"/>
          <!-- Root branch branches -->
          <path d="M44 80 L32 87" stroke="#92400E" stroke-width="4" stroke-linecap="round"/>
          <path d="M56 80 L68 87" stroke="#92400E" stroke-width="4" stroke-linecap="round"/>
        </svg>`
      },
      {
        char: "女",
        jyutping: "neoi5",
        pronunciationText: "女。女孩個女。留住長頭髮嘅可愛女仔。",
        meaning: "女孩 / 媽媽",
        clue: "女孩個女",
        color: "#EC4899", // pink
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <!-- Girl face and hair with bow -->
          <circle cx="50" cy="34" r="15" fill="#FED7AA"/>
          <!-- Hair & twin tails -->
          <path d="M34 30 C30 15 70 15 66 30" fill="#374151"/>
          <ellipse cx="26" cy="38" rx="7" ry="11" fill="#374151"/>
          <ellipse cx="74" cy="38" rx="7" ry="11" fill="#374151"/>
          <!-- Pink Bow ribbon -->
          <polygon points="46,18 42,12 46,15 50,15 54,12 50,18" fill="#F43F5E"/>
          <circle cx="48" cy="15" r="2.5" fill="#FB7185"/>
          <!-- Happy smiling expression -->
          <circle cx="45" cy="33" r="2" fill="#1F2937"/>
          <circle cx="55" cy="33" r="2" fill="#1F2937"/>
          <path d="M46 39 Q50 43 54 39" stroke="#E11D48" stroke-width="2" fill="none" stroke-linecap="round"/>
          <!-- Dress -->
          <path d="M40 52 L30 84 L70 84 L60 52 Z" fill="#F472B6" stroke="#DB2777" stroke-width="4" stroke-linejoin="round"/>
        </svg>`
      },
      {
        char: "山",
        jyutping: "saan1",
        pronunciationText: "山。高山個山。連綿起伏嘅三個山峰。",
        meaning: "群山 / 高山",
        clue: "高山個山",
        color: "#059669", // emerald
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <!-- 3 mountain peaks reflecting 山 shape -->
          <polygon points="12,82 32,45 50,82" fill="#6EE7B7" stroke="#047857" stroke-width="4" stroke-linejoin="round"/>
          <polygon points="50,82 68,48 88,82" fill="#6EE7B7" stroke="#047857" stroke-width="4" stroke-linejoin="round"/>
          <polygon points="28,82 50,22 72,82" fill="#34D399" stroke="#047857" stroke-width="5" stroke-linejoin="round"/>
          <!-- Snow caps on tallest center mountain -->
          <polygon points="45,35 50,23 55,35 51,32" fill="#FFFFFF"/>
        </svg>`
      },
      {
        char: "心",
        jyutping: "sam1",
        pronunciationText: "心。愛心個心。有愛心、紅卜卜嘅心形。",
        meaning: "愛心 / 內心",
        clue: "愛心個心",
        color: "#E11D48", // rose
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <path d="M50 78 C25 60 16 42 22 28 C28 16 42 18 50 29 C58 18 72 16 78 28 C84 42 75 60 50 78 Z" fill="#FB7185" stroke="#E11D48" stroke-width="6"/>
          <!-- Small shine highlight -->
          <ellipse cx="36" cy="32" rx="4" ry="7" transform="rotate(-30 36 32)" fill="#FFFFFF" opacity="0.7"/>
        </svg>`
      },
      {
        char: "中",
        jyutping: "zung1",
        pronunciationText: "中。中心個中。一枝箭射穿正中間嘅靶心。",
        meaning: "中間 / 靶心",
        clue: "中間個中",
        color: "#8B5CF6", // purple
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <!-- Target bullseye -->
          <circle cx="50" cy="50" r="32" fill="#DDD6FE" stroke="#7C3AED" stroke-width="5"/>
          <circle cx="50" cy="50" r="18" fill="#A78BFA" stroke="#6D28D9" stroke-width="4"/>
          <circle cx="50" cy="50" r="7" fill="#6D28D9"/>
          <!-- Arrow/line through middle representing 中 -->
          <line x1="50" y1="10" x2="50" y2="90" stroke="#4C1D95" stroke-width="7" stroke-linecap="round"/>
        </svg>`
      },
      {
        char: "大",
        jyutping: "daai6",
        pronunciationText: "大。大小個大。張開手腳、好似大大個人咁。",
        meaning: "巨大 / 大張手腳",
        clue: "大小個大",
        color: "#F97316", // orange
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <!-- Head -->
          <circle cx="50" cy="22" r="11" fill="#FDBA74"/>
          <!-- Wide spread arms and legs -->
          <line x1="16" y1="42" x2="84" y2="42" stroke="#EA580C" stroke-width="8" stroke-linecap="round"/>
          <line x1="50" y1="33" x2="50" y2="60" stroke="#EA580C" stroke-width="8" stroke-linecap="round"/>
          <line x1="50" y1="60" x2="26" y2="86" stroke="#EA580C" stroke-width="8" stroke-linecap="round"/>
          <line x1="50" y1="60" x2="74" y2="86" stroke="#EA580C" stroke-width="8" stroke-linecap="round"/>
          <!-- Sparkles -->
          <text x="14" y="30" font-size="14" fill="#F59E0B">✨</text>
          <text x="76" y="30" font-size="14" fill="#F59E0B">✨</text>
        </svg>`
      },
      {
        char: "十",
        jyutping: "sap6",
        pronunciationText: "十。數字十。橫加一豎、十全十美嘅十字。",
        meaning: "數字 10 / 十字",
        clue: "數字十個十",
        color: "#0284C7", // light blue
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <!-- Plus / Cross shape with 10 count badge -->
          <rect x="42" y="16" width="16" height="68" rx="8" fill="#38BDF8" stroke="#0284C7" stroke-width="4"/>
          <rect x="16" y="42" width="68" height="16" rx="8" fill="#38BDF8" stroke="#0284C7" stroke-width="4"/>
          <!-- Inner badge with "10" -->
          <circle cx="50" cy="50" r="14" fill="#0284C7"/>
          <text x="50" y="55" font-size="14" font-weight="900" fill="#FFFFFF" text-anchor="middle" font-family="sans-serif">10</text>
        </svg>`
      },
      {
        char: "土",
        jyutping: "tou2",
        pronunciationText: "土。泥土個土。肥沃嘅泥土上面生出小嫩芽。",
        meaning: "泥土 / 大地",
        clue: "泥土個土",
        color: "#A16207", // brown/amber
        svg: `<svg viewBox="0 0 100 100" class="w-full h-full">
          <!-- Soil ground mound -->
          <path d="M16 80 Q50 56 84 80 Z" fill="#D97706" stroke="#92400E" stroke-width="5"/>
          <rect x="12" y="80" width="76" height="8" rx="4" fill="#B45309"/>
          <!-- Growing green sprout -->
          <line x1="50" y1="62" x2="50" y2="38" stroke="#16A34A" stroke-width="5" stroke-linecap="round"/>
          <path d="M50 44 Q40 32 34 38 Q40 50 50 46" fill="#4ADE80" stroke="#16A34A" stroke-width="2"/>
          <path d="M50 40 Q60 28 66 34 Q60 46 50 42" fill="#4ADE80" stroke="#16A34A" stroke-width="2"/>
        </svg>`
      }
    ];

    /* STAMP TITLES FOR THE 8 LEVELS */
    const LEVEL_TITLES = [
      "自然小探索家",
      "認字聰明小神童",
      "漢字火眼金睛",
      "象形字好朋友",
      "字形記憶小達人",
      "獨體字小隊長",
      "文字積木大師",
      "獨體字大博士 🏆"
    ];

    const TOTAL_LEVELS = 8;
    const WORDS_PER_LEVEL = 5;

    let currentLevel = 1; // 1 to 8
    let currentWords = []; // 5 selected character objects
    let selectedCharId = null; // Currently clicked character card
    let matchedPairs = new Set(); // Set of matched char keys
    let speechRate = 0.75; // Slower default for SEN K3 clarity
    let audioCtx = null;

    // Encouragement Phrases in Cantonese
    const PRAISE_PHRASES = [
      "好叻呀！答啱喇！",
      "真係太棒喇！好聰明！",
      "答得好！好有心思！",
      "啱哂！好犀利呀！",
      "抵讚！完全正確！"
    ];

    const RETRY_PHRASES = [
      "唔緊要，再睇清楚啲，再試一次啦！",
      "差少少咋，摸一摸卡片再試多次啦！",
      "慢慢嚟，聽多次發音先啦！"
    ];

    // Initialize Web Audio Context on first user tap
    function getAudioContext() {
      if (!audioCtx) {
        const AudioContext = window.AudioContext || window.webkitAudioContext;
        if (AudioContext) {
          audioCtx = new AudioContext();
        }
      }
      if (audioCtx && audioCtx.state === 'suspended') {
        audioCtx.resume();
      }
      return audioCtx;
    }

    // Gentle Cheerful Chime for success (Web Audio Synthesizer)
    function playChime(success = true) {
      try {
        const ctx = getAudioContext();
        if (!ctx) return;
        const now = ctx.currentTime;
        const osc = ctx.createOscillator();
        const gain = ctx.createGain();

        osc.connect(gain);
        gain.connect(ctx.destination);

        if (success) {
          // Cheerful ascending arpeggio (C5 -> E5 -> G5 -> C6)
          osc.type = 'triangle';
          osc.frequency.setValueAtTime(523.25, now); // C5
          osc.frequency.exponentialRampToValueAtTime(659.25, now + 0.1); // E5
          osc.frequency.exponentialRampToValueAtTime(783.99, now + 0.2); // G5
          osc.frequency.exponentialRampToValueAtTime(1046.50, now + 0.3); // C6

          gain.gain.setValueAtTime(0.2, now);
          gain.gain.exponentialRampToValueAtTime(0.001, now + 0.6);

          osc.start(now);
          osc.stop(now + 0.6);
        } else {
          // Soft wobble (non-punishing warm tone)
          osc.type = 'sine';
          osc.frequency.setValueAtTime(320, now);
          osc.frequency.exponentialRampToValueAtTime(280, now + 0.15);

          gain.gain.setValueAtTime(0.12, now);
          gain.gain.exponentialRampToValueAtTime(0.001, now + 0.35);

          osc.start(now);
          osc.stop(now + 0.35);
        }
      } catch (e) {
        console.warn("Audio Context playback error:", e);
      }
    }

    function speakCantonese(text, onEndCallback = null) {
      if (!('speechSynthesis' in window)) {
        if (onEndCallback) onEndCallback();
        return;
      }

      window.speechSynthesis.cancel(); // Stop any ongoing speech

      const utterance = new SpeechSynthesisUtterance(text);
      utterance.rate = speechRate;
      utterance.pitch = 1.1; // Slightly higher pitch for friendly child warmth

      // Find Cantonese voice (zh-HK), fallback to zh-TW or zh
      const voices = window.speechSynthesis.getVoices();
      const hkVoice = voices.find(v => v.lang === 'zh-HK' || v.lang === 'zh_HK' || v.lang.includes('HK'));
      const twVoice = voices.find(v => v.lang === 'zh-TW' || v.lang === 'zh_TW');

      if (hkVoice) {
        utterance.voice = hkVoice;
        utterance.lang = 'zh-HK';
      } else if (twVoice) {
        utterance.voice = twVoice;
        utterance.lang = 'zh-TW';
      } else {
        utterance.lang = 'zh-HK';
      }

      if (onEndCallback) {
        utterance.onend = onEndCallback;
        utterance.onerror = onEndCallback;
      }

      window.speechSynthesis.speak(utterance);
    }

    // Ensure voices are loaded in browsers that fetch them asynchronously
    if ('speechSynthesis' in window) {
      window.speechSynthesis.onvoiceschanged = () => {
        // Pre-load
      };
    }

    // Helper to shuffle an array
    function shuffleArray(array) {
      const arr = [...array];
      for (let i = arr.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [arr[i], arr[j]] = [arr[j], arr[i]];
      }
      return arr;
    }

    // Pick 5 words for level. To ensure rich exposure across 8 levels:
    // Seeds vary per level so each level gives fresh mix
    function pickWordsForLevel(lvl) {
      // Deterministic yet pleasantly mixed rotation based on level number
      const shuffledAll = shuffleArray(CHARACTERS_DB);
      return shuffledAll.slice(0, WORDS_PER_LEVEL);
    }

    function updateProgressIndicators() {
      const levelDisplay = document.getElementById('level-display');
      levelDisplay.innerText = `第 ${currentLevel} / ${TOTAL_LEVELS} 關`;

      const starsContainer = document.getElementById('stars-container');
      starsContainer.innerHTML = '';
      for (let i = 1; i <= TOTAL_LEVELS; i++) {
        const star = document.createElement('div');
        star.className = `w-6 h-6 sm:w-7 sm:h-7 rounded-full flex items-center justify-center text-xs font-bold transition-transform ${
          i < currentLevel
            ? 'bg-amber-400 text-amber-950 shadow-sm scale-95'
            : i === currentLevel
            ? 'bg-orange-500 text-white shadow-md ring-2 ring-orange-300 scale-110 font-black'
            : 'bg-slate-200 text-slate-400'
        }`;
        star.innerHTML = i < currentLevel ? '⭐' : i;
        starsContainer.appendChild(star);
      }
    }

    // Render Preview View (認一認・聽一聽)
    function renderPreviewView() {
      const container = document.getElementById('preview-cards-container');
      container.innerHTML = '';

      currentWords.forEach((item, index) => {
        const card = document.createElement('div');
        card.className = "bg-white p-3 sm:p-4 rounded-3xl border-3 border-amber-200 shadow-md flex flex-col items-center justify-between text-center relative group hover:border-amber-400 transition transform hover:-translate-y-1";
        card.style.borderColor = `${item.color}40`;

        card.innerHTML = `
          <!-- Character Header -->
          <div class="text-xs font-black px-2 py-0.5 rounded-full text-white mb-1.5" style="background-color: ${item.color}">
            第 ${index + 1} 個字
          </div>

          <!-- Large Character with Clear Strokes -->
          <div class="char-font text-5xl sm:text-6xl my-1 font-black" style="color: ${item.color};">
            ${item.char}
          </div>

          <!-- Pictogram illustration -->
          <div class="w-16 h-16 sm:w-20 sm:h-20 my-2 p-1.5 bg-amber-50 rounded-2xl flex items-center justify-center border border-amber-100">
            ${item.svg}
          </div>

          <!-- Meaning label -->
          <div class="text-sm sm:text-base font-extrabold text-slate-700">
            ${item.meaning}
          </div>
          <div class="text-xs font-bold text-slate-400 mb-2">
            (${item.clue})
          </div>

          <!-- Audio Read Button -->
          <button class="w-full py-2 bg-amber-100 hover:bg-amber-200 text-amber-900 rounded-xl font-black text-xs sm:text-sm flex items-center justify-center gap-1 active:scale-95 transition" data-char="${item.char}">
            <span>🔊 點我讀音</span>
          </button>
        `;

        // Click audio triggers
        const triggerAudio = () => {
          getAudioContext();
          speakCantonese(item.pronunciationText);
          card.classList.add('pulse-selected');
          setTimeout(() => card.classList.remove('pulse-selected'), 1200);
        };

        card.addEventListener('click', triggerAudio);
        container.appendChild(card);
      });
    }

    function renderMatchingView() {
      selectedCharId = null;
      matchedPairs.clear();
      document.getElementById('matched-count').innerText = "0";

      const charCol = document.getElementById('char-column');
      const picCol = document.getElementById('pic-column');
      charCol.innerHTML = '';
      picCol.innerHTML = '';

      // Shuffled characters on left, shuffled pictograms on right
      const charsList = shuffleArray(currentWords);
      const picsList = shuffleArray(currentWords);

      // Render Left Cards: Chinese Characters
      charsList.forEach(item => {
        const btn = document.createElement('button');
        btn.id = `char-btn-${item.char}`;
        btn.dataset.char = item.char;
        btn.className = "w-full min-h-[72px] sm:min-h-[84px] bg-white hover:bg-amber-50 active:scale-98 border-3 border-amber-200 rounded-2xl p-2 sm:p-3 flex items-center justify-between shadow-sm transition-all";
        btn.innerHTML = `
          <div class="flex items-center gap-3">
            <span class="char-font text-4xl sm:text-5xl font-black px-2 py-0.5 rounded-xl bg-amber-50 border border-amber-200" style="color: ${item.color}">
              ${item.char}
            </span>
            <div class="text-left">
              <span class="text-xs sm:text-sm font-bold text-slate-500 block">獨體字</span>
              <span class="text-xs font-bold text-amber-700 bg-amber-100 px-1.5 py-0.5 rounded-md">${item.clue}</span>
            </div>
          </div>
          <div class="speaker-pill p-2 rounded-xl bg-orange-100 hover:bg-orange-200 text-orange-800 transition">
            <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 20 20">
              <path fill-rule="evenodd" d="M9.383 3.076A1 1 0 0110 4v12a1 1 0 01-1.617.774l-4.14-3.313H2a1 1 0 01-1-1V7.539a1 1 0 011-1h2.243l4.14-3.313a1 1 0 011-.15zm5.088 2.095a1 1 0 011.414 0 7.002 7.002 0 010 9.898 1 1 0 11-1.414-1.414 5.002 5.002 0 000-7.07 1 1 0 010-1.414z" clip-rule="evenodd"/>
            </svg>
          </div>
        `;

        btn.addEventListener('click', (e) => {
          handleCharClick(item.char);
        });

        charCol.appendChild(btn);
      });

      // Render Right Cards: Pictograms / Meanings
      picsList.forEach(item => {
        const btn = document.createElement('button');
        btn.id = `pic-btn-${item.char}`;
        btn.dataset.char = item.char;
        btn.className = "w-full min-h-[72px] sm:min-h-[84px] bg-white hover:bg-amber-50 active:scale-98 border-3 border-amber-200 rounded-2xl p-2 sm:p-3 flex items-center justify-between shadow-sm transition-all";
        btn.innerHTML = `
          <div class="flex items-center gap-3">
            <div class="w-14 h-14 sm:w-16 sm:h-16 p-1 bg-amber-50 rounded-xl border border-amber-100 flex items-center justify-center shrink-0">
              ${item.svg}
            </div>
            <div class="text-left">
              <span class="text-base sm:text-lg font-black text-slate-800 block">${item.meaning}</span>
              <span class="text-xs font-bold text-slate-400">象形圖案</span>
            </div>
          </div>
          <div class="check-pill w-8 h-8 rounded-full border-2 border-slate-200 flex items-center justify-center text-slate-300">
            ?
          </div>
        `;

        btn.addEventListener('click', () => {
          handlePicClick(item.char);
        });

        picCol.appendChild(btn);
      });
    }

    function handleCharClick(char) {
      getAudioContext();
      const charObj = currentWords.find(w => w.char === char);
      if (!charObj) return;

      // Speak pronunciation
      speakCantonese(`${charObj.char}。${charObj.clue}`);

      // If already matched, return
      if (matchedPairs.has(char)) return;

      selectedCharId = char;

      // Update UI highlight on left cards
      currentWords.forEach(w => {
        const btn = document.getElementById(`char-btn-${w.char}`);
        if (!btn || matchedPairs.has(w.char)) return;

        if (w.char === char) {
          btn.classList.add('ring-4', 'ring-orange-400', 'border-orange-500', 'bg-orange-50');
          btn.classList.remove('border-amber-200', 'bg-white');
        } else {
          btn.classList.remove('ring-4', 'ring-orange-400', 'border-orange-500', 'bg-orange-50');
          btn.classList.add('border-amber-200', 'bg-white');
        }
      });

      // Update status prompt banner
      const banner = document.getElementById('match-hint-banner');
      banner.innerHTML = `<span>👉 你揀選咗【${charObj.char}】！快啲喺右邊搵出屬於佢嘅圖案啦！</span>`;
    }

    function handlePicClick(char) {
      getAudioContext();
      const picObj = currentWords.find(w => w.char === char);
      if (!picObj) return;

      // If already matched, just speak description
      if (matchedPairs.has(char)) {
        speakCantonese(`${picObj.char}。${picObj.meaning}`);
        return;
      }

      // If user hasn't selected a character first, gently remind
      if (!selectedCharId) {
        speakCantonese("請先點選左邊嘅中文字，再點選圖案啦！");
        const picBtn = document.getElementById(`pic-btn-${char}`);
        if (picBtn) {
          picBtn.classList.add('wiggle-hint');
          setTimeout(() => picBtn.classList.remove('wiggle-hint'), 600);
        }
        return;
      }

      const leftCharBtn = document.getElementById(`char-btn-${selectedCharId}`);
      const rightPicBtn = document.getElementById(`pic-btn-${char}`);

      // CHECK IF MATCH IS CORRECT
      if (selectedCharId === char) {
        // MATCH SUCCESS!
        matchedPairs.add(char);
        playChime(true);

        // Sound celebration in Cantonese
        const randomPraise = PRAISE_PHRASES[Math.floor(Math.random() * PRAISE_PHRASES.length)];
        speakCantonese(`${randomPraise} ${picObj.char}，${picObj.clue}！`);

        // Style left button as completed
        if (leftCharBtn) {
          leftCharBtn.className = "w-full min-h-[72px] sm:min-h-[84px] bg-emerald-50 border-3 border-emerald-500 rounded-2xl p-2 sm:p-3 flex items-center justify-between shadow matched-bounce";
          leftCharBtn.innerHTML += `<span class="text-xl">✅</span>`;
        }

        // Style right button as completed
        if (rightPicBtn) {
          rightPicBtn.className = "w-full min-h-[72px] sm:min-h-[84px] bg-emerald-50 border-3 border-emerald-500 rounded-2xl p-2 sm:p-3 flex items-center justify-between shadow matched-bounce";
          const checkPill = rightPicBtn.querySelector('.check-pill');
          if (checkPill) {
            checkPill.className = "w-8 h-8 rounded-full bg-emerald-500 text-white font-bold flex items-center justify-center";
            checkPill.innerHTML = "✓";
          }
        }

        // Update count
        document.getElementById('matched-count').innerText = matchedPairs.size;

        // Reset current selection
        selectedCharId = null;

        // Reset banner
        const banner = document.getElementById('match-hint-banner');
        banner.innerHTML = `<span>🌟 答啱喇！繼續搵下一個獨體字啦！</span>`;

        // Check if level completed
        if (matchedPairs.size === WORDS_PER_LEVEL) {
          setTimeout(() => {
            handleLevelComplete();
          }, 800);
        }

      } else {
        // MATCH INCORRECT (Gentle, warm reassurance)
        playChime(false);
        const randomRetry = RETRY_PHRASES[Math.floor(Math.random() * RETRY_PHRASES.length)];
        speakCantonese(randomRetry);

        if (rightPicBtn) {
          rightPicBtn.classList.add('wiggle-hint', 'border-rose-300');
          setTimeout(() => {
            rightPicBtn.classList.remove('wiggle-hint', 'border-rose-300');
          }, 600);
        }
      }
    }

    function handleLevelComplete() {
      // Confetti firework
      if (typeof confetti === 'function') {
        confetti({
          particleCount: 80,
          spread: 70,
          origin: { y: 0.6 }
        });
      }

      // Check if all 8 levels are finished
      if (currentLevel >= TOTAL_LEVELS) {
        speakCantonese("好叻呀！你已經成功完成哂 8 關喇！你係獨體字大博士！");
        document.getElementById('modal-game-complete').classList.remove('hidden');
      } else {
        const titleBadge = LEVEL_TITLES[currentLevel - 1] || "獨體字小學士";
        document.getElementById('stamp-badge-title').innerText = titleBadge;
        document.getElementById('modal-success-msg').innerText = `你好有耐性，順利完成第 ${currentLevel} 關嘅 5 個獨體字配對！`;
        speakCantonese(`太好喇！你已經完成第 ${currentLevel} 關！獲得 ${titleBadge} 印章！`);
        document.getElementById('modal-level-success').classList.remove('hidden');
      }
    }

    // Initialize or load a specific level
    function startLevel(lvl) {
      currentLevel = lvl;
      currentWords = pickWordsForLevel(lvl);
      updateProgressIndicators();

      // Default to View A (Preview & Listen) so SEN children can see & hear before matching
      switchView('preview');
      renderPreviewView();
      renderMatchingView();

      // Cantonese voice introduction to level
      speakCantonese(`歡迎嚟到第 ${currentLevel} 關！我哋一齊先聽一聽、認一認呢 5 個獨體字啦！`);
    }

    // Switching views
    function switchView(mode) {
      const previewSec = document.getElementById('view-preview');
      const matchSec = document.getElementById('view-match');
      const tabPreview = document.getElementById('tab-step-preview');
      const tabMatch = document.getElementById('tab-step-match');

      if (mode === 'preview') {
        previewSec.classList.remove('hidden');
        matchSec.classList.add('hidden');
        tabPreview.className = "px-4 py-2 rounded-xl font-black text-sm sm:text-base transition bg-orange-500 text-white shadow-md flex items-center gap-1.5";
        tabMatch.className = "px-4 py-2 rounded-xl font-black text-sm sm:text-base transition bg-white/70 text-orange-900 hover:bg-white flex items-center gap-1.5";
      } else {
        previewSec.classList.add('hidden');
        matchSec.classList.remove('hidden');
        tabMatch.className = "px-4 py-2 rounded-xl font-black text-sm sm:text-base transition bg-orange-500 text-white shadow-md flex items-center gap-1.5";
        tabPreview.className = "px-4 py-2 rounded-xl font-black text-sm sm:text-base transition bg-white/70 text-orange-900 hover:bg-white flex items-center gap-1.5";
      }
    }

    document.addEventListener('DOMContentLoaded', () => {
      // Tab navigation
      document.getElementById('tab-step-preview').addEventListener('click', () => switchView('preview'));
      document.getElementById('tab-step-match').addEventListener('click', () => switchView('match'));
      document.getElementById('btn-start-matching').addEventListener('click', () => {
        switchView('match');
        speakCantonese("開始配對挑戰！請點選左邊嘅中文字，再搵出右邊嘅圖案啦！");
      });
      document.getElementById('btn-back-to-preview').addEventListener('click', () => switchView('preview'));

      // Replay audio guidance button
      document.getElementById('btn-sound-replay').addEventListener('click', () => {
        getAudioContext();
        speakCantonese(`第 ${currentLevel} 關。請小朋友點選左邊嘅中文字，再點選右邊相應嘅意思圖案！`);
      });

      // Reset level button
      document.getElementById('btn-reset-current-level').addEventListener('click', () => {
        renderMatchingView();
        speakCantonese("重新開始呢一關配對！");
      });

      // Modal level next button
      document.getElementById('btn-next-level').addEventListener('click', () => {
        document.getElementById('modal-level-success').classList.add('hidden');
        startLevel(currentLevel + 1);
      });

      // Modal replay level
      document.getElementById('btn-replay-modal').addEventListener('click', () => {
        document.getElementById('modal-level-success').classList.add('hidden');
        startLevel(currentLevel);
      });

      // Restart entire game
      document.getElementById('btn-restart-game').addEventListener('click', () => {
        document.getElementById('modal-game-complete').classList.add('hidden');
        startLevel(1);
      });

      // Teacher / Parent modal triggers
      const teacherModal = document.getElementById('modal-teacher');
      document.getElementById('btn-teacher-menu').addEventListener('click', () => {
        teacherModal.classList.remove('hidden');
      });
      document.getElementById('btn-close-teacher').addEventListener('click', () => {
        teacherModal.classList.add('hidden');
      });
      document.getElementById('btn-save-teacher').addEventListener('click', () => {
        teacherModal.classList.add('hidden');
      });

      // Teacher level jump buttons
      const jumperContainer = document.getElementById('level-jumper-buttons');
      for (let i = 1; i <= TOTAL_LEVELS; i++) {
        const btn = document.createElement('button');
        btn.className = "py-2 bg-amber-100 hover:bg-amber-200 text-amber-900 font-black rounded-xl text-xs sm:text-sm border border-amber-300";
        btn.innerText = `關卡 ${i}`;
        btn.addEventListener('click', () => {
          teacherModal.classList.add('hidden');
          startLevel(i);
        });
        jumperContainer.appendChild(btn);
      }

      // Speech speed slider
      const speedSlider = document.getElementById('speech-rate-slider');
      const speedLabel = document.getElementById('speech-rate-label');
      speedSlider.addEventListener('input', (e) => {
        speechRate = parseFloat(e.target.value);
        if (speechRate <= 0.7) {
          speedLabel.innerText = "特慢";
        } else if (speechRate <= 0.85) {
          speedLabel.innerText = "慢速";
        } else {
          speedLabel.innerText = "正常";
        }
      });

      // Fullscreen Toggle
      document.getElementById('btn-toggle-fullscreen').addEventListener('click', () => {
        if (!document.fullscreenElement) {
          document.documentElement.requestFullscreen().catch(err => {
            console.warn("Fullscreen request error:", err);
          });
        } else {
          if (document.exitFullscreen) {
            document.exitFullscreen();
          }
        }
      });

      // Kick off with Level 1
      startLevel(1);
    });
  </script>
</body>
</html>