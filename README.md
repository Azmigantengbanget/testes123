<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Kuis Pengetahuan Elektronika Dasar & Lanjutan</title> <style>
 html {
box-sizing: border-box;
}
*, *::before, *::after {
box-sizing: inherit;
}
body {
font-family: Arial, sans-serif;
display: flex;
justify-content: center;
align-items: center;
min-height: 100vh;
background-color: #f0f0f0;
margin: 0;
padding: 20px;
}

    .quiz-container {
        background-color: white;
        padding: 30px;
        border-radius: 10px;
        box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
        width: 100%;
        max-width: 600px;
        text-align: center;
    }

    h1 {
        color: #333;
        margin-bottom: 10px;
    }

    #completion-message {
        color: #28a745;
        font-size: 1.2em;
        font-weight: bold;
        margin-top: 5px;
        margin-bottom: 20px;
    }

    .question-counter-text {
        font-size: 0.9em;
        color: #666;
        margin-bottom: 20px;
    }

    #question-container {
        margin-bottom: 20px;
    }

    #question {
        font-size: 1.5em;
        font-weight: bold;
        margin-bottom: 25px;
        color: #444;
    }

    .btn-grid {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        gap: 10px;
        margin-bottom: 20px;
    }

    .btn {
        background-color: #007bff;
        color: white;
        border: none;
        padding: 12px 15px;
        border-radius: 5px;
        cursor: pointer;
        font-size: 1em;
        transition: background-color 0.2s ease, box-shadow 0.2s ease;
        word-wrap: break-word;
        min-height: 50px;
        display: flex;
        align-items: center;
        justify-content: center;
        outline: none; 
        font-weight: bold;
    }

    .btn:not(.correct):not(.wrong):not(.skip-btn):not(.btn-prev-q) { background-color: #007bff; }
    .btn:not(.correct):not(.wrong):not(.skip-btn):not(.btn-prev-q):focus {
        box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.5);
    }
    .btn:not([disabled]):not(.correct):not(.wrong):not(.skip-btn):not(.btn-prev-q):hover {
        background-color: #0056b3; 
    }

    .btn.correct { background-color: #28a745 !important; box-shadow: none; }
    .btn.correct:hover:not([disabled]) { background-color: #218838 !important; }
    .btn.correct:focus {
        box-shadow: 0 0 0 3px rgba(40, 167, 69, 0.6) !important;
    }

    .btn.wrong { background-color: #dc3545 !important; box-shadow: none; }
    .btn.wrong:hover:not([disabled]) { background-color: #c82333 !important; }
    .btn.wrong:focus {
        box-shadow: 0 0 0 3px rgba(220, 53, 69, 0.6) !important;
    }

    .btn:disabled {
        cursor: not-allowed;
        opacity: 0.65;
    }
    .btn:disabled:not(.correct):not(.wrong):not(.skip-btn):not(.btn-prev-q) {
        background-color: #6c757d !important;
        color: #ccc !important;
    }

    .controls {
        display: flex;
        justify-content: center;
        gap: 10px;
    }

    #skip-navigation-controls {
        display: flex; 
        justify-content: space-between; 
        align-items: center;
        margin-top: 40px;
        margin-bottom: 10px;
    }

    .skip-btn { 
        background-color: #28a745; 
        color: white;
        padding: 8px 12px;
        font-size: 0.9em;
        min-width: 80px;
    }
    .skip-btn:hover:not([disabled]) {
        background-color: #218838; 
    }
    .skip-btn:disabled { 
        background-color: #a3d8b0 !important; 
        color: #e9f5ec !important;
    }

    .btn-prev-q { 
        background-color: #5F9EA0; 
        color: white;
        padding: 8px 12px;
        font-size: 0.9em;
        min-width: 80px;
    }
    .btn-prev-q:hover:not([disabled]) {
        background-color: #4682B4; 
    }
    .btn-prev-q:disabled {
        background-color: #B0C4DE !important; 
        color: #666666 !important;
    }

    .hide { display: none !important; }
</style>
</head>
<body>
<div class="quiz-container">
<h1>Pengetahuan Elektronika Dasar & Lanjutan</h1>
<p id="completion-message" class="hide">Selamat Kuis Sudah Selesai 🎉</p>
<div id="initial-controls" class="controls">
<button id="start-btn" class="btn">Mulai</button>
<button id="continue-btn" class="btn hide">Lanjutkan</button>
</div>
<div id="question-counter" class="question-counter-text hide">0/0</div>
<div id="question-container" class="hide">
<div id="question">Pertanyaan akan muncul di sini</div>
<div id="answer-buttons" class="btn-grid">
</div>
<div id="skip-navigation-controls" class="controls hide">
<button id="prev-50-btn" class="btn skip-btn">&laquo; 50</button>
<button id="prev-question-btn" class="btn btn-prev-q">&lt;</button>
<button id="next-50-btn" class="btn skip-btn">50 &raquo;</button>
</div>
</div>
</div>

<script>
    const startButton = document.getElementById('start-btn');
    const continueButton = document.getElementById('continue-btn');
    const initialControls = document.getElementById('initial-controls');
    const completionMessageElement = document.getElementById('completion-message');
    const questionContainerElement = document.getElementById('question-container');
    const questionElement = document.getElementById('question');
    const answerButtonsElement = document.getElementById('answer-buttons');
    const questionCounterElement = document.getElementById('question-counter');

    const skipNavigationControls = document.getElementById('skip-navigation-controls');
    const prev50Button = document.getElementById('prev-50-btn');
    const prevQuestionButton = document.getElementById('prev-question-btn');
    const next50Button = document.getElementById('next-50-btn');
    
    const JUMP_AMOUNT = 50;
    const QUESTION_TIMEOUT_MS = 10000;

    let orderedQuestions, currentQuestionIndex;
    let score = 0;
    let questionTimeout;

    // Daftar 200 pertanyaan Anda (sesuai dengan yang Anda berikan)
    const rawVocabularyList = [
        // Set 1: 100 Pertanyaan Elektronika Dasar
        { en: 'Apa itu elektron?', id: 'Elektron adalah partikel subatomik bermuatan negatif.' },
        { en: 'Apa itu arus listrik?', id: 'Arus listrik adalah aliran muatan listrik.' },
        { en: 'Apa itu tegangan?', id: 'Tegangan adalah perbedaan potensial listrik antara dua titik.' },
        { en: 'Apa itu resistansi?', id: 'Resistansi adalah hambatan terhadap aliran arus listrik.' },
        { en: 'Apa itu Hukum Ohm?', id: 'Hukum Ohm menyatakan hubungan antara tegangan, arus, dan resistansi (V=IR).' },
        { en: 'Apa itu konduktor?', id: 'Konduktor adalah bahan yang mudah menghantarkan arus listrik.' },
        { en: 'Apa itu insulator?', id: 'Insulator adalah bahan yang sulit menghantarkan arus listrik.' },
        { en: 'Apa itu semikonduktor?', id: 'Semikonduktor adalah bahan dengan konduktivitas antara konduktor dan insulator.' },
        { en: 'Apa itu resistor?', id: 'Resistor adalah komponen elektronik yang berfungsi untuk menghambat arus listrik.' },
        { en: 'Apa itu kapasitor?', id: 'Kapasitor adalah komponen elektronik yang dapat menyimpan muatan listrik.' },
        { en: 'Apa itu induktor?', id: 'Induktor adalah komponen elektronik yang dapat menyimpan energi dalam medan magnet.' },
        { en: 'Apa itu dioda?', id: 'Dioda adalah komponen semikonduktor yang hanya memperbolehkan arus mengalir dalam satu arah.' },
        { en: 'Apa itu transistor?', id: 'Transistor adalah komponen semikonduktor yang berfungsi sebagai penguat atau saklar elektronik.' },
        { en: 'Apa itu LED?', id: 'LED (Light Emitting Diode) adalah dioda yang memancarkan cahaya ketika dialiri arus listrik.' },
	{ en: 'Apa perbedaan utama kecepatan rambat antara gelombang cahaya dan gelombang suara di udara?', id: 'Gelombang cahaya (elektromagnetik).' }


    ];

    let questions = [];

    function shuffleArray(array) {
        for (let i = array.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [array[i], array[j]] = [array[j], array[i]];
        }
        return array;
    }

    rawVocabularyList.sort((a, b) => {
        const enA = a.en.toLowerCase();
        const enB = b.en.toLowerCase();
        if (enA < enB) return -1;
        if (enA > enB) return 1;
        return 0;
    });

    function generateQuestions() {
        const allIndonesianTranslations = rawVocabularyList.map(item => item.id);
        questions = [];
        rawVocabularyList.forEach(vocabItem => {
            const correctAnswer = vocabItem.id;
            let distractors = [];
            
            const potentialDistractorPool = allIndonesianTranslations.filter(id => id !== correctAnswer);
            shuffleArray(potentialDistractorPool);

            for (let i = 0; i < potentialDistractorPool.length && distractors.length < 3; i++) {
                if (!distractors.includes(potentialDistractorPool[i])) {
                    distractors.push(potentialDistractorPool[i]);
                }
            }

            const genericFallbacks = ["Jawaban A (contoh)", "Jawaban B (contoh)", "Jawaban C (contoh)", "Jawaban D (contoh)"];
            let fallbackIndex = 0;
            while (distractors.length < 3 && fallbackIndex < genericFallbacks.length) {
                const fb = genericFallbacks[fallbackIndex];
                if (fb !== correctAnswer && !distractors.includes(fb)) {
                    distractors.push(fb);
                }
                fallbackIndex++;
            }
            while (distractors.length < 3) { // Pengaman terakhir jika masih kurang
                distractors.push("Pilihan lainnya " + (distractors.length + Math.floor(Math.random() * 100)));
            }


            const answerOptions = [
                { text: correctAnswer, correct: true },
                { text: distractors[0], correct: false },
                { text: distractors[1], correct: false },
                { text: distractors[2], correct: false }
            ];
            questions.push({
                question: vocabItem.en,
                answers: answerOptions
            });
        });
    }

    generateQuestions();

    function saveProgress() {
        if (!questionContainerElement.classList.contains('hide') && orderedQuestions && currentQuestionIndex < orderedQuestions.length) {
            const progress = {
                currentQuestionIndex: currentQuestionIndex,
                score: score,
                orderedQuestions: orderedQuestions
            };
            localStorage.setItem('quizProgress', JSON.stringify(progress));
        }
    }

    function loadProgress() {
        const savedProgress = localStorage.getItem('quizProgress');
        if (savedProgress) {
            try {
                const progressData = JSON.parse(savedProgress);
                if (progressData && typeof progressData.currentQuestionIndex === 'number' &&
                    typeof progressData.score === 'number' && Array.isArray(progressData.orderedQuestions) &&
                    progressData.orderedQuestions.length > 0 &&
                    progressData.currentQuestionIndex < progressData.orderedQuestions.length &&
                    progressData.orderedQuestions.length === questions.length) { 
                    return progressData;
                } else {
                    console.warn("Saved progress data is invalid or mismatched. Clearing.");
                    clearProgress();
                    return null;
                }
            } catch (e) {
                console.error("Error parsing saved progress:", e);
                clearProgress();
                return null;
            }
        }
        return null;
    }

    function clearProgress() {
        localStorage.removeItem('quizProgress');
    }

    prev50Button.addEventListener('click', () => navigateQuestions(-JUMP_AMOUNT));
    prevQuestionButton.addEventListener('click', () => navigateQuestions(-1));
    next50Button.addEventListener('click', () => navigateQuestions(JUMP_AMOUNT));

    function navigateQuestions(amount) {
        clearTimeout(questionTimeout);
        if (!orderedQuestions || orderedQuestions.length === 0) return;

        let newIndex = currentQuestionIndex + amount;
        if (newIndex < 0) newIndex = 0;
        else if (newIndex >= orderedQuestions.length) newIndex = orderedQuestions.length - 1;

        if (newIndex !== currentQuestionIndex) {
            currentQuestionIndex = newIndex;
            setNextQuestion();
        } else {
            updateSkipButtonStates(); 
        }
    }

    function updateSkipButtonStates() {
        if (!orderedQuestions || orderedQuestions.length === 0 || questionContainerElement.classList.contains('hide')) {
            skipNavigationControls.classList.add('hide');
            if(prev50Button) prev50Button.disabled = true;
            if(prevQuestionButton) prevQuestionButton.disabled = true;
            if(next50Button) next50Button.disabled = true;
            return;
        }
        skipNavigationControls.classList.remove('hide');
        const isFirstQuestion = currentQuestionIndex === 0;
        const isLastQuestion = currentQuestionIndex === (orderedQuestions.length - 1);

        if(prev50Button) prev50Button.disabled = isFirstQuestion;
        if(prevQuestionButton) prevQuestionButton.disabled = isFirstQuestion;
        if(next50Button) next50Button.disabled = isLastQuestion;

        if (orderedQuestions.length <= 1) {
            if(prev50Button) prev50Button.disabled = true;
            if(prevQuestionButton) prevQuestionButton.disabled = true;
            if(next50Button) next50Button.disabled = true;
        }
    }

    window.addEventListener('load', () => {
        const savedData = loadProgress();
        startButton.innerText = 'Mulai'; 
        completionMessageElement.classList.add('hide');

        if (savedData) {
            continueButton.classList.remove('hide');
        } else {
            continueButton.classList.add('hide');
        }

        if (questionContainerElement.classList.contains('hide')) {
            initialControls.classList.remove('hide');
            skipNavigationControls.classList.add('hide');
        } else {
            initialControls.classList.add('hide');
            updateSkipButtonStates(); 
        }
    });

    startButton.addEventListener('click', () => startGame(false));
    continueButton.addEventListener('click', () => startGame(true));

    function startGame(isContinuing = false) {
        clearTimeout(questionTimeout);
        completionMessageElement.classList.add('hide');
        
        initialControls.classList.add('hide');
        questionContainerElement.classList.remove('hide');
        questionCounterElement.classList.remove('hide');

        const savedData = loadProgress();
        if (isContinuing && savedData) { 
            orderedQuestions = savedData.orderedQuestions; 
            currentQuestionIndex = savedData.currentQuestionIndex;
            score = savedData.score;
        } else {
            clearProgress(); 
            orderedQuestions = shuffleArray([...questions]); 
            currentQuestionIndex = 0;
            score = 0;
        }

        if (!orderedQuestions || orderedQuestions.length === 0) {
            showResults("Tidak ada soal untuk ditampilkan.", "#dc3545");
            return;
        }
        setNextQuestion();
    }

    function setNextQuestion() {
        resetState();
        if (orderedQuestions && currentQuestionIndex < orderedQuestions.length) {
            questionCounterElement.innerText = `${currentQuestionIndex + 1} / ${orderedQuestions.length}`;
            showQuestion(orderedQuestions[currentQuestionIndex]);
            saveProgress(); 
            if (document.activeElement && typeof document.activeElement.blur === 'function') {
                document.activeElement.blur(); 
            }
        } else {
            showResults(); 
        }
        updateSkipButtonStates(); 
    }

    function showQuestion(questionData) {
        questionElement.innerText = questionData.question;
        answerButtonsElement.innerHTML = ''; 
        const shuffledAnswers = shuffleArray([...questionData.answers]); 
        shuffledAnswers.forEach(answer => {
            const button = document.createElement('button');
            button.innerText = answer.text;
            button.classList.add('btn');
            if (answer.correct) {
                button.dataset.correct = answer.correct;
            }
            button.addEventListener('click', selectAnswer);
            answerButtonsElement.appendChild(button);
        });
    }

    function resetState() {
        clearTimeout(questionTimeout);
        Array.from(answerButtonsElement.children).forEach(button => {
            button.disabled = false; 
            clearStatusClass(button);
        });
    }

    function selectAnswer(e) {
        const selectedButton = e.target;
        const correct = selectedButton.dataset.correct === 'true';
        
        if (correct) { 
            score++; 
        }

        Array.from(answerButtonsElement.children).forEach(button => {
            setStatusClass(button, button.dataset.correct === 'true');
            button.disabled = true; 
        });
       
        saveProgress(); 

        questionTimeout = setTimeout(() => {
            if (orderedQuestions && currentQuestionIndex < orderedQuestions.length - 1) {
                currentQuestionIndex++;
                setNextQuestion();
            } else { 
                showResults();
            }
        }, QUESTION_TIMEOUT_MS);
    }

    function setStatusClass(element, correct) {
        clearStatusClass(element);
        if (correct) { 
            element.classList.add('correct'); 
        } else { 
            element.classList.add('wrong'); 
        }
    }

    function clearStatusClass(element) {
        element.classList.remove('correct');
        element.classList.remove('wrong');
    }

    function showResults(message = "Selamat Kuis Sudah Selesai 🎉", color = "#28a745") {
        clearTimeout(questionTimeout);
        questionContainerElement.classList.add('hide');
        questionCounterElement.classList.add('hide');
        skipNavigationControls.classList.add('hide');
        
        completionMessageElement.innerText = message;
        completionMessageElement.style.color = color;
        completionMessageElement.classList.remove('hide');
        
        startButton.innerText = 'Ulangi Kuis';
        initialControls.classList.remove('hide');
        continueButton.classList.add('hide'); 
        
        clearProgress(); 
    }
</script>
</body>
</html>
