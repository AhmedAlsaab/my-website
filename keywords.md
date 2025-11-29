---
layout: default
title: كلمات مهمة
---

<div class="container">
	<header>
		<h1 class="arabic-text">كلمات مهمة</h1>
		<p class="section-subtitle arabic-text" style="color: rgba(255, 255, 255, 0.9); margin-top: 1rem;">
			تعلم الكلمات الأساسية باللغة الإنجليزية مع ترجمتها العربية
		</p>
	</header>

	<main id="keywordsContent">
		<p style="text-align: center; color: rgba(255, 255, 255, 0.9); font-size: 1.2rem; padding: 2rem;">
			<span class="arabic-text">جاري التحميل...</span>
		</p>
	</main>
</div>

<script>
	const KEYWORDS_PATH = "keywords_in_arabic/keywords.json";
	let keywordsData = null;
	let keywordsByCategory = {};

	// Category names in Arabic
	const categoryNames = {
		"people": "الأشخاص",
		"school": "المدرسة",
		"home": "المنزل",
		"places": "الأماكن",
		"transport": "المواصلات",
		"time": "الوقت",
		"weather": "الطقس",
		"food_drink": "الطعام والشراب",
		"actions": "الأفعال",
		"feelings": "المشاعر",
		"other": "أخرى"
	};

	// Load keywords data
	async function loadKeywords() {
		if (keywordsData) {
			return keywordsData;
		}

		try {
			const response = await fetch(KEYWORDS_PATH);
			if (!response.ok) {
				throw new Error(`Failed to load ${KEYWORDS_PATH}`);
			}
			keywordsData = await response.json();

			// Group keywords by category
			keywordsByCategory = {};
			keywordsData.keywords.forEach(keyword => {
				if (!keywordsByCategory[keyword.category]) {
					keywordsByCategory[keyword.category] = [];
				}
				keywordsByCategory[keyword.category].push(keyword);
			});

			console.log(`Loaded keywords from ${Object.keys(keywordsByCategory).length} categories`);
			return keywordsData;
		} catch (error) {
			console.error("Error loading keywords:", error);
			document.getElementById("keywordsContent").innerHTML =
				'<section class="section"><p style="color: #ff6b6b; text-align: center;">Error loading keywords. Please ensure the file exists.</p></section>';
			return null;
		}
	}

	// Get a random keyword from a category
	function getRandomKeyword(category) {
		const categoryKeywords = keywordsByCategory[category];
		if (!categoryKeywords || categoryKeywords.length === 0) {
			return null;
		}
		const randomIndex = Math.floor(Math.random() * categoryKeywords.length);
		return categoryKeywords[randomIndex];
	}

	// Display all categories with random keywords
	function displayKeywords() {
		const content = document.getElementById("keywordsContent");
		let html = "";

		// Sort categories for consistent display
		const sortedCategories = Object.keys(keywordsByCategory).sort();

		sortedCategories.forEach(category => {
			const randomKeyword = getRandomKeyword(category);
			if (!randomKeyword) return;

			const categoryNameAr = categoryNames[category] || category;
			
			html += `
				<section class="section" style="margin-bottom: 2rem;">
					<h2 class="arabic-text" style="color: #1e3c72; font-size: clamp(1.5rem, 3vw, 2rem); margin-bottom: 1.5rem; border-bottom: 2px solid #4a90e2; padding-bottom: 0.5rem;">
						${categoryNameAr}
					</h2>
					<div class="keyword-display" data-category="${category}" style="background: rgba(74, 144, 226, 0.08); padding: 2rem; border-radius: 0.5rem; border-left: 4px solid #4a90e2; position: relative; min-height: 120px;">
						<button class="tts-button" onclick="speakText('${randomKeyword.english.replace(/'/g, "\\'")}', this)" title="Read aloud" style="position: absolute; top: 1rem; right: 1rem;">🔊</button>
						<div style="padding-right: 3.5rem;">
							<div style="display: flex; align-items: center; gap: 1.5rem; margin-bottom: 1rem; flex-wrap: wrap;">
								<div style="flex: 1; min-width: 200px;">
									<p style="color: #666; font-size: 0.9rem; margin: 0; margin-bottom: 0.25rem; font-weight: 600;">English</p>
									<p class="keyword-english" style="font-size: clamp(1.5rem, 3vw, 2rem); margin: 0; color: #1e3c72; font-weight: 700;">
										${randomKeyword.english}
									</p>
								</div>
								<div style="flex: 1; min-width: 200px;">
									<p class="arabic-text" style="color: #666; font-size: 0.9rem; margin: 0; margin-bottom: 0.25rem; font-weight: 600; direction: rtl; text-align: right;">العربية</p>
									<p class="keyword-arabic arabic-text" style="font-size: clamp(1.5rem, 3vw, 2rem); margin: 0; color: #1e3c72; font-weight: 700; direction: rtl; text-align: right;">
										${randomKeyword.arabic}
									</p>
								</div>
							</div>
						</div>
						<button class="random-keyword-button" onclick="generateNewKeyword('${category}')" style="margin-top: 1rem; padding: 0.5rem 1.5rem; background: #4a90e2; color: white; border: none; border-radius: 0.5rem; cursor: pointer; font-weight: 600; font-size: 1rem; transition: all 0.2s;">
							<span class="arabic-text">كلمة جديدة</span>
						</button>
					</div>
				</section>
			`;
		});

		content.innerHTML = html;
	}

	// Generate a new random keyword for a specific category
	function generateNewKeyword(category) {
		const randomKeyword = getRandomKeyword(category);
		if (!randomKeyword) return;

		const keywordDisplay = document.querySelector(`.keyword-display[data-category="${category}"]`);
		if (!keywordDisplay) return;

		// Update the English text
		const englishElement = keywordDisplay.querySelector('.keyword-english');
		if (englishElement) {
			englishElement.textContent = randomKeyword.english;
		}

		// Update the Arabic text
		const arabicElement = keywordDisplay.querySelector('.keyword-arabic');
		if (arabicElement) {
			arabicElement.textContent = randomKeyword.arabic;
		}

		// Update the TTS button onclick
		const ttsButton = keywordDisplay.querySelector('.tts-button');
		if (ttsButton) {
			ttsButton.setAttribute('onclick', `speakText('${randomKeyword.english.replace(/'/g, "\\'")}', this)`);
		}
	}

	// Make function globally accessible
	window.generateNewKeyword = generateNewKeyword;

	// Load and display keywords on page load
	(async function() {
		await loadKeywords();
		if (keywordsData && Object.keys(keywordsByCategory).length > 0) {
			displayKeywords();
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
	.random-keyword-button:hover {
		background: #357abd !important;
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(74, 144, 226, 0.3);
	}

	.random-keyword-button:active {
		transform: translateY(0);
	}

	@media (max-width: 768px) {
		.keyword-display {
			padding: 1.5rem !important;
		}
	}
</style>

