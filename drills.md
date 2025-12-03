---
layout: default
title: تمارين الجمل
---

<div class="container">
	<header>
		<h1 class="arabic-text">تمارين الجمل</h1>
		<p class="section-subtitle arabic-text" style="color: #333333; margin-top: 1rem;">
			رتب الكلمات لبناء جملة صحيحة
		</p>
	</header>

	<main id="drillContent">
		<div style="text-align: center; color: #333333; font-size: 1.2rem; padding: 2rem;">
			<span class="arabic-text">جاري التحميل...</span>
		</div>
	</main>
</div>

<script>
	const DRILL_PATH = "drills/jumbled_drill.json";
	let drillData = null;
	let allItems = [];
	let currentItem = null;
	let selectedWords = [];
	let availableWords = [];

	// Load drill data
	async function loadDrillData() {
		if (drillData) {
			return drillData;
		}

		try {
			const response = await fetch(DRILL_PATH);
			if (!response.ok) {
				throw new Error(`Failed to load ${DRILL_PATH}`);
			}
			drillData = await response.json();
			allItems = drillData.items || [];

			console.log(`Loaded ${allItems.length} drill items`);
			return drillData;
		} catch (error) {
			console.error("Error loading drill data:", error);
			document.getElementById("drillContent").innerHTML =
				'<section class="section"><p style="color: #ff6b6b; text-align: center;">Error loading drill data. Please ensure the file exists.</p></section>';
			return null;
		}
	}

	// Get a random item
	function getRandomItem() {
		if (allItems.length === 0) {
			return null;
		}
		const randomIndex = Math.floor(Math.random() * allItems.length);
		return allItems[randomIndex];
	}

	// Handle word selection from available words
	function selectWord(displayIndex) {
		// Get all non-null words and their original indices
		const availableWordsWithIndices = availableWords
			.map((word, originalIndex) => ({ word, originalIndex }))
			.filter(item => item.word !== null);

		if (displayIndex >= availableWordsWithIndices.length) {
			return;
		}

		const { word, originalIndex } = availableWordsWithIndices[displayIndex];
		selectedWords.push(word);
		availableWords[originalIndex] = null; // Mark as used

		updateDisplay();
		checkAnswer();
	}

	// Handle word removal from selected sentence
	function removeWord(index) {
		const word = selectedWords[index];
		selectedWords.splice(index, 1);

		// Find the first null position and restore the word
		for (let i = 0; i < availableWords.length; i++) {
			if (availableWords[i] === null) {
				availableWords[i] = word;
				break;
			}
		}

		updateDisplay();
		checkAnswer();
	}

	// Normalize sentence for comparison (remove punctuation, trim, lowercase)
	function normalizeSentence(sentence) {
		return sentence
			.replace(/[.,!?;:]/g, '') // Remove punctuation
			.trim()
			.toLowerCase()
			.replace(/\s+/g, ' '); // Normalize whitespace
	}

	// Check if the answer is correct
	function checkAnswer() {
		if (!currentItem) return;

		const constructedSentence = selectedWords.join(" ").trim();
		const correctSentence = currentItem.correct.trim();
		const totalWords = currentItem.jumbled.length;

		// Only check if all words are selected
		if (selectedWords.length === totalWords) {
			// Normalize both sentences for comparison (ignore punctuation and case)
			const normalizedConstructed = normalizeSentence(constructedSentence);
			const normalizedCorrect = normalizeSentence(correctSentence);

			if (normalizedConstructed === normalizedCorrect) {
				// Correct!
				setTimeout(() => {
					showFeedback(true);
				}, 100);
			} else {
				// All words selected but incorrect
				setTimeout(() => {
					showFeedback(false);
				}, 100);
			}
		}
	}

	// Show feedback
	function showFeedback(isCorrect) {
		const feedbackDiv = document.getElementById("feedback");
		if (feedbackDiv) {
			if (isCorrect) {
				feedbackDiv.innerHTML = '<p style="color: #4CAF50; font-size: 1.2rem; font-weight: 600; text-align: center; margin: 1rem 0;">✓ صحيح! ممتاز!</p>';
				feedbackDiv.style.visibility = "visible";
				feedbackDiv.style.opacity = "1";
			} else {
				feedbackDiv.innerHTML = '<p style="color: #ff6b6b; font-size: 1.2rem; font-weight: 600; text-align: center; margin: 1rem 0;">✗ غير صحيح. حاول مرة أخرى</p>';
				feedbackDiv.style.visibility = "visible";
				feedbackDiv.style.opacity = "1";
			}
		}
	}

	// Reset current item
	function resetItem() {
		if (!currentItem) return;
		
		selectedWords = [];
		availableWords = [...currentItem.jumbled];

		const feedbackDiv = document.getElementById("feedback");
		if (feedbackDiv) {
			feedbackDiv.style.visibility = "hidden";
			feedbackDiv.style.opacity = "0";
			feedbackDiv.innerHTML = "";
		}

		updateDisplay();
	}

	// Update the display
	function updateDisplay() {
		const content = document.getElementById("drillContent");
		
		const html = `
			<section class="section" style="max-width: 1200px; margin: 0 auto;">
				<!-- Available Words -->
				<div style="margin-bottom: 2rem;">
					<h3 class="arabic-text" style="color: #1e3c72; font-size: 1.3rem; margin-bottom: 1rem; text-align: center;">
						الكلمات المتاحة
					</h3>
					<div id="availableWords" class="words-container" style="display: flex; flex-wrap: wrap; gap: 0.75rem; justify-content: center; min-height: 80px; align-content: flex-start;">
						${availableWords
							.map((word, originalIndex) => ({ word, originalIndex }))
							.filter(item => item.word !== null)
							.map((item, displayIndex) => `
								<button 
									class="word-card available-word" 
									onclick="selectWord(${displayIndex})"
									style="
										background: rgba(74, 144, 226, 0.2);
										border: 2px solid #4a90e2;
										padding: 0.75rem 1.25rem;
										border-radius: 0.5rem;
										cursor: pointer;
										font-size: 1.1rem;
										font-weight: 600;
										color: #1e3c72;
										transition: all 0.2s;
										min-width: 80px;
									"
								>
									${item.word}
								</button>
							`).join("")}
					</div>
				</div>

				<!-- Constructed Sentence -->
				<div style="margin-bottom: 2rem;">
					<h3 class="arabic-text" style="color: #1e3c72; font-size: 1.3rem; margin-bottom: 1rem; text-align: center;">
						الجملة المكونة
					</h3>
					<div id="constructedSentence" class="sentence-container" style="
						background: rgba(255, 255, 255, 0.95);
						padding: 2rem;
						border-radius: 0.75rem;
						min-height: 180px;
						display: flex;
						flex-wrap: wrap;
						gap: 0.5rem;
						align-items: flex-start;
						align-content: flex-start;
						justify-content: center;
						border: 2px solid #4a90e2;
					">
						${selectedWords.length === 0 ? 
							'<p style="color: #999; font-style: italic; margin: 0;">انقر على الكلمات لبناء الجملة</p>' :
							selectedWords.map((word, index) => `
								<button 
									class="word-card selected-word" 
									onclick="removeWord(${index})"
									style="
										background: rgba(126, 34, 206, 0.2);
										border: 2px solid #7e22ce;
										padding: 0.75rem 1.25rem;
										border-radius: 0.5rem;
										cursor: pointer;
										font-size: 1.1rem;
										font-weight: 600;
										color: #1e3c72;
										transition: all 0.2s;
									"
								>
									${word}
								</button>
							`).join("")
						}
					</div>
				</div>

				<!-- Feedback -->
				<div id="feedback" style="
					visibility: hidden;
					opacity: 0;
					min-height: 60px;
					transition: opacity 0.3s ease;
					margin: 1rem 0;
				"></div>

				<!-- Action Buttons -->
				<div style="text-align: center; margin-top: 2rem; display: flex; gap: 1rem; justify-content: center; flex-wrap: wrap;">
					<button 
						class="reset-button" 
						onclick="resetItem()"
						style="
							padding: 0.75rem 2rem;
							background: #666;
							color: white;
							border: none;
							border-radius: 0.5rem;
							font-size: 1rem;
							font-weight: 600;
							cursor: pointer;
							transition: all 0.2s;
						"
					>
						<span class="arabic-text">إعادة</span>
					</button>
					<button 
						class="next-button" 
						onclick="loadNextItem()"
						style="
							padding: 0.75rem 2rem;
							background: #4a90e2;
							color: white;
							border: none;
							border-radius: 0.5rem;
							font-size: 1rem;
							font-weight: 600;
							cursor: pointer;
							transition: all 0.2s;
						"
					>
						<span class="arabic-text">التالي</span>
					</button>
				</div>
			</section>
		`;

		content.innerHTML = html;
	}

	// Load and display a random item
	async function loadNextItem() {
		await loadDrillData();
		if (allItems.length === 0) {
			return;
		}

		currentItem = getRandomItem();
		if (!currentItem) {
			return;
		}

		// Reset state
		selectedWords = [];
		availableWords = [...currentItem.jumbled]; // Copy the jumbled array

		// Hide feedback
		const feedbackDiv = document.getElementById("feedback");
		if (feedbackDiv) {
			feedbackDiv.style.visibility = "hidden";
			feedbackDiv.style.opacity = "0";
			feedbackDiv.innerHTML = "";
		}

		updateDisplay();
	}

	// Make functions globally accessible
	window.selectWord = selectWord;
	window.removeWord = removeWord;
	window.loadNextItem = loadNextItem;
	window.resetItem = resetItem;

	// Load initial item on page load
	(async function() {
		await loadDrillData();
		if (allItems.length > 0) {
			loadNextItem();
		}
	})();
</script>

<style>
	.words-container {
		position: relative;
	}

	.word-card {
		height: auto;
		box-sizing: border-box;
	}

	.available-word:hover {
		background: rgba(74, 144, 226, 0.3) !important;
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(74, 144, 226, 0.3);
	}

	.selected-word:hover {
		background: rgba(126, 34, 206, 0.3) !important;
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(126, 34, 206, 0.3);
	}

	.next-button:hover {
		background: #357abd !important;
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(74, 144, 226, 0.3);
	}

	.next-button:active {
		transform: translateY(0);
	}

	.reset-button:hover {
		background: #555 !important;
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
	}

	.reset-button:active {
		transform: translateY(0);
	}

	#feedback {
		display: block;
	}

	.sentence-container {
		transition: min-height 0.2s ease;
		overflow: visible;
	}

	/* Ensure smooth expansion without bouncing */
	.sentence-container > * {
		flex-shrink: 0;
	}

	/* Smooth word card animations */
	.word-card {
		transition: transform 0.2s ease, opacity 0.15s ease;
		animation: fadeIn 0.2s ease;
	}

	@keyframes fadeIn {
		from {
			opacity: 0;
			transform: scale(0.9);
		}
		to {
			opacity: 1;
			transform: scale(1);
		}
	}

	@media (max-width: 768px) {
		.words-container {
			gap: 0.5rem !important;
		}

		.word-card {
			font-size: 0.9rem !important;
			padding: 0.5rem 1rem !important;
		}

		.sentence-container {
			padding: 1.5rem !important;
			min-height: 150px !important;
		}
	}
</style>

