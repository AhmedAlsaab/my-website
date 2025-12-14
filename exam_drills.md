---
layout: default
title: تمارين الامتحان
---

<div class="container">
	<header>
		<h1 class="arabic-text">تمارين الامتحان</h1>
		<p class="section-subtitle arabic-text" style="color: #333333; margin-top: 1rem;">
			صحّح الجمل مع المعلم
		</p>
	</header>

	<main id="examDrillsContent">
		<div style="text-align: center; color: #333333; font-size: 1.2rem; padding: 2rem;">
			<span class="arabic-text">جاري التحميل...</span>
		</div>
	</main>
</div>

<script>
	const EXAM_DRILLS_PATH = "exam_drills/final_exam_drills.json";
	let examDrillsData = null;
	let shuffledDrills = {};
	let revealedSolutions = {};

	// Shuffle array function
	function shuffleArray(array) {
		const shuffled = [...array];
		for (let i = shuffled.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
		}
		return shuffled;
	}

	// Load exam drills data
	async function loadExamDrills() {
		if (examDrillsData) {
			return examDrillsData;
		}

		try {
			const response = await fetch(EXAM_DRILLS_PATH);
			if (!response.ok) {
				throw new Error(`Failed to load ${EXAM_DRILLS_PATH}`);
			}
			examDrillsData = await response.json();

			// Shuffle sentences for each drill section
			Object.keys(examDrillsData).forEach(drillKey => {
				if (examDrillsData[drillKey].sentences) {
					shuffledDrills[drillKey] = {
						...examDrillsData[drillKey],
						sentences: shuffleArray(examDrillsData[drillKey].sentences),
						currentIndex: 0
					};
					revealedSolutions[drillKey] = {};
				}
			});

			console.log("Loaded exam drills data");
			return examDrillsData;
		} catch (error) {
			console.error("Error loading exam drills:", error);
			document.getElementById("examDrillsContent").innerHTML =
				'<section class="section"><p style="color: #ff6b6b; text-align: center;">Error loading exam drills. Please ensure the file exists.</p></section>';
			return null;
		}
	}

	// Display drill section
	function displayDrillSection(drillKey, drillData) {
		const currentIndex = shuffledDrills[drillKey].currentIndex;
		const currentSentenceObj = shuffledDrills[drillKey].sentences[currentIndex];
		const currentSentence = typeof currentSentenceObj === 'string' ? currentSentenceObj : currentSentenceObj.sentence;
		const currentSolution = typeof currentSentenceObj === 'object' ? currentSentenceObj.solution : null;
		const currentExplanation = typeof currentSentenceObj === 'object' ? currentSentenceObj.explanation : null;
		const totalSentences = shuffledDrills[drillKey].sentences.length;
		const isSolutionRevealed = revealedSolutions[drillKey][currentIndex] || false;

		// Get section title in Arabic
		const sectionTitles = {
			"ing_drills": "تمارين الحاضر المستمر",
			"present_drills": "تمارين الحاضر البسيط",
			"future_drills": "تمارين المستقبل",
			"past_drills": "تمارين الماضي البسيط"
		};

		const sectionTitle = sectionTitles[drillKey] || drillKey;

		return `
			<section class="section drill-section" style="max-width: 900px; margin: 0 auto 3rem;">
				<h2 class="arabic-text" style="color: #1e3c72; font-size: clamp(1.5rem, 3vw, 2rem); margin-bottom: 1.5rem; border-bottom: 3px solid #4a90e2; padding-bottom: 0.75rem;">
					${sectionTitle}
				</h2>

				<!-- Key Reminders -->
				<div style="background: rgba(74, 144, 226, 0.1); padding: 1.5rem; border-radius: 0.5rem; margin-bottom: 2rem; border-left: 4px solid #4a90e2;">
					<p class="arabic-text" style="color: #1e3c72; font-weight: 700; margin-bottom: 1rem; font-size: 1.1rem; direction: rtl; text-align: right;">
						تذكيرات مهمة:
					</p>
					<ul class="arabic-text" style="direction: rtl; text-align: right; color: #1a1a1a; font-size: 1rem; line-height: 1.8; margin: 0; padding-right: 1.5rem;">
						${drillData.key_reminder_ar.map(reminder => `<li style="margin-bottom: 0.5rem;">${reminder}</li>`).join("")}
					</ul>
				</div>

				<!-- Instructions -->
				<div style="margin-bottom: 2rem;">
					<p class="arabic-text" style="color: #1e3c72; font-weight: 600; font-size: 1.1rem; direction: rtl; text-align: right; margin-bottom: 1rem;">
						${drillData.instructions_ar}
					</p>
				</div>

				<!-- Current Sentence Card -->
				<div style="background: rgba(255, 255, 255, 0.98); padding: 2.5rem; border-radius: 1rem; box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2); min-height: 450px; display: flex; flex-direction: column; justify-content: flex-start; align-items: center; margin-bottom: 1.5rem;">
					<p style="font-size: 1.5rem; color: #1e3c72; line-height: 1.8; margin: 0 0 1.5rem 0; text-align: center; font-weight: 500;">
						${currentSentence}
					</p>
					${currentSolution ? `
						<div id="solution-${drillKey}-${currentIndex}" style="width: 100%; margin-top: 1.5rem; padding-top: 1.5rem; border-top: 2px solid rgba(74, 144, 226, 0.3); visibility: ${isSolutionRevealed ? 'visible' : 'hidden'}; opacity: ${isSolutionRevealed ? '1' : '0'}; transition: opacity 0.3s ease, visibility 0.3s ease;">
							<p class="arabic-text" style="color: #1e3c72; font-weight: 700; margin-bottom: 0.75rem; font-size: 1rem; direction: rtl; text-align: right;">
								الإجابة الصحيحة:
							</p>
							<p style="font-size: 1.5rem; color: #28a745; line-height: 1.8; margin: 0 0 1rem 0; text-align: center; font-weight: 600;">
								${currentSolution}
							</p>
							${currentExplanation ? `
								<div style="margin-top: 1rem; padding-top: 1rem; border-top: 1px solid rgba(74, 144, 226, 0.2);">
									<p class="arabic-text" style="color: #1e3c72; font-weight: 700; margin-bottom: 0.5rem; font-size: 0.95rem; direction: rtl; text-align: right;">
										الشرح:
									</p>
									<p class="arabic-text" style="color: #2a5298; line-height: 1.8; margin: 0; direction: rtl; text-align: right; font-size: 1rem;">
										${currentExplanation}
									</p>
								</div>
							` : ''}
						</div>
					` : ''}
				</div>

				<!-- Reveal Solution Button -->
				${currentSolution ? `
					<div style="text-align: center; margin-bottom: 2rem;">
						<button 
							class="reveal-solution-button" 
							onclick="toggleSolution('${drillKey}')"
							style="padding: 0.75rem 2rem; background: ${isSolutionRevealed ? '#666' : '#4a90e2'}; color: white; border: none; border-radius: 0.5rem; cursor: pointer; font-weight: 600; font-size: 1rem; transition: all 0.2s;">
							<span class="arabic-text">${isSolutionRevealed ? 'إخفاء الإجابة' : 'إظهار الإجابة'}</span>
						</button>
					</div>
				` : ''}

				<!-- Navigation -->
				<div style="display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 1rem;">
					<button 
						class="nav-drill-button" 
						onclick="goToPreviousSentence('${drillKey}')" 
						${currentIndex === 0 ? 'disabled style="opacity: 0.5; cursor: not-allowed;"' : ''}
						style="padding: 0.75rem 1.5rem; background: #4a90e2; color: white; border: none; border-radius: 0.5rem; cursor: pointer; font-weight: 600; font-size: 1rem; transition: all 0.2s;">
						<span class="arabic-text">السابق</span>
					</button>
					<div style="text-align: center; flex: 1;">
						<p style="color: #1e3c72; background: rgba(255, 255, 255, 0.95); padding: 0.5rem 1rem; border-radius: 0.5rem; font-size: 0.95rem; margin: 0; font-weight: 600; display: inline-block; box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);">
							<span class="arabic-text">الجملة</span> ${currentIndex + 1} / ${totalSentences}
						</p>
					</div>
					<button 
						class="nav-drill-button" 
						onclick="goToNextSentence('${drillKey}')" 
						${currentIndex === totalSentences - 1 ? 'disabled style="opacity: 0.5; cursor: not-allowed;"' : ''}
						style="padding: 0.75rem 1.5rem; background: #4a90e2; color: white; border: none; border-radius: 0.5rem; cursor: pointer; font-weight: 600; font-size: 1rem; transition: all 0.2s;">
						<span class="arabic-text">التالي</span>
					</button>
				</div>
			</section>
		`;
	}

	// Display all drill sections
	function displayAllDrills() {
		const content = document.getElementById("examDrillsContent");
		let html = "";

		Object.keys(shuffledDrills).forEach(drillKey => {
			html += displayDrillSection(drillKey, shuffledDrills[drillKey]);
		});

		content.innerHTML = html;
	}

	// Navigate to previous sentence
	function goToPreviousSentence(drillKey) {
		if (shuffledDrills[drillKey].currentIndex > 0) {
			shuffledDrills[drillKey].currentIndex--;
			displayAllDrills();
		}
	}

	// Navigate to next sentence
	function goToNextSentence(drillKey) {
		const totalSentences = shuffledDrills[drillKey].sentences.length;
		if (shuffledDrills[drillKey].currentIndex < totalSentences - 1) {
			shuffledDrills[drillKey].currentIndex++;
			displayAllDrills();
		}
	}

	// Toggle solution visibility
	function toggleSolution(drillKey) {
		const currentIndex = shuffledDrills[drillKey].currentIndex;
		revealedSolutions[drillKey][currentIndex] = !revealedSolutions[drillKey][currentIndex];
		
		// Smoothly toggle the solution visibility without re-rendering everything
		const solutionElement = document.getElementById(`solution-${drillKey}-${currentIndex}`);
		if (solutionElement) {
			if (revealedSolutions[drillKey][currentIndex]) {
				solutionElement.style.visibility = 'visible';
				solutionElement.style.opacity = '1';
			} else {
				solutionElement.style.visibility = 'hidden';
				solutionElement.style.opacity = '0';
			}
		} else {
			// Fallback: re-render if element not found
			displayAllDrills();
		}
	}

	// Make functions globally accessible
	window.goToPreviousSentence = goToPreviousSentence;
	window.goToNextSentence = goToNextSentence;
	window.toggleSolution = toggleSolution;

	// Load and display drills on page load
	(async function() {
		await loadExamDrills();
		if (examDrillsData) {
			displayAllDrills();
		}
	})();
</script>

<style>
	.nav-drill-button:hover:not(:disabled) {
		background: #357abd !important;
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(74, 144, 226, 0.3);
	}

	.nav-drill-button:active:not(:disabled) {
		transform: translateY(0);
	}

	.reveal-solution-button:hover {
		background: #357abd !important;
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(74, 144, 226, 0.3);
	}

	.reveal-solution-button:active {
		transform: translateY(0);
	}

	.drill-section {
		transition: all 0.3s ease;
	}

	@media (max-width: 768px) {
		.drill-section {
			margin-bottom: 2rem !important;
		}
	}
</style>

