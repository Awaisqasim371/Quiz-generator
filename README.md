#AI Quiz-generator
AA Quiz generator helps students to make multiple choice questions of a topic. Click on the link here
[Quiz generator](https://gemini.google.com/share/042585410111)























<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Quiz Generator (Gemini API)</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Use Inter font -->
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7f7f7;
            display: flex;
            justify-content: center;
            padding: 20px;
        }
        .container {
            width: 100%;
            max-width: 768px;
        }
        .card {
            background: #ffffff;
            border-radius: 1rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
            padding: 1.5rem;
            margin-bottom: 1.5rem;
        }
        .option-button {
            display: block;
            width: 100%;
            padding: 0.75rem 1rem;
            margin-bottom: 0.5rem;
            text-align: left;
            border-radius: 0.5rem;
            border: 1px solid #e5e7eb;
            transition: all 0.2s;
            cursor: pointer;
            color: #1f2937;
            background-color: #f9fafb;
        }
        .option-button:hover:not(.selected):not(.correct):not(.incorrect) {
            background-color: #eff6ff;
            border-color: #bfdbfe;
        }
        .option-button.selected {
            border-width: 2px;
            font-weight: 600;
        }
        .option-button.correct {
            background-color: #d1fae5; /* Green 100 */
            border-color: #10b981; /* Green 500 */
        }
        .option-button.incorrect {
            background-color: #fee2e2; /* Red 100 */
            border-color: #ef4444; /* Red 500 */
        }
        .disabled-button {
            cursor: not-allowed;
            opacity: 0.6;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1 class="text-3xl font-extrabold text-gray-900 mb-6 text-center">
            &#129504; AI Quiz Generator
        </h1>

        <!-- Input Section -->
        <div id="input-section" class="card">
            <label for="topic-input" class="block text-sm font-medium text-gray-700 mb-2">
                What topic should the quiz be about?
            </label>
            <input type="text" id="topic-input" placeholder="e.g., Quantum Computing, The French Revolution, React Hooks"
                   class="w-full p-3 border border-gray-300 rounded-lg mb-4 focus:ring-blue-500 focus:border-blue-500">
            
            <button id="generate-button"
                    class="w-full bg-blue-600 text-white p-3 rounded-lg font-semibold hover:bg-blue-700 transition duration-150 shadow-md">
                Generate 5-Question Quiz
            </button>
        </div>

        <!-- Loading Indicator -->
        <div id="loading-indicator" class="card hidden text-center">
            <div class="flex items-center justify-center space-x-2">
                <div class="w-4 h-4 rounded-full animate-pulse bg-blue-600"></div>
                <div class="w-4 h-4 rounded-full animate-pulse bg-blue-600 delay-150"></div>
                <div class="w-4 h-4 rounded-full animate-pulse bg-blue-600 delay-300"></div>
            </div>
            <p class="mt-3 text-gray-600">Generating quiz questions...</p>
        </div>

        <!-- Error Message -->
        <div id="error-message" class="card hidden bg-red-100 border border-red-400 text-red-700">
            <p class="font-bold">Error Generating Quiz</p>
            <p id="error-text" class="text-sm"></p>
        </div>

        <!-- Quiz Section -->
        <div id="quiz-section" class="card hidden">
            <h2 class="text-2xl font-bold mb-4" id="quiz-topic-title">Quiz:</h2>
            <div id="questions-container">
                <!-- Questions will be dynamically inserted here -->
            </div>
            <button id="submit-button"
                    class="w-full bg-green-600 text-white p-3 rounded-lg font-semibold hover:bg-green-700 transition duration-150 mt-6 shadow-md">
                Submit Quiz and See Score
            </button>
        </div>

        <!-- Results Section -->
        <div id="results-section" class="card hidden text-center">
            <h2 class="text-2xl font-bold text-gray-800 mb-4">Quiz Complete!</h2>
            <p class="text-4xl font-extrabold text-blue-600 mb-4">
                <span id="score-text">0/5</span>
            </p>
            <p id="feedback-text" class="text-lg text-gray-600 mb-6"></p>
            <button id="reset-button"
                    class="bg-blue-600 text-white p-3 rounded-lg font-semibold hover:bg-blue-700 transition duration-150 shadow-md">
                Start a New Quiz
            </button>
        </div>
    </div>

    <script type="module">
        // Gemini API Configuration
        const MODEL_NAME = "gemini-2.5-flash-preview-09-2025";
        const API_KEY = ""; // Canvas will provide this at runtime

        const inputSection = document.getElementById('input-section');
        const loadingIndicator = document.getElementById('loading-indicator');
        const errorContainer = document.getElementById('error-message');
        const errorText = document.getElementById('error-text');
        const quizSection = document.getElementById('quiz-section');
        const questionsContainer = document.getElementById('questions-container');
        const submitButton = document.getElementById('submit-button');
        const resultsSection = document.getElementById('results-section');
        const scoreText = document.getElementById('score-text');
        const feedbackText = document.getElementById('feedback-text');
        const topicInput = document.getElementById('topic-input');
        const generateButton = document.getElementById('generate-button');
        const resetButton = document.getElementById('reset-button');
        const quizTopicTitle = document.getElementById('quiz-topic-title');

        let quizData = [];
        let userAnswers = {};

        // --- Utility Functions ---

        /**
         * Hides all major application sections.
         */
        function hideAllSections() {
            inputSection.classList.add('hidden');
            loadingIndicator.classList.add('hidden');
            quizSection.classList.add('hidden');
            resultsSection.classList.add('hidden');
            errorContainer.classList.add('hidden');
        }

        /**
         * Retries the API call using exponential backoff.
         */
        async function fetchWithBackoff(url, options, retries = 3, delay = 1000) {
            for (let i = 0; i < retries; i++) {
                try {
                    const response = await fetch(url, options);
                    if (response.ok) {
                        return response;
                    }
                    if (response.status === 429 && i < retries - 1) { // 429: Too Many Requests
                        await new Promise(resolve => setTimeout(resolve, delay));
                        delay *= 2; // Exponential backoff
                        continue;
                    }
                    throw new Error(API failed with status: ${response.status} ${response.statusText});
                } catch (error) {
                    if (i === retries - 1) {
                        throw error;
                    }
                    await new Promise(resolve => setTimeout(resolve, delay));
                    delay *= 2;
                }
            }
        }
        
        // --- Quiz Rendering and Interaction ---

        /**
         * Renders the quiz questions and options into the UI.
         */
        function renderQuiz() {
            questionsContainer.innerHTML = '';
            userAnswers = {}; // Reset answers

            quizData.forEach((q, qIndex) => {
                const qCard = document.createElement('div');
                qCard.classList.add('card', 'mb-4');
                qCard.innerHTML = <p class="font-semibold text-lg mb-3 text-gray-800">Q${qIndex + 1}: ${q.question}</p>;

                const optionsDiv = document.createElement('div');
                q.options.forEach((option, oIndex) => {
                    const optionBtn = document.createElement('button');
                    optionBtn.classList.add('option-button');
                    optionBtn.textContent = ${String.fromCharCode(65 + oIndex)}. ${option};
                    optionBtn.dataset.qIndex = qIndex;
                    optionBtn.dataset.oIndex = option; // Store the option text for comparison

                    optionBtn.addEventListener('click', () => {
                        // Deselect any previously selected option for this question
                        const prevSelected = optionsDiv.querySelector('.option-button.selected');
                        if (prevSelected) {
                            prevSelected.classList.remove('selected', 'bg-blue-100', 'border-blue-500');
                        }
                        
                        // Select the current option
                        optionBtn.classList.add('selected', 'bg-blue-100', 'border-blue-500');
                        userAnswers[qIndex] = option; // Store the selected option text
                    });
                    optionsDiv.appendChild(optionBtn);
                });

                qCard.appendChild(optionsDiv);
                questionsContainer.appendChild(qCard);
            });
            hideAllSections();
            quizTopicTitle.textContent = Quiz on: ${topicInput.value};
            quizSection.classList.remove('hidden');
        }

        /**
         * Evaluates the quiz and updates the UI to show results.
         */
        function showResults() {
            let score = 0;
            quizData.forEach((q, qIndex) => {
                const questionDiv = questionsContainer.children[qIndex];
                const options = questionDiv.querySelectorAll('.option-button');
                const userAnswer = userAnswers[qIndex];

                options.forEach(optionBtn => {
                    optionBtn.disabled = true; // Disable clicks after submission
                    optionBtn.removeEventListener('click', () => {}); // Remove listeners

                    const optionText = optionBtn.dataset.oIndex;
                    
                    // Mark correct answer globally
                    if (optionText === q.correct_answer) {
                        optionBtn.classList.add('correct');
                        optionBtn.classList.remove('selected', 'bg-blue-100', 'border-blue-500'); // Remove blue selection if it was the correct one
                    }

                    // Check if the user selected this option
                    if (optionText === userAnswer) {
                        if (optionText === q.correct_answer) {
                            score++;
                        } else {
                            optionBtn.classList.add('incorrect');
                            optionBtn.classList.remove('selected'); // Keep the incorrect styling
                        }
                    }
                    
                    // If an option was selected but was incorrect, the 'incorrect' class takes precedence
                    // If the user didn't select the correct answer, it will still show up green.
                });
            });

            submitButton.classList.add('disabled-button');
            submitButton.disabled = true;

            hideAllSections();
            resultsSection.classList.remove('hidden');
            scoreText.textContent = ${score}/${quizData.length};
            feedbackText.textContent = getFeedback(score, quizData.length);
        }
        
        /**
         * Provides context-specific feedback based on the score.
         */
        function getFeedback(score, total) {
            const percentage = score / total;
            if (percentage === 1) return "Outstanding! A perfect score. You are a true expert!";
            if (percentage >= 0.8) return "Excellent job! You have a great understanding of the topic.";
            if (percentage >= 0.6) return "Good effort! You're almost there. Keep studying!";
            if (percentage >= 0.4) return "Solid start, but there's room for improvement. Review the highlighted answers!";
            return "Keep going! Review the correct answers to learn the material better.";
        }


        // --- Gemini API Call Logic ---

        async function generateQuiz() {
            const topic = topicInput.value.trim();
            if (!topic) {
                errorText.textContent = "Please enter a topic to generate a quiz.";
                errorContainer.classList.remove('hidden');
                return;
            }

            hideAllSections();
            loadingIndicator.classList.remove('hidden');
            
            // Define the strict JSON schema for the quiz
            const quizSchema = {
                type: "ARRAY",
                description: "A list of 5 multiple-choice questions about the user's topic.",
                items: {
                    type: "OBJECT",
                    properties: {
                        "question": { "type": "STRING", "description": "The quiz question text." },
                        "options": {
                            "type": "ARRAY",
                            "description": "An array of exactly 4 unique multiple-choice options.",
                            "items": { "type": "STRING" }
                        },
                        "correct_answer": {
                            "type": "STRING",
                            "description": "The text of the correct answer, which must exactly match one of the options in the options array."
                        }
                    },
                    required: ["question", "options", "correct_answer"],
                    propertyOrdering: ["question", "options", "correct_answer"]
                }
            };

            // Construct the API payload
            const userQuery = Generate a quiz with exactly 5 multiple-choice questions based on the topic: "${topic}". Ensure each question has exactly 4 distinct options.;
            
            const payload = {
                contents: [{ parts: [{ text: userQuery }] }],
                generationConfig: {
                    responseMimeType: "application/json",
                    responseSchema: quizSchema
                }
            };

            const apiUrl = https://generativelanguage.googleapis.com/v1beta/models/${MODEL_NAME}:generateContent?key=${API_KEY};

            try {
                const response = await fetchWithBackoff(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 &&
                    result.candidates[0].content && result.candidates[0].content.parts &&
                    result.candidates[0].content.parts.length > 0) {
                    
                    const jsonString = result.candidates[0].content.parts[0].text;
                    // The model returns the JSON object as a string. We must parse it.
                    quizData = JSON.parse(jsonString);

                    // Basic validation to ensure we got 5 questions
                    if (Array.isArray(quizData) && quizData.length === 5 && 
                        quizData.every(q => q.question && Array.isArray(q.options) && q.options.length === 4 && q.correct_answer)) {
                        renderQuiz();
                    } else {
                        throw new Error("Invalid or incomplete quiz structure returned by the model.");
                    }
                    
                } else {
                    throw new Error(result.error?.message || "Model failed to generate a response or returned an unexpected format.");
                }

            } catch (e) {
                console.error("Fetch or Parse Error:", e);
                hideAllSections();
                errorText.textContent = [${e.name}]: ${e.message}. Please check your topic and try again.;
                errorContainer.classList.remove('hidden');
                inputSection.classList.remove('hidden');
            }
        }


        // --- Event Listeners ---

        generateButton.addEventListener('click', generateQuiz);
        submitButton.addEventListener('click', showResults);
        
        resetButton.addEventListener('click', () => {
            hideAllSections();
            topicInput.value = '';
            quizData = [];
            userAnswers = {};
            inputSection.classList.remove('hidden');
            submitButton.classList.remove('disabled-button');
            submitButton.disabled = false;
        });
        
        // Initial state
        hideAllSections();
        inputSection.classList.remove('hidden');


    </script>
</body>
</html>
