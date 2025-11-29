---
layout: default
title: أسئلة عشوائية
---

<div class="container">
	<header>
		<h1 class="arabic-text">أسئلة عشوائية</h1>
		<p class="section-subtitle arabic-text" style="color: rgba(255, 255, 255, 0.9); margin-top: 1rem;">
			تدرب على أسئلة عشوائية لتحسين مهاراتك في اللغة الإنجليزية
		</p>
	</header>

	<main id="questionContent">
		<section class="section" style="min-height: 200px;">
			<p style="text-align: center; color: #555; font-size: 1.2rem; padding: 2rem;">
				<span class="arabic-text">جاري التحميل...</span>
			</p>
		</section>
	</main>
</div>

<script>
	const QUESTION_BANK_PATH = "simple_question_bank/question_bank.json";
	let questionBank = null;
	let allQuestions = [];

	// Load question bank
	async function loadQuestionBank() {
		if (questionBank) {
			return questionBank;
		}

		try {
			const response = await fetch(QUESTION_BANK_PATH);
			if (!response.ok) {
				throw new Error(`Failed to load ${QUESTION_BANK_PATH}`);
			}
			questionBank = await response.json();
			
			// Flatten all questions into a single array
			allQuestions = [];
			Object.keys(questionBank).forEach(category => {
				questionBank[category].forEach(question => {
					allQuestions.push(question);
				});
			});

			console.log(`Loaded ${allQuestions.length} questions from question bank`);
			return questionBank;
		} catch (error) {
			console.error("Error loading question bank:", error);
			document.getElementById("questionContent").innerHTML = 
				'<section class="section"><p style="color: #ff6b6b; text-align: center;">Error loading questions. Please ensure the file exists.</p></section>';
			return null;
		}
	}

	// Get a random question
	function getRandomQuestion() {
		if (allQuestions.length === 0) {
			return null;
		}
		const randomIndex = Math.floor(Math.random() * allQuestions.length);
		return allQuestions[randomIndex];
	}

	// Display a random question
	function displayQuestion(question) {
		const content = document.getElementById("questionContent");
		const html = `
			<section class="section">
				<div class="question-item" style="text-align: center; padding: 3rem 2rem;">
					<button class="tts-button" onclick="speakText('${question.replace(/'/g, "\\'")}', this)" title="Read aloud" style="position: absolute; top: 1rem; right: 1rem;">🔊</button>
					<p class="question-text" style="font-size: clamp(1.5rem, 3vw, 2.5rem); margin: 0; color: #1e3c72; font-weight: 600;">
						${question}
					</p>
				</div>
			</section>
			<div style="text-align: center; margin-top: 2rem;">
				<button class="random-button" id="randomQuestionButton" onclick="loadRandomQuestion()">
					<span class="arabic-text">سؤال جديد</span>
				</button>
			</div>
		`;
		content.innerHTML = html;
	}

	// Load and display a random question
	async function loadRandomQuestion() {
		await loadQuestionBank();
		if (allQuestions.length === 0) {
			return;
		}
		const question = getRandomQuestion();
		if (question) {
			displayQuestion(question);
		}
	}

	// Make function globally accessible
	window.loadRandomQuestion = loadRandomQuestion;

	// Load initial question on page load
	(async function() {
		await loadQuestionBank();
		if (allQuestions.length > 0) {
			loadRandomQuestion();
		}
	})();

	// TTS functionality (reuse from main page if available, otherwise define it)
	if (typeof speakText === 'undefined') {
		let currentUtterance = null;
		let currentButton = null;

		window.speakText = function(text, buttonElement) {
			// Stop any currently playing speech
			if (window.speechSynthesis.speaking) {
				window.speechSynthesis.cancel();
				if (currentButton) {
					currentButton.classList.remove("playing");
				}
			}

			// Create new utterance
			const utterance = new SpeechSynthesisUtterance(text);
			utterance.lang = "en-US";
			utterance.rate = 0.9;
			utterance.pitch = 1;
			utterance.volume = 1;

			// Try to use a high-quality female voice if available
			function setVoice() {
				const voices = window.speechSynthesis.getVoices();
				const femaleVoiceNames = ["Samantha", "Victoria", "Karen", "Fiona", "Tessa", "Zira", "Hazel", "Susan", "Linda", "Female", "Google UK English Female", "Microsoft Zira"];
				const preferredFemaleVoices = voices.filter(
					(voice) => voice.lang.startsWith("en") && femaleVoiceNames.some((name) => voice.name.includes(name))
				);

				if (preferredFemaleVoices.length > 0) {
					utterance.voice = preferredFemaleVoices[0];
				} else {
					const englishVoices = voices.filter((voice) => voice.lang.startsWith("en"));
					if (englishVoices.length > 0) {
						utterance.voice = englishVoices[0];
					}
				}
			}

			setVoice();
			if (window.speechSynthesis.getVoices().length === 0) {
				window.speechSynthesis.onvoiceschanged = setVoice;
			}

			currentButton = buttonElement || (window.event ? window.event.target : null);

			if (currentButton) {
				currentButton.classList.add("playing");
			}

			utterance.onend = function() {
				if (currentButton) {
					currentButton.classList.remove("playing");
				}
				currentUtterance = null;
				currentButton = null;
			};

			utterance.onerror = function() {
				if (currentButton) {
					currentButton.classList.remove("playing");
				}
				currentUtterance = null;
				currentButton = null;
			};

			currentUtterance = utterance;
			window.speechSynthesis.speak(utterance);
		};
	}
</script>

