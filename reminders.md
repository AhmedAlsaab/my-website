---
layout: default
title: تذكيرات أساسية للامتحان
---

<div class="container">
	<header>
		<h1 class="arabic-text">تذكيرات أساسية للامتحان</h1>
		<p class="section-subtitle arabic-text" style="color: #333333; margin-top: 1rem;">
			مراجعة سريعة للنقاط المهمة قبل الامتحان
		</p>
	</header>

	<main id="remindersContent">
		<p style="text-align: center; color: #333333; font-size: 1.2rem; padding: 2rem;">
			<span class="arabic-text">جاري التحميل...</span>
		</p>
	</main>
</div>

<script>
	const REMINDERS_PATH = "key_reminders/reminders.json";
	let remindersData = null;
	let currentCardIndex = 0;

	// Load reminders data
	async function loadReminders() {
		if (remindersData) {
			return remindersData;
		}

		try {
			const response = await fetch(REMINDERS_PATH);
			if (!response.ok) {
				throw new Error(`Failed to load ${REMINDERS_PATH}`);
			}
			remindersData = await response.json();
			console.log(`Loaded ${remindersData.cards.length} reminder cards`);
			return remindersData;
		} catch (error) {
			console.error("Error loading reminders:", error);
			document.getElementById("remindersContent").innerHTML =
				'<section class="section"><p style="color: #ff6b6b; text-align: center;">Error loading reminders. Please ensure the file exists.</p></section>';
			return null;
		}
	}

	// Display a specific card
	function displayCard(index) {
		if (!remindersData || !remindersData.cards || index < 0 || index >= remindersData.cards.length) {
			return;
		}

		const card = remindersData.cards[index];
		const content = document.getElementById("remindersContent");
		const totalCards = remindersData.cards.length;
		currentCardIndex = index;

		// Build body text HTML
		let bodyHtml = "";
		if (card.body_ar && Array.isArray(card.body_ar)) {
			bodyHtml = card.body_ar.map(line => `<p style="margin-bottom: 0.75rem; line-height: 1.8; color: #1a1a1a;">${line}</p>`).join("");
		}

		// Build examples HTML with Arabic only
		let examplesHtml = "";
		const hasExamples = card.examples_ar && card.examples_ar.length > 0;
		
		if (hasExamples) {
			const examplesAr = card.examples_ar || [];
			
			examplesHtml = `
				<div style="margin-top: 2rem; padding-top: 1.5rem; border-top: 2px solid rgba(74, 144, 226, 0.3);">
					<p class="arabic-text" style="color: #1e3c72; font-weight: 700; margin-bottom: 1rem; font-size: 1.2rem; direction: rtl; text-align: right;">أمثلة:</p>
					<div style="display: flex; flex-direction: column; gap: 1rem;">
						${examplesAr.map(arExample => `
							<div style="background: rgba(74, 144, 226, 0.08); padding: 1rem 1.25rem; border-radius: 0.5rem; border-left: 3px solid #4a90e2;">
								<p class="arabic-text" style="color: #2a5298; margin: 0; direction: rtl; text-align: right; font-size: 1rem;">${arExample}</p>
							</div>
						`).join("")}
					</div>
				</div>
			`;
		}

		const html = `
			<section class="section" style="max-width: 900px; margin: 0 auto;">
				<div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 2rem; flex-wrap: wrap; gap: 1rem;">
					<button 
						class="nav-card-button" 
						onclick="goToPreviousCard()" 
						${index === 0 ? 'disabled style="opacity: 0.5; cursor: not-allowed;"' : ''}
						style="padding: 0.75rem 1.5rem; background: #4a90e2; color: white; border: none; border-radius: 0.5rem; cursor: pointer; font-weight: 600; font-size: 1rem; transition: all 0.2s;">
						<span class="arabic-text">السابق</span>
					</button>
					<div style="text-align: center; flex: 1;">
						<p style="color: #1e3c72; background: rgba(255, 255, 255, 0.95); padding: 0.5rem 1rem; border-radius: 0.5rem; font-size: 0.95rem; margin: 0; font-weight: 600; display: inline-block; box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);">
							<span class="arabic-text">البطاقة</span> ${index + 1} / ${totalCards}
						</p>
					</div>
					<button 
						class="nav-card-button" 
						onclick="goToNextCard()" 
						${index === totalCards - 1 ? 'disabled style="opacity: 0.5; cursor: not-allowed;"' : ''}
						style="padding: 0.75rem 1.5rem; background: #4a90e2; color: white; border: none; border-radius: 0.5rem; cursor: pointer; font-weight: 600; font-size: 1rem; transition: all 0.2s;">
						<span class="arabic-text">التالي</span>
					</button>
				</div>

				<div class="reminder-card" style="background: rgba(255, 255, 255, 0.98); padding: 2.5rem; border-radius: 1rem; box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2); min-height: 300px;">
					<h2 class="arabic-text" style="color: #1e3c72; font-size: clamp(1.5rem, 3vw, 2rem); margin-bottom: 1.5rem; border-bottom: 3px solid #4a90e2; padding-bottom: 0.75rem; direction: rtl; text-align: right; font-weight: 700;">
						${card.title_ar}
					</h2>
					<div class="arabic-text" style="direction: rtl; text-align: right; color: #1a1a1a; font-size: clamp(1.05rem, 2vw, 1.25rem); line-height: 1.9; font-weight: 500;">
						${bodyHtml}
					</div>
					${examplesHtml}
				</div>
			</section>
		`;

		content.innerHTML = html;
	}

	// Navigate to previous card
	function goToPreviousCard() {
		if (currentCardIndex > 0) {
			displayCard(currentCardIndex - 1);
		}
	}

	// Navigate to next card
	function goToNextCard() {
		if (remindersData && currentCardIndex < remindersData.cards.length - 1) {
			displayCard(currentCardIndex + 1);
		}
	}

	// Make functions globally accessible
	window.goToPreviousCard = goToPreviousCard;
	window.goToNextCard = goToNextCard;

	// Keyboard navigation
	document.addEventListener('keydown', function(event) {
		if (event.key === 'ArrowLeft') {
			goToPreviousCard();
		} else if (event.key === 'ArrowRight') {
			goToNextCard();
		}
	});

	// Load and display first card on page load
	(async function() {
		await loadReminders();
		if (remindersData && remindersData.cards && remindersData.cards.length > 0) {
			displayCard(0);
		}
	})();
</script>

<style>
	.nav-card-button:hover:not(:disabled) {
		background: #357abd !important;
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(74, 144, 226, 0.3);
	}

	.nav-card-button:active:not(:disabled) {
		transform: translateY(0);
	}

	.reminder-card {
		transition: all 0.3s ease;
	}

	@media (max-width: 768px) {
		.reminder-card {
			padding: 1.5rem !important;
		}
	}
</style>

