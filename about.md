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
			
			// Use the flattened questions array directly
			allQuestions = questionBank.questions || [];

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

	// Toggle Arabic translation display
	function toggleArabicTranslation() {
		const arabicDiv = document.getElementById('question-arabic');
		const infoButton = document.getElementById('info-button');
		if (arabicDiv && infoButton) {
			if (arabicDiv.style.maxHeight && arabicDiv.style.maxHeight !== '0px') {
				// Collapse
				arabicDiv.style.maxHeight = '0px';
				arabicDiv.style.opacity = '0';
				infoButton.classList.remove('active');
			} else {
				// Expand - temporarily set to auto to get the actual height
				arabicDiv.style.maxHeight = 'none';
				const height = arabicDiv.scrollHeight;
				arabicDiv.style.maxHeight = '0px';
				// Force reflow
				void arabicDiv.offsetHeight;
				// Now animate to the actual height
				arabicDiv.style.maxHeight = height + 'px';
				arabicDiv.style.opacity = '1';
				infoButton.classList.add('active');
			}
		}
	}

	// Display a random question
	function displayQuestion(question) {
		const content = document.getElementById("questionContent");
		const html = `
			<section class="section">
				<div class="question-item" style="text-align: center; padding: 3rem 2rem; position: relative;">
					<button class="tts-button" onclick="speakText('${question.english.replace(/'/g, "\\'")}', this)" title="Read aloud" style="position: absolute; top: 1rem; right: 1rem;">🔊</button>
					<button id="info-button" class="info-button" onclick="toggleArabicTranslation()" title="Show Arabic translation" style="position: absolute; top: 1rem; left: 1rem;">ℹ️</button>
					<p class="question-text" style="font-size: clamp(1.5rem, 3vw, 2.5rem); margin: 0; color: #1e3c72; font-weight: 600;">
						${question.english}
					</p>
					<div id="question-arabic" class="arabic-translation" style="max-height: 0; opacity: 0; overflow: hidden; transition: max-height 0.3s ease, opacity 0.3s ease; margin-top: 1.5rem;">
						<p class="arabic-text" style="font-size: clamp(1.2rem, 2.5vw, 1.8rem); margin: 0; color: #2a5298; font-weight: 600; direction: rtl; text-align: center;">
							${question.arabic.replace(/'/g, "\\'")}
						</p>
					</div>
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

	// Make function globally accessible
	window.toggleArabicTranslation = toggleArabicTranslation;

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

<style>
	.info-button {
		background: rgba(74, 144, 226, 0.2);
		border: 2px solid #4a90e2;
		border-radius: 50%;
		width: 2.5rem;
		height: 2.5rem;
		display: flex;
		align-items: center;
		justify-content: center;
		cursor: pointer;
		font-size: 1.2rem;
		transition: all 0.3s ease;
		padding: 0;
	}

	.info-button:hover {
		background: rgba(74, 144, 226, 0.3);
		transform: scale(1.1);
		box-shadow: 0 4px 12px rgba(74, 144, 226, 0.3);
	}

	.info-button.active {
		background: rgba(74, 144, 226, 0.4);
		border-color: #357abd;
	}

	.arabic-translation {
		border-top: 2px solid rgba(74, 144, 226, 0.2);
		padding-top: 1.5rem;
	}

	@media (max-width: 768px) {
		.info-button {
			width: 2rem;
			height: 2rem;
			font-size: 1rem;
		}
	}
</style>

