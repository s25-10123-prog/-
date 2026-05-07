```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>타자 연습 마스터</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700&display=swap');
        
        body {
            font-family: 'Noto Sans KR', sans-serif;
            background-color: #f3f4f6;
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .word-display {
            font-size: 3.5rem;
            min-height: 5rem;
            display: flex;
            justify-content: center;
            align-items: center;
            font-weight: 700;
        }

        .correct { color: #10b981; }
        .incorrect { color: #ef4444; }
        
        .input-box:focus {
            outline: none;
            border-color: #3b82f6;
            box-shadow: 0 0 0 4px rgba(59, 130, 246, 0.2);
        }

        /* 시작 오버레이 스타일 - 초기 상태를 flex로 설정 */
        #overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.95);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 50;
            border-radius: 1rem;
            transition: opacity 0.3s ease;
        }

        /* 오버레이 숨김 클래스 */
        .hide-overlay {
            display: none !important;
        }
    </style>
</head>
<body>

    <div class="w-full max-w-xl p-4">
        <!-- 제목 -->
        <div class="text-center mb-6">
            <h1 class="text-3xl font-bold text-gray-800">⌨️ 타자 연습</h1>
            <p class="text-gray-500 mt-1">단어를 정확하고 빠르게 입력하세요</p>
        </div>

        <!-- 게임 컨테이너 -->
        <div class="relative bg-white p-8 rounded-2xl shadow-xl min-h-[400px] flex flex-col justify-center">
            
            <!-- 시작 오버레이 -->
            <div id="overlay">
                <div class="mb-6 space-x-2">
                    <button id="lang-ko" onclick="setLanguage('ko')" class="px-5 py-2 bg-blue-600 text-white rounded-full font-bold transition-colors">한글</button>
                    <button id="lang-en" onclick="setLanguage('en')" class="px-5 py-2 bg-gray-200 text-gray-700 rounded-full font-bold hover:bg-gray-300 transition-colors">English</button>
                </div>
                <button id="start-btn" class="px-10 py-5 bg-blue-600 text-white text-2xl font-bold rounded-2xl shadow-lg hover:bg-blue-700 transform transition hover:scale-105 active:scale-95">
                    시작하기
                </button>
                <p class="mt-4 text-sm text-gray-400">시작 버튼을 누르면 타이머가 작동합니다</p>
            </div>

            <!-- 타겟 단어 표시 -->
            <div id="word-target" class="word-display text-gray-300 mb-8">
                준비완료!
            </div>

            <!-- 입력창 -->
            <input type="text" id="input-field" 
                   class="input-box w-full p-4 text-2xl text-center border-2 border-gray-200 rounded-xl mb-8"
                   placeholder="여기에 입력..." autocomplete="off">

            <!-- 통계 섹션 -->
            <div class="grid grid-cols-3 gap-4 text-center border-t pt-6">
                <div>
                    <div class="text-gray-400 text-xs font-bold uppercase mb-1">속도 (WPM)</div>
                    <div id="stat-wpm" class="text-2xl font-bold text-blue-500">0</div>
                </div>
                <div>
                    <div class="text-gray-400 text-xs font-bold uppercase mb-1">정확도</div>
                    <div id="stat-accuracy" class="text-2xl font-bold text-green-500">100%</div>
                </div>
                <div>
                    <div class="text-gray-400 text-xs font-bold uppercase mb-1">남은 시간</div>
                    <div id="stat-timer" class="text-2xl font-bold text-red-500">60s</div>
                </div>
            </div>
        </div>

        <!-- 하단 재설정 -->
        <div class="text-center mt-6">
            <button onclick="resetGame()" class="text-gray-400 hover:text-gray-600 underline text-sm">처기화 및 처음으로</button>
        </div>
    </div>

    <!-- 결과 모달 -->
    <div id="result-modal" class="fixed inset-0 bg-black/60 hidden items-center justify-center z-[100] p-4">
        <div class="bg-white p-8 rounded-2xl w-full max-w-xs text-center shadow-2xl border border-gray-100">
            <h2 class="text-2xl font-bold mb-4 text-gray-800">게임 결과</h2>
            <div class="bg-gray-50 p-4 rounded-xl mb-6 space-y-3 text-left">
                <div class="flex justify-between border-b border-gray-200 pb-2">
                    <span class="text-gray-500">평균 속도</span>
                    <span id="res-wpm" class="font-bold text-blue-600">0 WPM</span>
                </div>
                <div class="flex justify-between">
                    <span class="text-gray-500">입력 정확도</span>
                    <span id="res-acc" class="font-bold text-green-600">0%</span>
                </div>
            </div>
            <button onclick="resetGame()" class="w-full py-4 bg-blue-600 text-white rounded-xl font-bold hover:bg-blue-700 transition-colors">다시 도전하기</button>
        </div>
    </div>

    <script>
        // 데이터셋
        const data = {
            ko: ["하늘", "바다", "나무", "구름", "컴퓨터", "바람", "여름", "겨울", "성공", "노력", "우주", "지구", "태양", "사랑", "행복", "즐거움", "기억", "오늘", "내일", "학교", "공부", "운동", "사진", "음악", "영화", "독서", "사람", "친구", "가족", "인사", "포기", "도전", "열정", "창의", "희망", "미소", "풍경", "여행", "지도", "시계"],
            en: ["apple", "banana", "coffee", "laptop", "future", "success", "effort", "happy", "memory", "today", "tomorrow", "school", "music", "photo", "coding", "design", "python", "script", "world", "planet", "science", "travel", "winter", "summer", "ocean", "forest", "friend", "family", "dream", "vision", "orange", "yellow", "purple", "rocket", "energy", "silver", "button", "screen", "mobile", "player"]
        };

        // 상태 변수
        let currentLang = 'ko';
        let timer = 60;
        let score = 0;
        let totalInput = 0;
        let correctInput = 0;
        let isPlaying = false;
        let timerInterval = null;
        let targetWord = "";

        // DOM 요소
        const wordDisplay = document.getElementById('word-target');
        const inputField = document.getElementById('input-field');
        const startBtn = document.getElementById('start-btn');
        const overlay = document.getElementById('overlay');
        const timerText = document.getElementById('stat-timer');
        const wpmText = document.getElementById('stat-wpm');
        const accText = document.getElementById('stat-accuracy');
        const resultModal = document.getElementById('result-modal');

        // 언어 설정
        function setLanguage(lang) {
            currentLang = lang;
            document.getElementById('lang-ko').className = lang === 'ko' ? "px-5 py-2 bg-blue-600 text-white rounded-full font-bold transition-colors" : "px-5 py-2 bg-gray-200 text-gray-700 rounded-full font-bold hover:bg-gray-300 transition-colors";
            document.getElementById('lang-en').className = lang === 'en' ? "px-5 py-2 bg-blue-600 text-white rounded-full font-bold transition-colors" : "px-5 py-2 bg-gray-200 text-gray-700 rounded-full font-bold hover:bg-gray-300 transition-colors";
        }

        // 새로운 단어 출제
        function nextWord() {
            const list = data[currentLang];
            targetWord = list[Math.floor(Math.random() * list.length)];
            wordDisplay.innerText = targetWord;
            wordDisplay.className = "word-display text-gray-700";
            inputField.value = "";
        }

        // 타이머 시작
        function startTimer() {
            timerInterval = setInterval(() => {
                timer--;
                timerText.innerText = timer + "s";
                
                // 실시간 속도 계산
                const timeElapsed = (60 - timer) / 60;
                const wpm = Math.round(score / (timeElapsed || 1));
                wpmText.innerText = wpm;

                if (timer <= 0) {
                    endGame();
                }
            }, 1000);
        }

        // 게임 종료
        function endGame() {
            clearInterval(timerInterval);
            isPlaying = false;
            inputField.disabled = true;
            
            const accuracy = totalInput === 0 ? 0 : Math.round((correctInput / totalInput) * 100);
            document.getElementById('res-wpm').innerText = wpmText.innerText + " WPM";
            document.getElementById('res-acc').innerText = accuracy + "%";
            
            resultModal.classList.replace('hidden', 'flex');
        }

        // 게임 리셋/초기화
        function resetGame() {
            clearInterval(timerInterval);
            isPlaying = false;
            timer = 60;
            score = 0;
            totalInput = 0;
            correctInput = 0;
            
            timerText.innerText = "60s";
            wpmText.innerText = "0";
            accText.innerText = "100%";
            inputField.value = "";
            inputField.disabled = false;
            wordDisplay.innerText = "준비완료!";
            wordDisplay.className = "word-display text-gray-300";
            
            resultModal.classList.replace('flex', 'hidden');
            overlay.classList.remove('hide-overlay'); // 오버레이 다시 보이기
        }

        // 입력 체크
        inputField.addEventListener('input', (e) => {
            if (!isPlaying) return;

            const val = e.target.value.trim();
            
            // 시각 피드백
            if (targetWord.startsWith(val)) {
                wordDisplay.classList.remove('incorrect');
                wordDisplay.classList.add('correct');
            } else {
                wordDisplay.classList.remove('correct');
                wordDisplay.classList.add('incorrect');
            }

            // 단어 완성 시
            if (val === targetWord) {
                score++;
                correctInput += targetWord.length;
                totalInput += targetWord.length;
                
                const accuracy = Math.round((correctInput / totalInput) * 100);
                accText.innerText = accuracy + "%";
                
                nextWord();
            }
        });

        // 정확도 계산을 위한 입력 추적
        inputField.addEventListener('keydown', (e) => {
            if (!isPlaying) return;
            // 특수키 제외하고 실제 글자 입력 시에만 totalInput 증가
            if (e.key.length === 1 && e.key !== 'Enter' && e.key !== 'Tab') {
                totalInput++;
            }
        });

        // 시작 버튼 이벤트
        startBtn.addEventListener('click', () => {
            overlay.classList.add('hide-overlay'); // 오버레이 확실하게 제거
            isPlaying = true;
            inputField.disabled = false;
            inputField.focus(); // 입력창에 바로 커서 두기
            nextWord();
            startTimer();
        });

        // 초기 상태 설정
        window.onload = () => {
            resetGame();
        };
    </script>
</body>
</html>

```
