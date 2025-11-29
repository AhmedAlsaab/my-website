---
layout: default
title: ربط الجمل
---

<div class="container">
	<header>
		<h1 class="arabic-text">ربط الجمل</h1>
		<p class="section-subtitle arabic-text" style="color: rgba(255, 255, 255, 0.9); margin-top: 1rem;">
			اربط الجمل اليسرى بالجمل اليمنى الصحيحة
		</p>
	</header>

	<main id="crosswordContent">
		<div style="text-align: center; color: rgba(255, 255, 255, 0.9); font-size: 1.2rem; padding: 2rem;">
			<span class="arabic-text">جاري التحميل...</span>
		</div>
	</main>
</div>

<script>
	const CROSSWORD_PATH = "crossword/sentence_matching.json";
	let gameData = null;
	let shuffledStarters = [];
	let shuffledFinishers = [];
	let matchedPairs = [];
	let selectedStarter = null;
	let selectedFinisher = null;

	// Load game data
	async function loadGameData() {
		if (gameData) {
			return gameData;
		}

		try {
			const response = await fetch(CROSSWORD_PATH);
			if (!response.ok) {
				throw new Error(`Failed to load ${CROSSWORD_PATH}`);
			}
			gameData = await response.json();

			// Shuffle starters and finishers separately
			shuffledStarters = shuffleArray([...gameData.starters]);
			shuffledFinishers = shuffleArray([...gameData.finishers]);

			console.log(`Loaded ${shuffledStarters.length} starters and ${shuffledFinishers.length} finishers`);
			return gameData;
		} catch (error) {
			console.error("Error loading game data:", error);
			document.getElementById("crosswordContent").innerHTML =
				'<section class="section"><p style="color: #ff6b6b; text-align: center;">Error loading game data. Please ensure the file exists.</p></section>';
			return null;
		}
	}

	// Shuffle array function
	function shuffleArray(array) {
		const shuffled = [...array];
		for (let i = shuffled.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
		}
		return shuffled;
	}

	// Check if a starter-finisher pair is correct
	function isCorrectMatch(starter, finisher) {
		return gameData.correct_matches.some(
			match => match.starter === starter && match.finisher === finisher
		);
	}

	// Check if a starter has any remaining possible matches
	function hasStarterRemainingMatches(starter) {
		const matchedFinishers = matchedPairs.map(pair => pair.finisher);
		return gameData.correct_matches.some(
			match => match.starter === starter && !matchedFinishers.includes(match.finisher)
		);
	}

	// Get total possible matches for a finisher
	function getTotalPossibleMatchesForFinisher(finisher) {
		return gameData.correct_matches.filter(match => match.finisher === finisher).length;
	}

	// Get current match count for a finisher
	function getFinisherMatchCount(finisher) {
		return matchedPairs.filter(pair => pair.finisher === finisher).length;
	}

	// Check if a finisher has all its matches completed
	function isFinisherFullyMatched(finisher) {
		const totalMatches = getTotalPossibleMatchesForFinisher(finisher);
		const currentMatches = getFinisherMatchCount(finisher);
		return totalMatches > 0 && currentMatches >= totalMatches;
	}

	// Check if a finisher has any remaining matches
	function hasFinisherRemainingMatches(finisher) {
		const totalMatches = getTotalPossibleMatchesForFinisher(finisher);
		const currentMatches = getFinisherMatchCount(finisher);
		return currentMatches < totalMatches;
	}

	// Handle starter card click
	function handleStarterClick(index) {
		const starter = shuffledStarters[index];

		// Check if starter has any remaining possible matches
		if (!hasStarterRemainingMatches(starter)) {
			return; // No more matches possible for this starter
		}

		// If this starter is already selected, deselect it
		if (selectedStarter === index) {
			selectedStarter = null;
			displayGame();
			return;
		}

		// Select this starter
		selectedStarter = index;

		// If finisher is also selected, check match
		if (selectedFinisher !== null) {
			checkMatch();
		} else {
			displayGame();
		}
	}

	// Handle finisher card click
	function handleFinisherClick(index) {
		const finisher = shuffledFinishers[index];

		// If all matches are complete, do nothing
		if (isFinisherFullyMatched(finisher)) {
			return;
		}

		// If this finisher is already selected, deselect it
		if (selectedFinisher === index) {
			selectedFinisher = null;
			displayGame();
			return;
		}

		// Select this finisher
		selectedFinisher = index;

		// If starter is also selected, check match
		if (selectedStarter !== null) {
			checkMatch();
		} else {
			displayGame();
		}
	}

	// Check if selected cards match
	function checkMatch() {
		const starter = shuffledStarters[selectedStarter];
		const finisher = shuffledFinishers[selectedFinisher];

		if (isCorrectMatch(starter, finisher)) {
			// Correct match!
			matchedPairs.push({ starter: starter, finisher: finisher });

			// Clear selections
			selectedStarter = null;
			selectedFinisher = null;

			// Update display
			setTimeout(() => {
				displayGame();
			}, 300);
		} else {
			// Wrong match - shake and deselect
			const starterElement = document.querySelector(`.starter-card[data-index="${selectedStarter}"]`);
			const finisherElement = document.querySelector(`.finisher-card[data-index="${selectedFinisher}"]`);

			if (starterElement) starterElement.style.animation = 'shake 0.5s';
			if (finisherElement) finisherElement.style.animation = 'shake 0.5s';

			setTimeout(() => {
				selectedStarter = null;
				selectedFinisher = null;
				displayGame();
			}, 500);
		}
	}

	// Display the game
	function displayGame() {
		const content = document.getElementById("crosswordContent");
		
		const html = `
			<section class="section" style="max-width: 1400px; margin: 0 auto;">
				<!-- Starters Section (Top) -->
				<div style="margin-bottom: 3rem;">
					<h2 class="arabic-text" style="color: #1e3c72; font-size: 1.5rem; margin-bottom: 1.5rem; text-align: center; direction: rtl;">
						بدايات الجمل
					</h2>
					<div id="startersContainer" class="cards-container" style="display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 1rem; min-height: auto;">
							${shuffledStarters.map((starter, index) => {
								const hasRemaining = hasStarterRemainingMatches(starter);
								const isSelected = selectedStarter === index;
								const matchedCount = matchedPairs.filter(pair => pair.starter === starter).length;
								return `
									<div 
										class="card starter-card ${!hasRemaining ? 'no-matches' : ''} ${isSelected ? 'selected' : ''}" 
										data-index="${index}"
										onclick="${hasRemaining ? `handleStarterClick(${index})` : ''}"
										style="
											background: ${!hasRemaining ? 'rgba(200, 200, 200, 0.3)' : isSelected ? 'rgba(74, 144, 226, 0.3)' : 'rgba(255, 255, 255, 0.95)'};
											border: 2px solid ${!hasRemaining ? '#999' : isSelected ? '#4a90e2' : '#4a90e2'};
											border-width: ${isSelected ? '3px' : '2px'};
											padding: 1.25rem;
											border-radius: 0.75rem;
											cursor: ${hasRemaining ? 'pointer' : 'default'};
											transition: all 0.3s ease;
											box-shadow: ${isSelected ? '0 6px 20px rgba(74, 144, 226, 0.4)' : '0 4px 12px rgba(0, 0, 0, 0.1)'};
											opacity: ${!hasRemaining ? '0.5' : '1'};
											transform: ${isSelected ? 'scale(1.02)' : 'scale(1)'};
											position: relative;
										"
									>
										<p style="margin: 0; color: #1e3c72; font-size: 1.1rem; font-weight: 600; text-align: center;">
											${starter}
										</p>
										${matchedCount > 0 ? `<span style="position: absolute; top: 0.5rem; right: 0.5rem; background: #4CAF50; color: white; border-radius: 50%; width: 1.5rem; height: 1.5rem; display: flex; align-items: center; justify-content: center; font-size: 0.75rem; font-weight: 700;">${matchedCount}</span>` : ''}
									</div>
								`;
							}).join("")}
					</div>
				</div>

				<!-- Finishers Section (Bottom) -->
				<div style="margin-top: 3rem;">
					<h2 class="arabic-text" style="color: #1e3c72; font-size: 1.5rem; margin-bottom: 1.5rem; text-align: center; direction: rtl;">
						نهايات الجمل
					</h2>
					<div id="finishersContainer" class="cards-container" style="display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 1rem; min-height: auto;">
							${shuffledFinishers.map((finisher, index) => {
								const isFullyMatched = isFinisherFullyMatched(finisher);
								const isSelected = selectedFinisher === index;
								const currentMatches = getFinisherMatchCount(finisher);
								const totalMatches = getTotalPossibleMatchesForFinisher(finisher);
								const hasRemaining = hasFinisherRemainingMatches(finisher);
								return `
									<div 
										class="card finisher-card ${isFullyMatched ? 'matched' : ''} ${isSelected ? 'selected' : ''}" 
										data-index="${index}"
										onclick="${hasRemaining ? `handleFinisherClick(${index})` : ''}"
										style="
											background: ${isFullyMatched ? 'rgba(76, 175, 80, 0.2)' : isSelected ? 'rgba(126, 34, 206, 0.3)' : 'rgba(255, 255, 255, 0.95)'};
											border: 2px solid ${isFullyMatched ? '#4CAF50' : isSelected ? '#7e22ce' : '#7e22ce'};
											border-width: ${isSelected ? '3px' : '2px'};
											padding: 1.25rem;
											border-radius: 0.75rem;
											cursor: ${hasRemaining ? 'pointer' : 'default'};
											transition: all 0.3s ease;
											box-shadow: ${isSelected ? '0 6px 20px rgba(126, 34, 206, 0.4)' : '0 4px 12px rgba(0, 0, 0, 0.1)'};
											opacity: ${isFullyMatched ? '0.6' : '1'};
											transform: ${isSelected ? 'scale(1.02)' : 'scale(1)'};
											position: relative;
										"
									>
										<p style="margin: 0; color: #1e3c72; font-size: 1.1rem; font-weight: 600; text-align: center;">
											${finisher}
										</p>
										${totalMatches > 0 ? `
											<div style="position: absolute; top: 0.5rem; right: 0.5rem; background: ${isFullyMatched ? '#4CAF50' : '#7e22ce'}; color: white; border-radius: 0.5rem; padding: 0.25rem 0.5rem; font-size: 0.75rem; font-weight: 700;">
												${currentMatches} / ${totalMatches}
											</div>
										` : ''}
									</div>
								`;
							}).join("")}
					</div>
				</div>

				<div style="text-align: center; margin-top: 2rem;">
					<button 
						class="reset-button" 
						onclick="resetGame()"
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
						<span class="arabic-text">إعادة اللعب</span>
					</button>
					<p style="margin-top: 1rem; color: #666; font-size: 0.9rem;">
						<span class="arabic-text">النتيجة:</span> ${matchedPairs.length} / ${gameData.correct_matches.length}
					</p>
				</div>
			</section>
		`;

		content.innerHTML = html;
	}

	// Reset game
	function resetGame() {
		matchedPairs = [];
		selectedStarter = null;
		selectedFinisher = null;
		shuffledStarters = shuffleArray([...gameData.starters]);
		shuffledFinishers = shuffleArray([...gameData.finishers]);
		displayGame();
	}

	// Make functions globally accessible
	window.handleStarterClick = handleStarterClick;
	window.handleFinisherClick = handleFinisherClick;
	window.resetGame = resetGame;

	// Load and display game on page load
	(async function() {
		await loadGameData();
		if (gameData && shuffledStarters.length > 0) {
			displayGame();
		}
	})();
</script>

<style>
	.starter-card:hover:not(.matched) {
		transform: translateY(-2px) scale(1.01);
		box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
	}

	.finisher-card:hover:not(.matched) {
		transform: translateY(-2px) scale(1.01);
		box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
	}

	.reset-button:hover {
		background: #357abd !important;
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(74, 144, 226, 0.3);
	}

	.reset-button:active {
		transform: translateY(0);
	}

	@keyframes shake {
		0%, 100% { transform: translateX(0) scale(1); }
		25% { transform: translateX(-10px) scale(1); }
		75% { transform: translateX(10px) scale(1); }
	}

	@media (max-width: 768px) {
		.cards-container {
			grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)) !important;
			gap: 0.75rem !important;
		}
	}
</style>
