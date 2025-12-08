---
layout: default
title: املأ الفراغات
---

<div class="container">
	<header>
		<h1 class="arabic-text">املأ الفراغات</h1>
		<p class="section-subtitle arabic-text" style="color: #333333; margin-top: 1rem;">
			املأ الفراغات بالكلمات الصحيحة
		</p>
	</header>

	<main id="fillBlanksContent">
		<div style="text-align: center; color: #333333; font-size: 1.2rem; padding: 2rem;">
			<span class="arabic-text">جاري التحميل...</span>
		</div>
	</main>
</div>

<script>
	const EXERCISES_PATH = "fill_blanks/exercises.json";
	let allExercises = [];
	let shownExercises = [];
	let currentPageExercises = [];
	let currentAnswers = {};

	// Load exercises data
	async function loadExercises() {
		if (allExercises.length > 0) {
			return;
		}

		try {
			const response = await fetch(EXERCISES_PATH);
			if (!response.ok) {
				throw new Error(`Failed to load ${EXERCISES_PATH}`);
			}
			allExercises = await response.json();
			console.log(`Loaded ${allExercises.length} exercises`);
			loadPage();
		} catch (error) {
			console.error("Error loading exercises:", error);
			document.getElementById("fillBlanksContent").innerHTML =
				'<section class="section"><p style="color: #ff6b6b; text-align: center;">Error loading exercises. Please ensure the file exists.</p></section>';
		}
	}

	// Get 10 random exercises that haven't been shown
	function getRandomExercises() {
		if (shownExercises.length >= allExercises.length) {
			// All exercises shown, reset
			shownExercises = [];
		}

		const available = allExercises.filter(ex => !shownExercises.includes(ex.id));
		const selected = [];
		
		for (let i = 0; i < 10 && available.length > 0; i++) {
			const randomIndex = Math.floor(Math.random() * available.length);
			const exercise = available.splice(randomIndex, 1)[0];
			selected.push(exercise);
			shownExercises.push(exercise.id);
		}

		return selected;
	}

	// Load a page with 10 exercises
	function loadPage() {
		currentPageExercises = getRandomExercises();
		currentAnswers = {};
		
		if (currentPageExercises.length === 0) {
			displayMessage("تم عرض جميع التمارين!");
			return;
		}

		displayExercises();
	}

	// Display exercises
	function displayExercises() {
		const content = document.getElementById("fillBlanksContent");
		let html = '<div class="exercises-container">';

		currentPageExercises.forEach((exercise, index) => {
			const exerciseId = `exercise-${exercise.id}`;
			const blankId = `blank-${exercise.id}`;
			const revealedAnswer = currentAnswers[exercise.id] || '';
			
			html += `
				<section class="section fill-blank-exercise" id="${exerciseId}">
					<div class="exercise-number">${index + 1}</div>
					<div class="exercise-content">
						<p class="exercise-sentence">${exercise.sentence.replace('___', `<span id="${blankId}" class="blank-span">${revealedAnswer || '______'}</span>`)}</p>
						<button class="reveal-button" onclick="revealAnswer(${exercise.id})">
							<span class="arabic-text">إظهار الإجابة</span>
						</button>
					</div>
				</section>
			`;
		});

		html += '</div>';
		
		// Add next page button
		const remaining = allExercises.length - shownExercises.length;
		if (remaining > 0) {
			html += `
				<div style="text-align: center; margin-top: 2rem;">
					<button class="random-button" onclick="loadNextPage()">
						<span class="arabic-text">الصفحة التالية (${remaining} متبقي)</span>
					</button>
				</div>
			`;
		} else {
			html += `
				<div style="text-align: center; margin-top: 2rem;">
					<button class="random-button" onclick="resetAndLoad()">
						<span class="arabic-text">ابدأ من جديد</span>
					</button>
				</div>
			`;
		}

		content.innerHTML = html;
	}


	// Reveal answer - randomly picks one correct answer
	function revealAnswer(exerciseId) {
		const exercise = currentPageExercises.find(ex => ex.id === exerciseId);
		if (!exercise) return;

		const blankEl = document.getElementById(`blank-${exerciseId}`);
		
		// Pick a random correct answer
		const randomIndex = Math.floor(Math.random() * exercise.correct.length);
		const randomAnswer = exercise.correct[randomIndex];
		
		// Fill in the answer
		blankEl.textContent = randomAnswer;
		blankEl.classList.add('revealed');
		currentAnswers[exerciseId] = randomAnswer;
	}

	// Load next page
	function loadNextPage() {
		loadPage();
	}

	// Reset and start over
	function resetAndLoad() {
		shownExercises = [];
		loadPage();
	}

	// Display message
	function displayMessage(message) {
		document.getElementById("fillBlanksContent").innerHTML = `
			<section class="section">
				<p style="color: #333333; text-align: center; font-size: 1.2rem;">${message}</p>
				<div style="text-align: center; margin-top: 1rem;">
					<button class="random-button" onclick="resetAndLoad()">
						<span class="arabic-text">ابدأ من جديد</span>
					</button>
				</div>
			</section>
		`;
	}

	// Make functions globally accessible
	window.revealAnswer = revealAnswer;
	window.loadNextPage = loadNextPage;
	window.resetAndLoad = resetAndLoad;

	// Initialize on page load
	loadExercises();
</script>

<style>
	.exercises-container {
		display: flex;
		flex-direction: column;
		gap: 1.5rem;
		margin-top: 2rem;
	}

	.fill-blank-exercise {
		display: flex;
		gap: 1rem;
		align-items: flex-start;
		padding: 1.5rem;
		background: #ffffff;
		border: 2px solid #e0e0e0;
		border-radius: 0.5rem;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
	}

	.exercise-number {
		font-size: 1.5rem;
		font-weight: bold;
		color: #4a90e2;
		min-width: 2.5rem;
		text-align: center;
		padding-top: 0.25rem;
	}

	.exercise-content {
		flex: 1;
	}

	.exercise-sentence {
		font-size: 1.1rem;
		color: #333333;
		margin: 0 0 1rem 0;
		line-height: 1.8;
	}

	.blank-span {
		display: inline-block;
		min-width: 60px;
		border-bottom: 2px solid #4a90e2;
		padding: 0 0.25rem;
		margin: 0 0.15rem;
		color: #333333;
		font-weight: 600;
		transition: all 0.2s;
		text-align: center;
	}

	.blank-span.revealed {
		border-bottom-color: #28a745;
		color: #28a745;
		background-color: rgba(40, 167, 69, 0.1);
		padding: 0 0.5rem;
		border-radius: 0.25rem;
	}

	.reveal-button {
		margin-top: 1rem;
		padding: 0.5rem 1rem;
		background: #666;
		color: white;
		border: none;
		border-radius: 0.5rem;
		cursor: pointer;
		font-weight: 600;
		font-size: 0.9rem;
		transition: all 0.2s;
	}

	.reveal-button:hover {
		background: #555;
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
	}

	.reveal-button:active {
		transform: translateY(0);
	}


	@media (max-width: 768px) {
		.fill-blank-exercise {
			flex-direction: column;
			padding: 1rem;
		}

		.exercise-number {
			align-self: flex-start;
		}

		.exercise-sentence {
			font-size: 1rem;
		}

		.blank-span {
			min-width: 50px;
		}
	}
</style>

