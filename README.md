<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>우리 반 톡톡 (Talk Talk)</title>
    <!-- Tailwind CSS for modern UI -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Awesome for Icons -->
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Nanum+Round:wght@400;700&display=swap');
        body { font-family: 'Nanum Round', sans-serif; background-color: #f8fafc; }
        
        /* 메시지 애니메이션 */
        .message-appear { animation: fadeIn 0.3s ease-out forwards; }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        /* 커스텀 스크롤바 */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
    </style>
</head>
<body class="h-screen flex flex-col overflow-hidden">

    <!-- Header -->
    <header class="bg-indigo-700 text-white p-4 shadow-lg flex justify-between items-center z-30 shrink-0">
        <div class="flex items-center gap-3">
            <div class="bg-white p-2 rounded-xl text-indigo-700 shadow-inner">
                <i class="fa-solid fa-comments text-xl"></i>
            </div>
            <div>
                <h1 class="text-lg font-bold leading-tight text-white">우리 반 톡톡</h1>
                <p class="text-[10px] opacity-70 uppercase tracking-widest text-indigo-100">Global Classroom Talk</p>
            </div>
        </div>
        <div class="hidden sm:flex bg-indigo-800 rounded-full px-4 py-1.5 text-[11px] font-bold gap-3 text-indigo-100">
            <span>🇰🇷 KO</span>
            <span class="opacity-30">|</span>
            <span>🇷🇺 RU</span>
            <span class="opacity-30">|</span>
            <span>🇨🇳 ZH</span>
        </div>
    </header>

    <!-- Settings Bar -->
    <div class="bg-white border-b px-4 py-3 flex items-center justify-between shadow-sm z-20 shrink-0 overflow-x-auto">
        <div class="flex items-center gap-3 shrink-0">
            <span class="text-slate-400 text-xs font-bold">내 역할:</span>
            <div class="flex bg-slate-100 p-1 rounded-full">
                <button id="role-teacher" onclick="setRole('teacher')" class="px-4 py-1.5 rounded-full text-xs font-bold transition-all bg-indigo-600 text-white shadow-sm">
                    선생님
                </button>
                <button id="role-student" onclick="setRole('student')" class="px-4 py-1.5 rounded-full text-xs font-bold transition-all text-slate-500">
                    학생
                </button>
            </div>
        </div>
        <div id="student-lang-area" class="flex items-center gap-2 border-l pl-4 opacity-30 pointer-events-none transition-opacity">
            <span class="text-slate-400 text-xs font-bold">학생 언어:</span>
            <select id="student-base-lang" class="bg-slate-100 border-none rounded-lg px-2 py-1 text-xs font-bold outline-none text-slate-700">
                <option value="ru">🇷🇺 러시아어</option>
                <option value="zh">🇨🇳 중국어</option>
            </select>
        </div>
    </div>

    <!-- Chat Area: flex-1을 사용하여 남은 모든 공간을 차지하게 함 -->
    <main id="chat-display" class="flex-1 overflow-y-auto p-4 space-y-8 relative bg-slate-50">
        <div class="absolute inset-0 opacity-[0.03] pointer-events-none bg-[url('https://www.transparenttextures.com/patterns/cubes.png')]"></div>
        
        <!-- Welcome Message -->
        <div class="flex flex-col items-start message-appear relative z-10">
            <div class="max-w-[90%] rounded-3xl p-5 shadow-sm bg-white border border-slate-200 rounded-tl-none">
                <div class="flex items-center gap-2 mb-2">
                    <span class="text-[10px] font-black text-indigo-600 bg-indigo-50 px-1.5 py-0.5 rounded">KOR</span>
                    <span class="text-[10px] text-slate-400 font-bold">시스템</span>
                </div>
                <p class="font-bold text-lg text-slate-900">안녕하세요! 아래 입력창에 메시지를 쓰면 러시아어와 중국어로 동시에 번역됩니다.</p>
            </div>
        </div>
    </main>

    <!-- Loading Indicator -->
    <div id="loading" class="hidden px-6 py-2 bg-white text-indigo-600 text-xs font-bold italic border-t border-indigo-50">
        <i class="fa-solid fa-spinner fa-spin mr-2"></i> 번역 엔진 가동 중...
    </div>

    <!-- Error Banner -->
    <div id="error-banner" class="hidden m-4 p-4 bg-red-50 border-2 border-red-100 rounded-2xl flex items-center justify-between text-red-700 shadow-lg">
        <div class="flex items-center gap-3 text-sm">
            <i class="fa-solid fa-circle-exclamation text-lg"></i>
            <span id="error-text">번역 중 오류가 발생했습니다.</span>
        </div>
        <button onclick="hideError()" class="text-red-400 hover:text-red-600"><i class="fa-solid fa-xmark text-lg"></i></button>
    </div>

    <!-- Input Area: shrink-0으로 압축되지 않도록 설정 -->
    <footer class="bg-white p-4 border-t border-slate-200 z-30 shadow-[0_-4px_20px_-5px_rgba(0,0,0,0.1)] shrink-0">
        <div class="max-w-4xl mx-auto space-y-4">
            <!-- Lang Selector -->
            <div class="flex bg-slate-100 p-1 rounded-2xl">
                <button id="lang-ko" onclick="setInputLang('ko')" class="flex-1 py-2 rounded-xl text-xs font-bold transition-all bg-white shadow-sm text-indigo-600">🇰🇷 한국어</button>
                <button id="lang-ru" onclick="setInputLang('ru')" class="flex-1 py-2 rounded-xl text-xs font-bold transition-all text-slate-400">🇷🇺 러시아어</button>
                <button id="lang-zh" onclick="setInputLang('zh')" class="flex-1 py-2 rounded-xl text-xs font-bold transition-all text-slate-400">🇨🇳 중국어</button>
            </div>

            <!-- Input Box -->
            <div class="flex gap-2">
                <input id="text-input" type="text" placeholder="메시지를 입력하세요..." 
                       class="flex-1 border-2 border-slate-100 rounded-2xl px-5 py-4 focus:outline-none focus:border-indigo-500 focus:ring-4 focus:ring-indigo-500/10 transition-all text-base font-medium shadow-sm">
                <button onclick="handleSend()" id="send-btn" 
                        class="bg-indigo-700 text-white px-8 rounded-2xl shadow-lg shadow-indigo-200 transition-all active:scale-90 flex items-center justify-center hover:bg-indigo-800">
                    <i class="fa-solid fa-paper-plane text-xl text-white"></i>
                </button>
            </div>
        </div>
    </footer>

    <script>
        const API_KEY = "AIzaSyAamHgTQJ6GUB7-HlUp7jQgpNPr8zib8Dk";
        let currentRole = 'teacher';
        let currentInputLang = 'ko';

        function setRole(role) {
            currentRole = role;
            document.getElementById('role-teacher').className = role === 'teacher' ? 'px-4 py-1.5 rounded-full text-xs font-bold transition-all bg-indigo-600 text-white shadow-sm' : 'px-4 py-1.5 rounded-full text-xs font-bold transition-all text-slate-500';
            document.getElementById('role-student').className = role === 'student' ? 'px-4 py-1.5 rounded-full text-xs font-bold transition-all bg-amber-500 text-white shadow-sm' : 'px-4 py-1.5 rounded-full text-xs font-bold transition-all text-slate-500';
            
            const studentArea = document.getElementById('student-lang-area');
            if(role === 'student') {
                studentArea.classList.remove('opacity-30', 'pointer-events-none');
            } else {
                studentArea.classList.add('opacity-30', 'pointer-events-none');
            }
        }

        function setInputLang(lang) {
            currentInputLang = lang;
            const buttons = ['ko', 'ru', 'zh'];
            buttons.forEach(l => {
                const btn = document.getElementById(`lang-${l}`);
                if(l === lang) {
                    btn.className = 'flex-1 py-2 rounded-xl text-xs font-bold transition-all bg-white shadow-sm text-indigo-600';
                } else {
                    btn.className = 'flex-1 py-2 rounded-xl text-xs font-bold transition-all text-slate-400 hover:text-slate-600';
                }
            });
        }

        async function fetchTranslation(text, source, target) {
            const url = `https://translation.googleapis.com/language/translate/v2?key=${API_KEY}`;
            const response = await fetch(url, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ q: text, source: source, target: target, format: 'text' })
            });
            const data = await response.json();
            if (!response.ok) throw new Error(data.error?.message || "번역 실패");
            return data.data.translations[0].translatedText;
        }

        function speak(text, lang) {
            if (!window.speechSynthesis) return;
            window.speechSynthesis.cancel();
            const utterance = new SpeechSynthesisUtterance(text);
            const langMap = { 'ko': 'ko-KR', 'ru': 'ru-RU', 'zh': 'zh-CN' };
            utterance.lang = langMap[lang] || lang;
            window.speechSynthesis.speak(utterance);
        }

        function hideError() {
            document.getElementById('error-banner').classList.add('hidden');
        }

        async function handleSend() {
            const input = document.getElementById('text-input');
            const text = input.value.trim();
            if (!text) return;

            input.value = '';
            document.getElementById('loading').classList.remove('hidden');
            hideError();

            try {
                let transData = {};
                if (currentInputLang === 'ko') {
                    const [ru, zh] = await Promise.all([
                        fetchTranslation(text, 'ko', 'ru'),
                        fetchTranslation(text, 'ko', 'zh')
                    ]);
                    transData = { 'ru': ru, 'zh': zh };
                } else if (currentInputLang === 'ru') {
                    const [ko, zh] = await Promise.all([
                        fetchTranslation(text, 'ru', 'ko'),
                        fetchTranslation(text, 'ru', 'zh')
                    ]);
                    transData = { 'ko': ko, 'zh': zh };
                } else {
                    const [ko, ru] = await Promise.all([
                        fetchTranslation(text, 'zh', 'ko'),
                        fetchTranslation(text, 'zh', 'ru')
                    ]);
                    transData = { 'ko': ko, 'ru': ru };
                }

                addMessageToUI(text, currentInputLang, transData);
            } catch (error) {
                console.error(error);
                document.getElementById('error-text').innerText = error.message.includes("blocked") ? "API 차단됨: 구글 콘솔 설정을 확인하세요." : "번역 중 오류가 발생했습니다.";
                document.getElementById('error-banner').classList.remove('hidden');
            } finally {
                document.getElementById('loading').classList.add('hidden');
            }
        }

        function addMessageToUI(original, sourceLang, translations) {
            const display = document.getElementById('chat-display');
            const bubble = document.createElement('div');
            bubble.className = `flex flex-col ${currentRole === 'teacher' ? 'items-start' : 'items-end'} message-appear`;

            let transHtml = '';
            for (const [lang, text] of Object.entries(translations)) {
                const langLabel = lang === 'ko' ? '🇰🇷 KOR' : (lang === 'ru' ? '🇷🇺 RUS' : '🇨🇳 CHI');
                transHtml += `
                    <div class="flex items-start gap-3 p-3 bg-slate-50 rounded-2xl border border-slate-100">
                        <div class="flex-1">
                            <span class="text-[10px] font-black text-slate-400 uppercase">${langLabel}</span>
                            <p class="text-base font-medium text-slate-700 leading-snug">${text}</p>
                        </div>
                        <button onclick="speak('${text.replace(/'/g, "\\'")}', '${lang}')" class="p-1.5 text-slate-300 hover:text-indigo-600 transition-colors">
                            <i class="fa-solid fa-volume-high"></i>
                        </button>
                    </div>
                `;
            }

            const time = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
            const sourceLabel = sourceLang === 'ko' ? 'KOR' : (sourceLang === 'ru' ? 'RUS' : 'CHI');

            bubble.innerHTML = `
                <div class="max-w-[90%] rounded-3xl p-5 shadow-sm relative ${currentRole === 'teacher' ? 'bg-white border border-slate-200 rounded-tl-none' : 'bg-white border-2 border-indigo-600 rounded-tr-none'}">
                    <div class="flex justify-between items-start gap-4 mb-4">
                        <div class="flex-1">
                            <div class="flex items-center gap-2 mb-1">
                                <span class="text-[10px] font-black text-indigo-600 bg-indigo-50 px-1.5 py-0.5 rounded uppercase">${sourceLabel}</span>
                                <span class="text-[10px] text-slate-400 font-bold">${currentRole === 'teacher' ? '선생님' : '학생'}</span>
                            </div>
                            <p class="font-bold text-xl leading-tight text-slate-900">${original}</p>
                        </div>
                        <button onclick="speak('${original.replace(/'/g, "\\'")}', '${sourceLang}')" class="p-2 bg-slate-100 hover:bg-indigo-100 text-indigo-600 rounded-full transition-colors">
                            <i class="fa-solid fa-volume-high"></i>
                        </button>
                    </div>
                    <div class="space-y-3 pt-2 border-t border-slate-50">${transHtml}</div>
                    <span class="text-[9px] font-bold absolute -bottom-5 ${currentRole === 'teacher' ? 'left-2' : 'right-2'} text-slate-400">${time}</span>
                </div>
            `;

            display.appendChild(bubble);
            display.scrollTop = display.scrollHeight;
        }

        // Enter key support
        document.getElementById('text-input').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') handleSend();
        });
    </script>
</body>
</html>
