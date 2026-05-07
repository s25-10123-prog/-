```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>타자 연습 마스터</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;700&display=swap');
        
        body {
            font-family: 'Noto Sans KR', sans-serif;
            background-color: #f0f4f8;
        }

        .word-display {
            font-size: 3rem;
            min-height: 4.5rem;
            transition: all 0.2s ease;
        }

        .correct { color: #10b981; }
        .incorrect { color: #ef4444; }
        
        .stats-card {
            background: white;
            border-radius: 1rem;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }

        .input-area:focus {
            outline: none;
            border-color: #3b82f6;
            box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.3);
        }

        .input-area:disabled {
            background-color: #f9fafb;
            cursor: not-allowed;
        }

        /* Start Overlay */
        #start-overlay {
            z-index: 10;
        }
    </style>
</head>
<body class="flex flex-col min-h-screen items-center justify-center p-4">

    <div class="w-full max-w-2xl relative">
        <!-- Header -->
        <div class="text-center mb-8">
            <h1 class="text-4xl font-bold text-gray-800 mb-2">⌨️ 타자 연습 마스터</h1>
            <p class="text-gray-600">속도와 정확도를 높여보세요!</p>
        </div>

        <!-- Settings -->
        <div class="flex justify-center gap-4 mb-6">
            <button id="mode-ko" class="px-4 py-2 rounded-full bg-blue-600 text-white font-medium shadow-md transition-all hover:bg-blue-700">한글 모드</button>
            <button id="mode-en" class="px-4 py-2 rounded-full bg-gray-200 text-gray-700 font-medium shadow-sm transition-all hover:bg-gray-300">English Mode</button>
        </div>

        <!-- Main Content Area with Start Button Overlay -->
        <div class="relative">
            <!-- Start Button Overlay -->
            <div id="start-overlay" class="absolute inset-0 bg-white/80 backdrop-blur-sm flex items-center justify-center rounded-xl transition-opacity duration-300">
                <button id="start-game-btn" class="px-10 py-5 bg-blue-600 text-white text-2xl font-bold rounded-2xl shadow-xl hover:bg-blue-700 hover:scale-105 transition-all">
                    시작하기
                </button>
            </div>

            <div class="stats-card p-8 text-center">
                <!-- Current Word -->
                <div id="word-target" class="word-display font-bold text-gray-700 mb-6 flex justify-center items-center">
                    준비되셨나요?
                </div>

                <!-- Input Box -->
                <input type="text" id="typing-input" 
                       class="input-area w-full p-4 text-2xl text-center border-2 border-gray-200 rounded-xl mb-6"
                       placeholder="여기에 단어를 입력하세요..." autocomplete="off" disabled>

                <!-- Stats Bar -->
                <div class="grid grid-cols-3 gap-4 border-t pt-6">
                    <div>
                        <p class="text-gray-500 text-sm mb-1">타수 (WPM)</p>
                        <p id="stat-wpm" class="text-2xl font-bold text-blue-600">0</p>
                    </div>
                    <div>
                        <p class="text-gray-500 text-sm mb-1">정확도</p>
                        <p id="stat-accuracy" class="text-2xl font-bold text-green-600">100%</p>
                    </div>
                    <div>
                        <p class="text-gray-500 text-sm mb-1">남은 시간</p>
                        <p id="stat-timer" class="text-2xl font-bold text-red-500">60s</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- Controls -->
        <div class="mt-8 flex justify-center">
            <button id="restart-btn" class="flex items-center gap-2 px-6 py-3 bg-gray-800 text-white rounded-lg hover:bg-gray-900 transition-colors">
                <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 12a9 9 0 0 1 9-9 9.75 9.75 0 0 1 6.74 2.74L21 8"></path><path d="M21 3v5h-5"></path><path d="M21 12a9 9 0 0 1-9 9 9.75 9.75 0 0 1-6.74-2.74L3 16"></path><path d="M8 16H3v5"></path></svg>
                재설정
            </button>
        </div>
    </div>

    <!-- Notification Overlay (Game Over) -->
    <div id="game-over-modal" class="fixed inset-0 bg-black/50 hidden z-50 flex items-center justify-center p-4">
        <div class="bg-white p-8 rounded-2xl max-w-sm w-full text-center shadow-2xl">
            <h2 class="text-3xl font-bold mb-4">게임 종료!</h2>
            <div class="space-y-2 mb-6">
                <p class="text-lg">최종 속도: <span id="final-wpm" class="font-bold text-blue-600">0</span> WPM</p>
                <p class="text-lg">정확도: <span id="final-accuracy" class="font-bold text-green-600">0%</span></p>
            </div>
            <button id="modal-close" class="w-full py-3 bg-blue-600 text-white rounded-xl font-bold hover:bg-blue-700">확인</button>
        </div>
    </div>

    <script>
        const wordsKo = ["하늘", "바다", "나무", "구름", "대한민국", "컴퓨터", "프로그래밍", "타이핑", "커피", "노트북", "안녕", "미래", "성공", "노력", "바람", "여름", "겨울", "단풍", "우주", "지구", "태양", "사랑", "행복", "즐거움", "기억", "오늘", "내일", "사과", "바나나", "학교", "공부", "운동", "수영", "자전거", "등산", "여행", "사진", "음악", "영화", "독서"];
        const wordsEn = ["sky", "ocean", "tree", "cloud", "korea", "computer", "programming", "typing", "coffee", "laptop", "hello", "future", "success", "effort", "wind", "summer", "winter", "autumn", "space", "earth", "sun", "love", "happy", "joy", "memory", "today", "tomorrow", "apple", "banana", "school", "study", "exercise", "swim", "bicycle", "hiking", "travel", "photo", "music", "movie", "reading"];

        let currentWords = wordsKo;
        let currentTarget = "";
        let timer = 60;
        let interval = null;
        let isPlaying = false;
        let score = 0;
        let totalTyped = 0;
        let correctTyped = 0;

        const wordDisplay = document.getElementById('word-target');
        const inputField = document.getElementById('typing-input');
        const wpmDisplay = document.getElementById('stat-wpm');
        const accuracyDisplay = document.getElementById('stat-accuracy');
        const timerDisplay = document.getElementById('stat-timer');
        const restartBtn = document.getElementById('restart-btn');
        const modeKoBtn = document.getElementById('mode-ko');
        const modeEnBtn = document.getElementById('mode-en');
        
        const startOverlay = document.getElementById('start-overlay');
        const startGameBtn = document.getElementById('start-game-btn');
        
        const modal = document.getElementById('game-over-modal');
        const finalWpm = document.getElementById('final-wpm');
        const finalAccuracy = document.getElementById('final-accuracy');
        const modalClose = document.getElementById('modal-close');

        // Initial State
        function initApp() {
            clearInterval(interval);
            isPlaying = false;
            timer = 60;
            score = 0;
            totalTyped = 0;
            correctTyped = 0;
            inputField.value = "";
            inputField.disabled = true;
            timerDisplay.innerText = `60s`;
            wpmDisplay.innerText = "0";
            accuracyDisplay.innerText = "100%";
            wordDisplay.innerText = "준비되셨나요?";
            wordDisplay.className = "word-display font-bold text-gray-400 mb-6 flex justify-center items-center";
            startOverlay.classList.remove('hidden');
        }

        function startGame() {
            startOverlay.classList.add('hidden');
            inputField.disabled = false;
            inputField.focus();
            setNewWord();
            startTimer();
        }

        function setNewWord() {
            const randomIndex = Math.floor(Math.random() * currentWords.length);
            currentTarget = currentWords[randomIndex];
            wordDisplay.innerText = currentTarget;
            wordDisplay.className = "word-display font-bold text-gray-700 mb-6 flex justify-center items-center";
        }

        function startTimer() {
            isPlaying = true;
            interval = setInterval(() => {
                timer--;
                timerDisplay.innerText = `${timer}s`;
                
                const timePassed = (60 - timer) / 60;
                const wpm = Math.round((score) / (timePassed || 1));
                wpmDisplay.innerText = wpm;

                if (timer <= 0) {
                    endGame();
                }
            }, 1000);
        }

        function endGame() {
            clearInterval(interval);
            isPlaying = false;
            inputField.disabled = true;
            
            const accuracy = totalTyped === 0 ? 0 : Math.round((correctTyped / totalTyped) * 100);
            const timePassed = (60 - timer) / 60;
            const wpm = Math.round((score) / (timePassed || 1));

            finalWpm.innerText = wpm;
            finalAccuracy.innerText = accuracy + "%";
            modal.classList.remove('hidden');
        }

        inputField.addEventListener('input', (e) => {
            if (!isPlaying) return;

            const val = e.target.value.trim();
            
            if (currentTarget.startsWith(val)) {
                wordDisplay.classList.remove('incorrect');
                wordDisplay.classList.add('correct');
            } else {
                wordDisplay.classList.remove('correct');
                wordDisplay.classList.add('incorrect');
            }

            if (val === currentTarget) {
                score++;
                correctTyped += currentTarget.length;
                totalTyped += currentTarget.length;
                e.target.value = "";
                setNewWord();
                
                const accuracy = Math.round((correctTyped / totalTyped) * 100);
                accuracyDisplay.innerText = accuracy + "%";
            }
        });

        inputField.addEventListener('keydown', (e) => {
            if (e.key.length === 1 && isPlaying) {
                totalTyped++;
            }
        });

        startGameBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', initApp);
        
        modalClose.addEventListener('click', () => {
            modal.classList.add('hidden');
            initApp();
        });

        modeKoBtn.addEventListener('click', () => {
            currentWords = wordsKo;
            modeKoBtn.className = "px-4 py-2 rounded-full bg-blue-600 text-white font-medium shadow-md transition-all hover:bg-blue-700";
            modeEnBtn.className = "px-4 py-2 rounded-full bg-gray-200 text-gray-700 font-medium shadow-sm transition-all hover:bg-gray-300";
            initApp();
        });

        modeEnBtn.addEventListener('click', () => {
            currentWords = wordsEn;
            modeEnBtn.className = "px-4 py-2 rounded-full bg-blue-600 text-white font-medium shadow-md transition-all hover:bg-blue-700";
            modeKoBtn.className = "px-4 py-2 rounded-full bg-gray-200 text-gray-700 font-medium shadow-sm transition-all hover:bg-gray-300";
            initApp();
        });

        window.onload = initApp;
    </script>
</body>
</html>

```
