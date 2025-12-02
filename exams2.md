---
layout: default
title: الامتحان 2
---

<div class="exam-container">
	<header class="exam-header">
		<h1 class="exam-title"><strong>LanguageCert Preliminary A1 - Speaking & Listening</strong></h1>
		<div class="exam-controls">
			<div class="controls-group">
				<button id="timerBtn" class="timer-btn">09:00</button>
				<button id="nextExamBtn" class="next-exam-btn">الامتحان التالي</button>
				<span id="examCounter" class="exam-counter">1 / 21</span>
			</div>
		</div>
		<div class="exam-meta">
			<p class="exam-level">CEFR A1</p>
			<p class="exam-time">⏱️ Total Time: 8-9 minutes</p>
		</div>
	</header>

	<main id="examContent">
		<div style="text-align: center; padding: 2rem;">
			<p>Loading exam...</p>
		</div>
	</main>
</div>

<script>
	// List of all exam files
	const EXAM_FILES = [
		"new_exams/a1_speaking_practice_paper_1.json",
		"new_exams/a1_speaking_practice_paper_2.json",
		"new_exams/a1_speaking_practice_paper_3.json",
		"new_exams/a1_speaking_practice_paper_4.json",
		"new_exams/a1_speaking_practice_paper_5.json",
		"new_exams/a1_speaking_practice_paper_6.json",
		"new_exams/a1_speaking_practice_paper_7.json",
		"new_exams/a1_speaking_practice_paper_8.json",
		"new_exams/a1_speaking_practice_paper_9.json",
		"new_exams/a1_speaking_practice_paper_10.json",
		"new_exams/a1_speaking_practice_paper_11.json",
		"new_exams/a1_speaking_practice_paper_12.json",
		"new_exams/a1_speaking_practice_paper_13.json",
		"new_exams/a1_speaking_practice_paper_14.json",
		"new_exams/a1_speaking_practice_paper_15.json",
		"new_exams/a1_speaking_practice_paper_16.json",
		"new_exams/a1_speaking_practice_paper_17.json",
		"new_exams/a1_speaking_practice_paper_18.json",
		"new_exams/a1_speaking_practice_paper_19.json",
		"new_exams/a1_speaking_practice_paper_20.json",
		"new_exams/a1_speaking_practice_paper_21.json"
	];

	let shuffledExams = [];
	let currentExamIndex = 0;
	let examData = null;
	let timerInterval = null;
	let timeRemaining = 9 * 60; // 9 minutes in seconds
	let isTimerRunning = false;
	let imageList = [];
	let currentImage = null;

	// Shuffle array function
	function shuffleArray(array) {
		const shuffled = [...array];
		for (let i = shuffled.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
		}
		return shuffled;
	}

	// Load image list - matching the pattern from scripts.html
	async function loadImageList() {
		if (imageList.length > 0) {
			return imageList;
		}

		const IMAGES_DIR = "past_papers_images/";

		try {
			// Try to load index.json from images directory
			const indexPath = `${IMAGES_DIR}index.json`;
			const response = await fetch(indexPath);
			if (response.ok) {
				const index = await response.json();
				if (Array.isArray(index)) {
					imageList = index.map((file) =>
						file.endsWith(".png") ? `${IMAGES_DIR}${file}` : `${IMAGES_DIR}${file}.png`
					);
				} else if (index.images && Array.isArray(index.images)) {
					imageList = index.images.map((file) =>
						file.endsWith(".png") ? `${IMAGES_DIR}${file}` : `${IMAGES_DIR}${file}.png`
					);
				}
				console.log(`Loaded ${imageList.length} images from index`);
				return imageList;
			}
		} catch (error) {
			console.log("No images index.json found, using fallback list");
		}

		// Fallback: use hardcoded list
		imageList = [
			`${IMAGES_DIR}ChatGPT Image Nov 30, 2025, 01_00_27 PM.png`,
			`${IMAGES_DIR}ChatGPT Image Nov 30, 2025, 01_07_47 PM.png`,
			`${IMAGES_DIR}ChatGPT Image Nov 30, 2025, 01_09_46 PM.png`,
			`${IMAGES_DIR}ChatGPT Image Nov 30, 2025, 12_29_40 PM.png`,
			`${IMAGES_DIR}ChatGPT Image Nov 30, 2025, 12_32_22 PM.png`,
			`${IMAGES_DIR}ChatGPT Image Nov 30, 2025, 12_34_05 PM.png`,
			`${IMAGES_DIR}ChatGPT Image Nov 30, 2025, 12_55_07 PM.png`,
			`${IMAGES_DIR}ChatGPT Image Nov 30, 2025, 12_56_45 PM.png`
		];
		return imageList;
	}

	// Get random image - returns full path (synchronous, not async)
	function getRandomExamImage() {
		console.log("getRandomExamImage called - imageList:", imageList, "length:", imageList ? imageList.length : 0);
		if (!imageList || imageList.length === 0) {
			console.log("Image list is empty or undefined");
			return null;
		}
		const randomIndex = Math.floor(Math.random() * imageList.length);
		const selectedImage = imageList[randomIndex];
		console.log("Selected image at index", randomIndex, ":", selectedImage);
		// Ensure it's a string, not a Promise or other object
		if (typeof selectedImage !== 'string') {
			console.error("Selected image is not a string:", selectedImage, "Type:", typeof selectedImage);
			return null;
		}
		console.log("Selected image path:", selectedImage);
		return selectedImage;
	}

	// Initialize shuffled exams on first load
	function initializeExams() {
		if (shuffledExams.length === 0) {
			shuffledExams = shuffleArray(EXAM_FILES);
		}
	}

	// Load exam data
	async function loadExamData(examPath) {
		try {
			const response = await fetch(examPath);
			if (!response.ok) {
				throw new Error(`Failed to load ${examPath}`);
			}
			examData = await response.json();
			console.log("Loaded exam data:", examData);
			return examData;
		} catch (error) {
			console.error("Error loading exam data:", error);
			document.getElementById("examContent").innerHTML =
				'<div class="exam-section"><p style="color: #d32f2f; text-align: center;">Error loading exam data. Please ensure the file exists.</p></div>';
			return null;
		}
	}

	// Timer functions - using unique names to avoid conflicts with scripts.html
	function toggleExamTimer() {
		if (isTimerRunning) {
			pauseExamTimer();
		} else {
			startExamTimer();
		}
	}

	function startExamTimer() {
		if (timeRemaining <= 0) {
			timeRemaining = 9 * 60; // Reset to 9 minutes if expired
		}
		
		isTimerRunning = true;
		updateExamTimerDisplay();
		
		if (timerInterval) {
			clearInterval(timerInterval);
		}
		
		timerInterval = setInterval(() => {
			timeRemaining--;
			updateExamTimerDisplay();
			
			if (timeRemaining <= 0) {
				clearInterval(timerInterval);
				timerInterval = null;
				isTimerRunning = false;
				const timerBtn = document.getElementById("timerBtn");
				if (timerBtn) {
					timerBtn.textContent = "00:00";
					timerBtn.classList.add("timer-expired");
					timerBtn.classList.remove("timer-running");
				}
			}
		}, 1000);
	}

	function pauseExamTimer() {
		isTimerRunning = false;
		if (timerInterval) {
			clearInterval(timerInterval);
			timerInterval = null;
		}
		updateExamTimerDisplay(); // Update display to remove running state
	}

	function resetExamTimer() {
		pauseExamTimer();
		timeRemaining = 9 * 60;
		updateExamTimerDisplay();
	}

	function updateExamTimerDisplay() {
		const minutes = Math.floor(timeRemaining / 60);
		const seconds = timeRemaining % 60;
		const timerElement = document.getElementById("timerBtn");
		if (timerElement) {
			const timeString = `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
			timerElement.textContent = timeString;
			timerElement.classList.remove("timer-expired");
			
			// Update button state visual indicator
			if (isTimerRunning) {
				timerElement.classList.add("timer-running");
			} else {
				timerElement.classList.remove("timer-running");
			}
		}
	}

	function updateExamCounter() {
		document.getElementById("examCounter").textContent = `${currentExamIndex + 1} / ${shuffledExams.length}`;
	}

	// Load next exam
	async function loadNextExam() {
		currentExamIndex = (currentExamIndex + 1) % shuffledExams.length;
		await loadAndDisplayExam();
		resetExamTimer();
	}

	// Display exam content
	function displayExam() {
		if (!examData) return;

		const content = document.getElementById("examContent");
		let html = "";

		// Exam instructions in Arabic
		html += `
			<div class="exam-instructions">
				<p><strong>ملاحظات مهمة:</strong></p>
				<p>عندما ترى "Interlocutor" أو "الممتحن" قبل السؤال أو النص، فهذا يعني أن الممتحن هو من سيقول هذا الكلام أو يطرح هذا السؤال.</p>
				<p>عندما ترى "Candidate" أو "الممتحَن" قبل النص، فهذا يعني أنك أنت من يجب أن تقول هذا الكلام أو تطرح هذا السؤال.</p>
				<p>تحتوي هذه الورقة على تعليمات للممتحن أيضاً. تعليمات الممتحن وما قد يقوله محدد باللون البرتقالي.</p>
			</div>
		`;

		// Part 1 - Interview (hardcoded structure)
		const part1 = examData.parts.part1;
		html += `
			<div class="exam-section">
				<div class="part-header">
					<h2>Part 1 – Interview</h2>
					<span class="duration">1 minute 30 seconds</span>
				</div>
				<div class="section-description">
					<p>في هذا الجزء، سيقوم الممتحن بطرح أسئلة شخصية بسيطة عليك. ستبدأ بأسئلة تعريفية ثابتة مثل تهجئة اسمك والعيش، ثم أسئلة شخصية عن روتينك اليومي واهتماماتك. استجب بشكل طبيعي وواضح.</p>
				</div>
				<div class="interlocutor-section">
					<h3 class="speaker-label">Interlocutor (I):</h3>
					<p class="dialogue orange-dialogue">LanguageCert ESOL International, Speaking and Listening, Preliminary level</p>
					
					<h4 class="subsection-title">Fixed Introduction Questions:</h4>
					<ul class="question-list">
						${part1.fixed_intro_questions.map(q => `
							<li class="question-item">
								<p class="dialogue"><strong>Interlocutor:</strong> ${q.text}</p>
							</li>
						`).join("")}
					</ul>
					
					<h4 class="subsection-title">Personal Questions:</h4>
					<ul class="question-list">
						${part1.personal_questions.map(q => `
							<li class="question-item">
								<p class="dialogue"><strong>Interlocutor:</strong> ${q.text}</p>
							</li>
						`).join("")}
					</ul>
					
					<p class="dialogue orange-dialogue"><strong>Interlocutor:</strong> Thank you.</p>
				</div>
			</div>
		`;

		// Part 2 - Role-play (hardcoded structure with contextual instructions)
		const part2 = examData.parts.part2;
		html += `
			<div class="exam-section">
				<div class="part-header">
					<h2>Part 2 – Role-play</h2>
					<span class="duration">1 minute 30 seconds</span>
				</div>
				<div class="section-description">
					<p>سيضعك الممتحن في مواقف يومية مختلفة. في الموقف الأول (A)، سيبدأ الممتحن بطرح سؤال عليك وعليك أن تجيب عليه. في الموقف الثاني (B)، ستحصل على سيناريو وعليك أن تبتكر سؤالاً مناسباً بناءً على الموقف. استخدم اللغة المناسبة للموقف (صديق، زميل، جار، زميل دراسة) وتفاعل بشكل طبيعي.</p>
				</div>
				<div class="interlocutor-section">
					<h3 class="speaker-label">Interlocutor (I):</h3>
					<p class="dialogue orange-dialogue"><strong>Interlocutor:</strong> Now, Part Two. I'm going to read some situations. I want you to start or answer. First situation (choose one situation from A).</p>
					
					<h4 class="subsection-title">Situation A (Interlocutor starts):</h4>
					<p class="instruction-note">(Role-play the situation with the candidate – approximately two turns each.)</p>
					<ul class="situation-list">
						${part2.situations_A.map(sit => `
							<li class="situation-item">
								<p class="context"><strong>Context:</strong> ${sit.context}</p>
								<p class="dialogue"><strong>Interlocutor:</strong> ${sit.interlocutor_opening}</p>
							</li>
						`).join("")}
					</ul>
					
					<h4 class="subsection-title">Situation B (Candidate starts):</h4>
					<p class="dialogue orange-dialogue"><strong>Interlocutor:</strong> Second situation (choose one situation from B).</p>
					<p class="instruction-note">(Role-play the situation with the candidate – approximately two turns each.)</p>
					<ul class="situation-list">
						${part2.situations_B.map(sit => `
							<li class="situation-item">
								<p class="context"><strong>Context:</strong> ${sit.context}</p>
								<p class="candidate-prompt"><strong>Interlocutor:</strong> ${sit.candidate_prompt}</p>
							</li>
						`).join("")}
					</ul>
					
					<p class="dialogue orange-dialogue"><strong>Interlocutor:</strong> Thank you.</p>
				</div>
			</div>
		`;

		// Part 3 - Comparing pictures (hardcoded structure)
		const part3 = examData.parts.part3;
		html += `
			<div class="exam-section">
				<div class="part-header">
					<h2>Part 3 – Comparing pictures (hotel)</h2>
					<span class="duration">1 minute – 1 minute 30 seconds</span>
				</div>
				<div class="section-description">
					<p>ستحصل على صورتين. عن الصورة الأولى، سيسألك الممتحن أسئلة وعليك الإجابة عليها. عن الصورة الثانية، أنت من ستبتكر الأسئلة وطرحها على الممتحن. استخدم كلمات بسيطة لوصف ما تراه واطرح أسئلة واضحة.</p>
				</div>
				<div class="interlocutor-section">
					<h3 class="speaker-label">Interlocutor (I):</h3>
					<p class="dialogue orange-dialogue"><strong>Interlocutor:</strong> Here are two pictures. Let's ask and answer questions about the two pictures. I'll ask you questions from the left picture which you will answer, and you will ask me questions from the right picture which I will answer.</p>
					<div class="exam-image-container">
						<img id="part3-image" src="" alt="Exam pictures" class="exam-image" />
					</div>
					<p class="dialogue orange-dialogue"><strong>Interlocutor:</strong> Thank you.</p>
				</div>
			</div>
		`;

		// Part 4 - Listening and long turn (hardcoded structure)
		const part4 = examData.parts.part4;
		
		// Part 4A - Listening task
		const part4A = part4.subparts.part4A;
		html += `
			<div class="exam-section">
				<div class="part-header">
					<h2>Part 4A – Listening task</h2>
					<span class="duration">2 minutes – 2 minutes 30 seconds</span>
				</div>
				<div class="section-description">
					<p>سيقرأ الممتحن قصة قصيرة مرتين. استمع جيداً واكتب ملاحظات على ورقتك. بعد ذلك، سيسألك الممتحن أسئلة عن القصة. ركز على المعلومات المهمة مثل الوظيفة، الوقت، والأماكن.</p>
				</div>
				<div class="interlocutor-section">
					<h3 class="speaker-label">Interlocutor (I):</h3>
					<p class="dialogue orange-dialogue"><strong>Interlocutor:</strong> In Part Four I'm going to read something. I'm going to tell you about ______'s best friend. I'll read it two times. Listen and take notes on your paper. I'll then ask you these questions.</p>
					<p class="instruction-note">(Allow 10 seconds for the candidate to read the questions.)</p>
					
					<h4 class="subsection-title">Listening Script:</h4>
					<p class="instruction-note">(Read script at an appropriate pace.)</p>
					<ul class="question-list">
						<li class="question-item purple-item">
							<p class="dialogue"><strong>Interlocutor:</strong> <span class="script-text">${part4A.listening_script}</span></p>
						</li>
					</ul>
					
					<p class="dialogue orange-dialogue"><strong>Interlocutor:</strong> Now I'll read it again. (Read the script again.)</p>
					
					<h4 class="subsection-title">Questions:</h4>
					<p class="dialogue orange-dialogue"><strong>Interlocutor:</strong> Questions. (Ask the following questions, allowing time for the candidate to respond orally.)</p>
					<ul class="question-list">
						${part4A.questions.map(q => `
							<li class="question-item">
								<p class="dialogue"><strong>Interlocutor:</strong> ${q.text}</p>
							</li>
						`).join("")}
					</ul>
					
					<p class="dialogue orange-dialogue"><strong>Interlocutor:</strong> Thank you.</p>
				</div>
			</div>
		`;

		// Part 4B - Long turn and follow-up
		const part4B = part4.subparts.part4B;
		html += `
			<div class="exam-section">
				<div class="part-header">
					<h2>Part 4B – Long turn and follow-up</h2>
					<span class="duration">2 minutes including follow-up questions</span>
				</div>
				<div class="section-description">
					<p>ستتحدث وحدك لمدة نصف دقيقة عن موضوع محدد. ستحصل على 30 ثانية لكتابة ملاحظات قبل البدء. تحدث بوضوح عن الموضوع باستخدام كلمات بسيطة. بعد ذلك، سيسألك الممتحن أسئلة متابعة عن الموضوع.</p>
				</div>
				<div class="interlocutor-section">
					<h3 class="speaker-label">Interlocutor (I):</h3>
					<p class="dialogue orange-dialogue"><strong>Interlocutor:</strong> Now you're going to talk on your own for half a minute. Your topic is ______. You now have thirty seconds to write some notes to help you. You're going to talk about your topic.</p>
					
					<h4 class="subsection-title">Topic:</h4>
					<ul class="question-list">
						<li class="question-item purple-item">
							<p class="dialogue"><strong>Interlocutor:</strong> <span class="topic-title">${part4B.topic.title}</span></p>
						</li>
					</ul>
					
					<p class="dialogue orange-dialogue">(Candidate's name), please start.</p>
					<p class="instruction-note">(When candidate has talked for a maximum of half a minute, say, 'Thank you', and then ask follow-up questions, as time allows.)</p>
					
					<h4 class="subsection-title">Follow-up Questions:</h4>
					<ul class="question-list">
						${part4B.follow_up_questions.map(q => `
							<li class="question-item">
								<p class="dialogue"><strong>Interlocutor:</strong> ${q.text}</p>
							</li>
						`).join("")}
					</ul>
					
					<p class="dialogue orange-dialogue"><strong>Interlocutor:</strong> Thank you, (give candidate's name). That is the end of the exam.</p>
				</div>
			</div>
		`;

		content.innerHTML = html;

		// Load and set image after content is rendered (matching scripts.html pattern)
		// Use setTimeout to ensure DOM is ready
		setTimeout(() => {
			const imgElement = document.getElementById("part3-image");
			if (!imgElement) {
				console.error("Image element not found");
				return;
			}
			
			if (currentImage && typeof currentImage === 'string' && currentImage.trim() !== '') {
				console.log("Setting image src to:", currentImage);
				// Ensure the path is a valid string
				const imagePath = String(currentImage).trim();
				imgElement.src = imagePath;
				imgElement.onerror = function () {
					console.error("Failed to load image:", imagePath);
					const container = this.closest(".exam-image-container");
					if (container) {
						container.style.display = "none";
					}
				};
			} else {
				console.log("No image available (currentImage:", currentImage, "), hiding container");
				const container = imgElement.closest(".exam-image-container");
				if (container) {
					container.style.display = "none";
				}
			}
		}, 0);
	}

	// Load and display exam
	async function loadAndDisplayExam() {
		initializeExams();
		
		// Load images first and wait for completion
		await loadImageList();
		console.log("Image list after loading:", imageList, "Length:", imageList.length);
		
		// Get random image - ensure it's a string (synchronous function)
		const randomImage = getRandomExamImage();
		console.log("Random image from getRandomExamImage():", randomImage, "Type:", typeof randomImage);
		
		if (randomImage && typeof randomImage === 'string') {
			currentImage = randomImage; // Store as string
			console.log("Current image path set to:", currentImage, "Type:", typeof currentImage);
		} else {
			currentImage = null;
			console.log("No valid image selected - imageList length:", imageList.length);
		}
		
		const examPath = shuffledExams[currentExamIndex];
		await loadExamData(examPath);
		if (examData) {
			displayExam();
			updateExamCounter();
		}
	}

	// Load and display exam on page load
	(async function() {
		initializeExams();
		// Start with the first exam in the shuffled array
		currentExamIndex = 0;
		await loadAndDisplayExam();
		resetExamTimer();
		
		// Set up button event listeners - wait for DOM to be ready
		setTimeout(() => {
			const nextBtn = document.getElementById("nextExamBtn");
			const timerBtn = document.getElementById("timerBtn");
			
			if (nextBtn) {
				nextBtn.addEventListener("click", loadNextExam);
			} else {
				console.error("Next exam button not found");
			}
			
			if (timerBtn) {
				timerBtn.addEventListener("click", toggleExamTimer);
				console.log("Timer button event listener attached");
			} else {
				console.error("Timer button not found");
			}
		}, 100);
	})();
</script>

<style>
	.exam-container {
		background: #ffffff;
		color: #000000;
		min-height: 100vh;
		padding: 0;
		margin: 0;
	}

	.exam-header {
		background: #ffffff;
		color: #000000;
		padding: 2rem 2rem 1.5rem 2rem;
		text-align: center;
		border-bottom: 2px solid #000000;
	}

	.exam-title {
		font-size: 2rem;
		font-weight: 900;
		margin: 0 0 0.75rem 0;
		color: #000000 !important;
		letter-spacing: -0.5px;
		line-height: 1.2;
		background: none !important;
		-webkit-background-clip: unset !important;
		-webkit-text-fill-color: #000000 !important;
		background-clip: unset !important;
	}

	.exam-title strong {
		font-weight: 900;
		color: #000000 !important;
		-webkit-text-fill-color: #000000 !important;
	}

	.exam-controls {
		display: flex;
		justify-content: center;
		align-items: center;
		margin-bottom: 1rem;
		flex-wrap: wrap;
		gap: 1rem;
	}

	.controls-group {
		display: flex;
		align-items: center;
		gap: 1rem;
		flex-wrap: wrap;
		justify-content: center;
	}

	.timer-btn {
		font-size: 1.2rem;
		font-weight: 700;
		color: #000000;
		background-color: #f0f0f0;
		padding: 0.5rem 1rem;
		border-radius: 4px;
		font-family: monospace;
		border: 2px solid #000000;
		cursor: pointer;
		transition: all 0.3s ease;
		min-width: 80px;
		user-select: none;
		-webkit-user-select: none;
		-moz-user-select: none;
		-ms-user-select: none;
	}

	.timer-btn:hover {
		background-color: #e0e0e0;
	}

	.timer-btn:active {
		transform: scale(0.98);
	}

	.timer-expired {
		color: #ffffff;
		background-color: #ff6b6b;
		border-color: #ff6b6b;
	}

	.next-exam-btn {
		background-color: #000000;
		color: #ffffff;
		border: 2px solid #000000;
		padding: 0.5rem 1.5rem;
		border-radius: 4px;
		font-size: 0.95rem;
		font-weight: 600;
		cursor: pointer;
		transition: all 0.3s ease;
	}

	.next-exam-btn:hover {
		background-color: #333333;
		border-color: #333333;
	}

	.exam-counter {
		font-size: 0.95rem;
		color: #000000;
		font-weight: 600;
		background-color: #f0f0f0;
		padding: 0.5rem 1rem;
		border-radius: 4px;
		border: 2px solid #000000;
	}

	.exam-meta {
		display: flex;
		justify-content: center;
		gap: 2rem;
		flex-wrap: wrap;
		margin-top: 0.5rem;
	}

	.exam-level,
	.exam-time {
		margin: 0;
		font-size: 0.95rem;
		color: #000000;
		font-weight: 500;
	}

	#examContent {
		max-width: 800px;
		margin: 0 auto;
		padding: 3rem 2rem;
		background: #ffffff;
		line-height: 1.6;
	}

	.exam-instructions {
		background-color: #fff7ed;
		border: 2px solid #ffedd5;
		padding: 1.5rem;
		margin-bottom: 2rem;
		border-radius: 4px;
	}

	.exam-instructions p {
		margin: 0.75rem 0;
		color: #333333;
		line-height: 1.7;
		text-align: right;
		direction: rtl;
	}

	.exam-instructions p:first-child {
		margin-top: 0;
	}

	.exam-instructions p:last-child {
		margin-bottom: 0;
	}

	.exam-instructions strong {
		color: #000000;
		font-weight: 700;
	}

	.exam-section {
		background: #ffffff;
		margin-bottom: 3rem;
		padding: 0;
		page-break-inside: avoid;
	}

	.part-header {
		display: flex;
		justify-content: space-between;
		align-items: baseline;
		margin-bottom: 1.5rem;
		padding-bottom: 0.5rem;
		border-bottom: 1px solid #000000;
	}

	.part-header h2 {
		font-size: 1.4rem;
		font-weight: 700;
		margin: 0;
		color: #000000;
	}

	.duration {
		color: #000000;
		font-weight: 600;
		font-size: 0.95rem;
	}

	.section-description {
		background-color: #f8f9fa;
		padding: 1rem 1.5rem;
		margin: 1rem 0;
		border-left: 3px solid #666666;
		border-radius: 4px;
	}

	.section-description p {
		margin: 0;
		color: #333333;
		line-height: 1.6;
		font-size: 0.95rem;
		text-align: right;
		direction: rtl;
	}

	.interlocutor-section {
		color: #000000;
		margin-top: 1rem;
	}

	.speaker-label {
		font-size: 1.1rem;
		font-weight: 700;
		margin: 0 0 1rem 0;
		color: #000000;
	}

	.subsection-title {
		font-size: 1rem;
		font-weight: 600;
		margin: 2rem 0 0.75rem 0;
		color: #000000;
		text-transform: uppercase;
		letter-spacing: 0.5px;
	}

	.dialogue {
		margin: 0.5rem 0;
		padding: 0;
		line-height: 1.7;
		color: #000000;
	}

	.dialogue p {
		margin: 0.5rem 0;
		line-height: 1.7;
		color: #000000;
	}

	.situation-list {
		list-style: none;
		padding: 0;
		margin: 1rem 0;
	}

	.situation-item {
		margin: 1.25rem 0;
		padding-left: 1.5rem;
		position: relative;
	}

	.situation-item::before {
		content: "•";
		position: absolute;
		left: 0;
		font-size: 1.2rem;
		color: #000000;
		font-weight: bold;
	}

	.context {
		font-weight: 600;
		margin: 0 0 0.5rem 0;
		color: #000000;
	}

	.candidate-prompt {
		margin: 0.5rem 0;
		padding: 0;
		color: #000000;
		line-height: 1.7;
	}

	.candidate-label {
		color: #0066cc;
		font-weight: 700;
	}

	.notes {
		margin: 0.5rem 0 0 0;
		color: #666666;
		font-size: 0.9rem;
		font-style: italic;
	}

	.instruction-note {
		margin: 0.5rem 0;
		padding: 0;
		color: #666666;
		font-size: 0.9rem;
		font-style: italic;
	}

	.script-text {
		line-height: 1.8;
		color: #000000;
	}

	.topic-title {
		font-size: 1.1rem;
		margin: 0.5rem 0;
		color: #000000;
		font-weight: 600;
	}

	.topic-prompt {
		margin: 0.5rem 0 0 0;
		color: #000000;
		line-height: 1.7;
	}

	.question-list {
		list-style: none;
		padding: 0;
		margin: 0.75rem 0;
	}

	.question-item {
		margin: 0.75rem 0;
		padding-left: 1.5rem;
		position: relative;
	}

	.question-item::before {
		content: "•";
		position: absolute;
		left: 0;
		font-size: 1.2rem;
		color: #000000;
		font-weight: bold;
	}

	.purple-item {
		background-color: #e9d5ff;
		padding: 0.75rem 1rem;
		margin: 0.75rem 0;
		border-radius: 4px;
	}

	.purple-item::before {
		color: #7b2cbf;
	}

	.purple-item .dialogue,
	.purple-item .dialogue strong,
	.purple-item .script-text,
	.purple-item .topic-title,
	.purple-item .topic-prompt {
		color: #000000;
	}

	.orange-dialogue {
		background-color: #ffedd5;
		padding: 0.75rem 1rem;
		margin: 0.5rem 0;
		border-radius: 4px;
		color: #000000;
	}

	.orange-dialogue strong {
		color: #000000;
	}

	.exam-image-container {
		margin: 1.5rem 0;
		text-align: center;
	}

	.exam-image {
		max-width: 100%;
		width: 100%;
		height: auto;
		border: 2px solid #000000;
		border-radius: 4px;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
		display: block;
		margin: 0 auto;
	}

	.timer-running {
		background-color: #d4edda !important;
		border-color: #28a745 !important;
	}

	@media (max-width: 768px) {
		.exam-header {
			padding: 1rem 0.75rem;
		}

		.exam-title {
			font-size: 1.5rem;
			font-weight: 900;
			margin-bottom: 0.5rem;
			line-height: 1.3;
		}

		.exam-controls {
			flex-direction: column;
			align-items: stretch;
			gap: 0.75rem;
			margin-bottom: 0.75rem;
		}

		.controls-group {
			width: 100%;
			justify-content: center;
			flex-wrap: wrap;
			gap: 0.75rem;
		}

		.timer-btn {
			flex: 1;
			min-width: 100px;
			font-size: 1rem;
			padding: 0.5rem 0.75rem;
		}

		.next-exam-btn {
			flex: 1;
			min-width: 140px;
			font-size: 0.9rem;
			padding: 0.5rem 1rem;
		}

		.exam-counter {
			font-size: 0.85rem;
			padding: 0.4rem 0.75rem;
		}

		.exam-meta {
			flex-direction: column;
			gap: 0.5rem;
			margin-top: 0.5rem;
		}

		.exam-level,
		.exam-time {
			font-size: 0.85rem;
		}

		#examContent {
			padding: 1.5rem 1rem;
		}

		.exam-instructions {
			padding: 1rem;
			margin-bottom: 1.5rem;
		}

		.exam-instructions p {
			font-size: 0.9rem;
		}

		.exam-section {
			margin-bottom: 2rem;
		}

		.part-header {
			flex-direction: column;
			align-items: flex-start;
			gap: 0.5rem;
		}

		.part-header h2 {
			font-size: 1.2rem;
		}

		.duration {
			font-size: 0.85rem;
		}

		.situation-item,
		.question-item {
			padding-left: 1.25rem;
		}

		.exam-image-container {
			margin: 1rem 0;
			padding: 0;
		}

		.exam-image {
			max-width: 100%;
			width: 100%;
			border-width: 1px;
		}

		.section-description {
			padding: 0.75rem 1rem;
			margin: 0.75rem 0;
		}

		.section-description p {
			font-size: 0.85rem;
		}

		.dialogue {
			font-size: 0.95rem;
		}

		.orange-dialogue {
			padding: 0.5rem 0.75rem;
		}

		.purple-item {
			padding: 0.5rem 0.75rem;
		}
	}
</style>

