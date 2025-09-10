<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>커리어 유형 진단 테스트</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Noto Sans KR', sans-serif;
        }
        /* Custom styles for radio button labels */
        .radio-label {
            transition: all 0.2s ease-in-out;
        }
        input[type="radio"]:checked + .radio-label {
            background-color: #4f46e5; /* indigo-600 */
            color: white;
            border-color: #4f46e5; /* indigo-600 */
        }
        /* Fade-in animation */
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .fade-in {
            animation: fadeIn 0.5s ease-in-out forwards;
        }
        #question-image-container {
            height: 200px; /* Set a fixed height for the image container */
        }
        #question-image {
            width: 100%;
            height: 100%;
            object-fit: contain; /* Ensures the image fits without being cropped or stretched */
        }
    </style>
</head>
<body class="bg-gray-100 dark:bg-gray-900 text-gray-800 dark:text-gray-200 flex items-center justify-center min-h-screen p-4">

    <div id="test-container" class="bg-white dark:bg-gray-800 shadow-2xl rounded-2xl p-6 sm:p-10 w-full max-w-2xl transition-all duration-300">

        <!-- Start Screen -->
        <div id="start-screen" class="text-center fade-in">
            <h1 class="text-3xl sm:text-4xl font-bold text-indigo-600 dark:text-indigo-400 mb-2">회사가 지옥처럼 느껴진다면?</h1>
            <p class="text-lg sm:text-xl text-gray-600 dark:text-gray-400 mb-8">지금 내 모습, 정말 괜찮은 걸까?<br>간단한 테스트로 나의 커리어 유형을 확인해보세요.</p>
            <button id="start-btn" class="w-full sm:w-auto bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-8 rounded-lg text-lg transition-transform transform hover:scale-105 focus:outline-none focus:ring-4 focus:ring-indigo-300 dark:focus:ring-indigo-800">
                테스트 시작하기
            </button>
        </div>

        <!-- Question Screen -->
        <div id="question-screen" class="hidden">
            <div class="mb-6">
                <div class="w-full bg-gray-200 dark:bg-gray-700 rounded-full h-2.5">
                    <div id="progress-bar" class="bg-indigo-600 h-2.5 rounded-full transition-all duration-300" style="width: 0%"></div>
                </div>
                <p id="progress-text" class="text-center text-sm text-gray-500 mt-2">0 / 10</p>
            </div>
            <div id="question-content-area">
                <!-- Question content will be injected here -->
            </div>
        </div>

        <!-- Result Screen -->
        <div id="result-screen" class="hidden text-center">
            <!-- Result content will be injected here -->
        </div>
    </div>

    <script>
        // --- DATA ---
        // V V V V V  이미지 경로 안내 V V V V V
        // GitHub Pages에서 이미지를 올바르게 표시하기 위해 경로를 "images/파일이름" 형식으로 설정했습니다.
        // 예를 들어, 1번 질문에는 'images' 폴더 안에 있는 '01.png' 파일이 연결됩니다.
        // 가지고 계신 10개의 그림 파일 이름을 01.png, 02.png, ... , 10.png 로 맞춰주세요.
        const questions = [
            { id: 1, text: "나는 내 일이 누군가에게 혹은 사회에 의미 있는 영향을 주고 있다고 느낀다.", image: "images/01.png" },
            { id: 2, text: "나는 업무를 통해 새로운 것을 배우고 성장하고 있다는 느낌을 받는다.", image: "images/02.png" },
            { id: 3, text: "나는 업무의 상당 부분을 스스로의 판단과 자율성을 가지고 처리할 수 있다.", image: "images/03.png" },
            { id: 4, text: "나는 회사 동료들과 건전한 관계를 맺고 있으며, 협업 과정이 즐거울 때가 있다.", image: "images/04.png" },
            { id: 5, text: "나는 퇴근 후에도 다른 무언가를 배울 수 있는 정신적, 육체적 에너지가 남아있다.", image: "images/05.png" },
            { id: 6, text: "나는 회사의 가치관이나 방향성이 나의 개인적인 신념과 크게 부딪히지 않는다고 생각한다.", image: "images/06.png" },
            { id: 7, text: "나는 1년 뒤 나의 모습이 지금보다 더 나아져 있을 것이라는 기대감이 있다.", image: "images/07.png" },
            { id: 8, text: "나는 월급 외에 이 일을 통해 얻는 만족감이나 성취감이 분명히 존재한다.", image: "images/08.png" },
            { id: 9, text: "나는 아침에 일어날 때, 출근할 생각에 마음이 무겁지 않은 날이 더 많다.", image: "images/09.png" },
            { id: 10, text: "만약 다시 직업을 선택할 기회가 와도, 현재와 비슷한 분야의 일을 고려할 것 같다.", image: "images/10.png" }
        ];

        const results = [
            {
                score: 10,
                type: "⌛ 의무적 생존가",
                title: "월급 날만을 기다리며, 영혼 없이 시간을 견디는 중.",
                features: [
                    "일에 대한 의미나 목적의식을 거의 느끼지 못하고, 오직 생계를 위해 일합니다.",
                    "자율성이나 성장 가능성이 거의 없는 환경에 무기력하게 적응한 상태입니다.",
                    "회사 안에서의 '나'와 진짜 '나'의 괴리감이 매우 큽니다."
                ],
                nextStep: "'월급으로부터의 독립'은 먼 이야기처럼 느껴지겠지만, 지금 당장 아주 작은 것부터 시작해야 합니다. 당신의 마음을 뛰게 했던 것이 무엇인지, 아주 사소한 것이라도 다시 찾아보세요. 회사 밖에서 **나의 가능성에 도전하는 작은 씨앗**을 심는 것이 변화의 첫걸음입니다."
            },
            {
                score: 15,
                type: "🔥 소울 버너",
                title: "에너지가 소진되어, 재만 남은 기분.",
                features: [
                    "과도한 업무, 불합리한 조직 문화 등으로 인해 번아웃 직전이거나 이미 겪고 있습니다.",
                    "일에 대한 보람이나 성취감보다 스트레스와 압박감이 훨씬 큽니다.",
                    "퇴근 후에는 아무것도 할 수 없을 만큼 정신적, 육체적 에너지가 고갈된 상태입니다."
                ],
                nextStep: "가장 시급한 것은 **'나를 지키는 것'**입니다. 의식적으로 휴식 시간을 확보하고, 일과 나를 분리하는 연습이 필요합니다. 거절하는 용기, 도움을 요청할 용기를 내어 스스로를 보호해야 할 때입니다."
            },
            {
                score: 24,
                type: "🤔 현실적 탐색가",
                title: "이 길이 맞나? 고민과 현실 사이, 갈림길에 서다.",
                features: [
                    "'이 일을 계속하는 게 맞을까?'라는 질문을 스스로에게 자주 던집니다.",
                    "일에서 만족을 느끼는 순간도 있지만, 무의미함과 회의감을 느끼는 순간도 잦습니다.",
                    "이직이나 새로운 도전을 막연히 생각하지만, 현실적인 조건 때문에 망설이고 있습니다."
                ],
                nextStep: "본문 글의 주인공처럼, **내가 주로 가는 곳, 만나는 사람들을 서서히 바꿔보세요.** 사이드 프로젝트, 스터디, 새로운 취미 등 회사 밖 세상과 연결될수록 내가 진짜 원하는 삶의 모양이 선명해질 거예요."
            },
            {
                score: 33,
                type: "🧭 안정적 항해사",
                title: "큰 파도는 없지만, 나의 길을 꾸준히 나아가는 중.",
                features: [
                    "현재 직장에 큰 불만은 없으며, 안정적인 생활에 만족하는 편입니다.",
                    "익숙한 업무를 능숙하게 처리하지만, 가슴 뛰는 열정이나 도전은 다소 부족할 수 있습니다.",
                    "'워라밸'을 중요하게 생각하며, 회사 밖에서 삶의 의미를 찾기도 합니다."
                ],
                nextStep: "안정감에 안주하기보다, 현재 업무에 적용할 수 있는 작은 변화(새로운 툴 배우기, 업무 프로세스 개선 등)를 시도해보세요. 작은 도전이 새로운 활력을 줄 수 있습니다."
            },
            {
                score: 42,
                type: "🌱 성장 개척가",
                title: "일과 내가 함께 성장하는 최고의 파트너!",
                features: [
                    "현재 자신의 일에 높은 만족도와 자부심을 느끼고 있습니다.",
                    "업무를 통해 배우고 도전하며, 스스로의 성장을 명확히 체감합니다.",
                    "회사와 나의 비전이 같은 방향을 보고 있어 긍정적인 시너지를 냅니다."
                ],
                nextStep: "현재의 좋은 흐름을 유지하며, 자신의 경험을 주변에 나누는 멘토 역할을 해보세요. 당신의 긍정적 에너지가 새로운 기회를 이끌어줄 거예요!"
            }
        ];
        
        const answerOptions = [
            { score: 1, text: "전혀 그렇지 않다" },
            { score: 2, text: "그렇지 않은 편이다" },
            { score: 3, text: "보통이다" },
            { score: 4, text: "그런 편이다" },
            { score: 5, text: "매우 그렇다" }
        ];

        // --- DOM Elements ---
        const startScreen = document.getElementById('start-screen');
        const questionScreen = document.getElementById('question-screen');
        const resultScreen = document.getElementById('result-screen');
        const startBtn = document.getElementById('start-btn');
        const questionContentArea = document.getElementById('question-content-area');
        const progressBar = document.getElementById('progress-bar');
        const progressText = document.getElementById('progress-text');

        // --- State Variables ---
        let currentQuestionIndex = 0;
        let userScores = [];

        // --- Functions ---
        
        function showQuestion(index) {
            if (index >= questions.length) {
                calculateAndShowResult();
                return;
            }

            const question = questions[index];
            
            const optionsHTML = answerOptions.map(opt => `
                <div>
                    <input type="radio" name="answer" id="q${index}_opt_${opt.score}" value="${opt.score}" class="sr-only">
                    <label for="q${index}_opt_${opt.score}" class="radio-label block cursor-pointer border-2 border-gray-300 dark:border-gray-600 rounded-md py-3 px-1 text-xs sm:text-sm font-medium hover:bg-indigo-100 dark:hover:bg-gray-700">${opt.text}</label>
                </div>
            `).join('');

            const questionHTML = `
                <div class="fade-in text-center">
                    <div id="question-image-container" class="mb-5 rounded-lg overflow-hidden flex items-center justify-center bg-gray-200 dark:bg-gray-700">
                         <img id="question-image" src="${question.image}" alt="질문 ${question.id} 이미지" class="transition-opacity duration-300 opacity-0" onload="this.style.opacity=1" onerror="this.src='https://placehold.co/600x300/e2e8f0/4a5568?text=Image+Not+Found'">
                    </div>
                    <p class="text-lg font-semibold mb-5 h-16 flex items-center justify-center">${question.id}. ${question.text}</p>
                    <div class="grid grid-cols-5 gap-1 sm:gap-2 text-center">
                        ${optionsHTML}
                    </div>
                </div>`;
            
            questionContentArea.innerHTML = questionHTML;

            const radioButtons = questionContentArea.querySelectorAll('input[type="radio"]');
            radioButtons.forEach(radio => {
                radio.addEventListener('change', handleAnswerSelection);
            });
        }

        function handleAnswerSelection(event) {
            questionContentArea.querySelectorAll('input[type="radio"]').forEach(radio => radio.disabled = true);
            
            userScores[currentQuestionIndex] = parseInt(event.target.value);
            currentQuestionIndex++;
            updateProgress();

            setTimeout(() => {
                showQuestion(currentQuestionIndex);
            }, 300);
        }

        function updateProgress() {
            const progress = (currentQuestionIndex / questions.length) * 100;
            progressBar.style.width = `${progress}%`;
            progressText.textContent = `${currentQuestionIndex} / ${questions.length}`;
        }
        
        function calculateAndShowResult() {
            const totalScore = userScores.reduce((sum, score) => sum + score, 0);

            let finalResult = results[0];
            // Find the correct result type by checking scores from lowest to highest
            for (const result of [...results].reverse()) {
                if (totalScore < result.score) {
                    finalResult = result;
                } else {
                    break;
                }
            }
             // Handle the highest score case separately
            if (totalScore >= 42) {
                finalResult = results.find(r => r.score === 42);
            } else if (totalScore >= 33) {
                 finalResult = results.find(r => r.score === 33);
            } else if (totalScore >= 24) {
                 finalResult = results.find(r => r.score === 24);
            } else if (totalScore >= 15) {
                 finalResult = results.find(r => r.score === 15);
            } else {
                 finalResult = results.find(r => r.score === 10);
            }


            resultScreen.innerHTML = `
                <div class="fade-in">
                    <p class="text-2xl font-bold text-indigo-500 mb-2">${finalResult.type}</p>
                    <h2 class="text-2xl sm:text-3xl font-bold mb-6">${finalResult.title}</h2>
                    
                    <div class="text-left bg-gray-50 dark:bg-gray-700/50 p-6 rounded-lg mb-6">
                        <h3 class="font-bold text-lg mb-3">특징:</h3>
                        <ul class="list-disc list-inside space-y-2 text-gray-700 dark:text-gray-300">
                            ${finalResult.features.map(f => `<li>${f}</li>`).join('')}
                        </ul>
                    </div>

                    <div class="text-left bg-indigo-50 dark:bg-indigo-900/40 p-6 rounded-lg mb-8 border-l-4 border-indigo-500">
                        <h3 class="font-bold text-lg mb-3 text-indigo-800 dark:text-indigo-300">NEXT STEP:</h3>
                        <p class="text-indigo-900 dark:text-indigo-200">${finalResult.nextStep}</p>
                    </div>

                    <button id="restart-btn" class="w-full sm:w-auto bg-gray-600 hover:bg-gray-700 text-white font-bold py-3 px-8 rounded-lg text-lg transition-transform transform hover:scale-105 focus:outline-none focus:ring-4 focus:ring-gray-300 dark:focus:ring-gray-500">
                        테스트 다시하기
                    </button>
                </div>
            `;
            
            questionScreen.classList.add('hidden');
            resultScreen.classList.remove('hidden');

            document.getElementById('restart-btn').addEventListener('click', restartTest);
        }

        function restartTest() {
            resultScreen.classList.add('hidden');
            startScreen.classList.remove('hidden');
            startScreen.classList.add('fade-in');

            // Reset state
            currentQuestionIndex = 0;
            userScores = [];
            updateProgress();
        }

        // --- Event Listeners ---
        startBtn.addEventListener('click', () => {
            startScreen.classList.add('hidden');
            questionScreen.classList.remove('hidden');
            questionScreen.classList.add('fade-in');
            showQuestion(currentQuestionIndex);
        });

    </script>
</body>
</html>

