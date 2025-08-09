<template>
  <div id="app">
    <!-- Email Input -->
    <div v-if="currentStep === 'email'" class="email-container">
      <h1>Добро пожаловать в тест профессиональных навыков!</h1>
      <div class="email-form">
        <h3>Введите ваш email для начала тестирования</h3>
        <input 
          v-model="userEmail" 
          type="email" 
          placeholder="your@email.com"
          required
        />
        <button @click="submitEmail" :disabled="!userEmail || isSubmittingEmail">
          {{ isSubmittingEmail ? 'Проверяем...' : 'Начать тест' }}
        </button>
      </div>
    </div>

    <!-- Profession Selection -->
    <div v-if="currentStep === 'profession'" class="profession-selection">
      <h1>Выберите профессию для тестирования</h1>
      <div class="professions-grid">
        <div
          v-for="profession in professions"
          :key="profession.id"
          class="profession-card"
          @click="selectProfession(profession)"
        >
          <h3>{{ profession.name }}</h3>
          <p>{{ profession.description }}</p>
        </div>
      </div>
    </div>

    <!-- Test Questions -->
    <div v-if="currentStep === 'test'" class="test-container">
      <div class="test-header">
        <h2>Тест: {{ selectedProfession.name }}</h2>
        <div class="timer">
          <span>Время: {{ formatTime(timeLeft) }}</span>
        </div>
        <div class="progress">
          <span>Вопрос {{ currentQuestionIndex + 1 }} из {{ questions.length }}</span>
        </div>
      </div>

      <div v-if="currentQuestion" class="question-container">
        <h3>{{ currentQuestion.question }}</h3>
        <div class="options">
          <div
            v-for="(option, index) in currentQuestion.options"
            :key="index"
            class="option"
            :class="{ selected: selectedAnswer === index }"
            @click="selectAnswer(index)"
          >
            {{ option }}
          </div>
        </div>
        <button 
          @click="submitAnswer" 
          :disabled="selectedAnswer === null || isSubmitting"
          class="submit-btn"
        >
          {{ isSubmitting ? 'Проверяем...' : 'Ответить' }}
        </button>
      </div>
    </div>

    <!-- Results -->
    <div v-if="currentStep === 'results'" class="results-container">
      <h2>Результаты теста</h2>
      <div class="results-summary">
        <p>Правильных ответов: {{ correctAnswers }} из {{ questions.length }}</p>
        <p>Процент правильных ответов: {{ Math.round((correctAnswers / questions.length) * 100) }}%</p>
      </div>
      <div class="question-results">
        <div 
          v-for="(result, index) in questionResults" 
          :key="index"
          class="question-result"
          :class="{ correct: result.isCorrect, incorrect: !result.isCorrect }"
        >
          <h4>Вопрос {{ index + 1 }}</h4>
          <p><strong>Вопрос:</strong> {{ result.question }}</p>
          <p><strong>Ваш ответ:</strong> {{ result.userAnswer }}</p>
          <p><strong>Правильный ответ:</strong> {{ result.correctAnswer }}</p>
          <p v-if="result.explanation"><strong>Объяснение:</strong> {{ result.explanation }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted } from 'vue'
import axios from 'axios'

// State
const currentStep = ref('email')
const selectedProfession = ref(null)
const questions = ref([])
const currentQuestionIndex = ref(0)
const selectedAnswer = ref(null)
const timeLeft = ref(60)
const timer = ref(null)
const isSubmitting = ref(false)
const correctAnswers = ref(0)
const questionResults = ref([])
const userEmail = ref('')
const isSubmittingEmail = ref(false)

// Professions
const professions = ref([
  {
    id: 'frontend',
    name: 'Frontend Developer',
    description: 'Разработка пользовательских интерфейсов с использованием HTML, CSS, JavaScript'
  },
  {
    id: 'backend',
    name: 'Backend Developer',
    description: 'Разработка серверной части приложений, API, баз данных'
  },
  {
    id: 'fullstack',
    name: 'Full Stack Developer',
    description: 'Разработка как клиентской, так и серверной части приложений'
  },
  {
    id: 'mobile',
    name: 'Mobile Developer',
    description: 'Разработка мобильных приложений для iOS и Android'
  },
  {
    id: 'devops',
    name: 'DevOps Engineer',
    description: 'Автоматизация процессов разработки и развертывания'
  }
])

// Computed
const currentQuestion = computed(() => {
  return questions.value[currentQuestionIndex.value] || null
})

// Methods
const selectProfession = (profession) => {
  selectedProfession.value = profession
  generateQuestions(profession.id)
  currentStep.value = 'test'
  startTimer()
}

const generateQuestions = (professionId) => {
  const questionTemplates = {
    frontend: [
      {
        question: "Что такое HTML?",
        options: [
          "Язык программирования",
          "Язык разметки гипертекста",
          "Язык стилей",
          "Язык скриптов"
        ],
        correctAnswer: 1
      },
      {
        question: "Какой тег используется для создания ссылки?",
        options: ["<link>", "<a>", "<href>", "<url>"],
        correctAnswer: 1
      },
      {
        question: "Что такое CSS?",
        options: [
          "Язык программирования",
          "Язык разметки",
          "Каскадные таблицы стилей",
          "Язык скриптов"
        ],
        correctAnswer: 2
      },
      {
        question: "Как изменить цвет текста в CSS?",
        options: [
          "text-color: red;",
          "color: red;",
          "font-color: red;",
          "text: red;"
        ],
        correctAnswer: 1
      },
      {
        question: "Что такое JavaScript?",
        options: [
          "Язык разметки",
          "Язык стилей",
          "Язык программирования",
          "Язык запросов"
        ],
        correctAnswer: 2
      }
    ],
    backend: [
      {
        question: "Что такое API?",
        options: [
          "Язык программирования",
          "Интерфейс прикладного программирования",
          "База данных",
          "Веб-сервер"
        ],
        correctAnswer: 1
      },
      {
        question: "Какой протокол используется для веб-приложений?",
        options: ["FTP", "HTTP", "SMTP", "SSH"],
        correctAnswer: 1
      },
      {
        question: "Что такое REST API?",
        options: [
          "Язык программирования",
          "Архитектурный стиль для веб-сервисов",
          "База данных",
          "Фреймворк"
        ],
        correctAnswer: 1
      },
      {
        question: "Какой метод HTTP используется для создания ресурса?",
        options: ["GET", "POST", "PUT", "DELETE"],
        correctAnswer: 1
      },
      {
        question: "Что такое база данных?",
        options: [
          "Веб-сервер",
          "Структурированная совокупность данных",
          "Язык программирования",
          "Фреймворк"
        ],
        correctAnswer: 1
      }
    ],
    fullstack: [
      {
        question: "Что такое MERN stack?",
        options: [
          "MongoDB, Express, React, Node.js",
          "MySQL, Express, React, Node.js",
          "MongoDB, Express, Ruby, Node.js",
          "MySQL, Express, Ruby, Node.js"
        ],
        correctAnswer: 0
      },
      {
        question: "Что такое MEAN stack?",
        options: [
          "MongoDB, Express, Angular, Node.js",
          "MySQL, Express, Angular, Node.js",
          "MongoDB, Express, Angular, .NET",
          "MySQL, Express, Angular, .NET"
        ],
        correctAnswer: 0
      },
      {
        question: "Что такое JWT?",
        options: [
          "JavaScript Web Token",
          "JSON Web Token",
          "Java Web Token",
          "JSON Web Transfer"
        ],
        correctAnswer: 1
      },
      {
        question: "Что такое CORS?",
        options: [
          "Cross-Origin Resource Sharing",
          "Cross-Origin Request Service",
          "Cross-Origin Resource Service",
          "Cross-Origin Request Sharing"
        ],
        correctAnswer: 0
      },
      {
        question: "Что такое middleware?",
        options: [
          "База данных",
          "Промежуточное ПО",
          "Фронтенд фреймворк",
          "Язык программирования"
        ],
        correctAnswer: 1
      }
    ],
    mobile: [
      {
        question: "Что такое React Native?",
        options: [
          "Фреймворк для веб-разработки",
          "Фреймворк для мобильной разработки",
          "Язык программирования",
          "База данных"
        ],
        correctAnswer: 1
      },
      {
        question: "Что такое Flutter?",
        options: [
          "Фреймворк от Google для мобильной разработки",
          "Фреймворк от Facebook для мобильной разработки",
          "Язык программирования",
          "База данных"
        ],
        correctAnswer: 0
      },
      {
        question: "Что такое Swift?",
        options: [
          "Язык программирования для Android",
          "Язык программирования для iOS",
          "Фреймворк для мобильной разработки",
          "База данных"
        ],
        correctAnswer: 1
      },
      {
        question: "Что такое Kotlin?",
        options: [
          "Язык программирования для iOS",
          "Язык программирования для Android",
          "Фреймворк для мобильной разработки",
          "База данных"
        ],
        correctAnswer: 1
      },
      {
        question: "Что такое APK?",
        options: [
          "Файл установки для iOS",
          "Файл установки для Android",
          "Файл установки для Windows",
          "Файл установки для macOS"
        ],
        correctAnswer: 1
      }
    ],
    devops: [
      {
        question: "Что такое Docker?",
        options: [
          "Операционная система",
          "Платформа для контейнеризации",
          "База данных",
          "Язык программирования"
        ],
        correctAnswer: 1
      },
      {
        question: "Что такое Kubernetes?",
        options: [
          "База данных",
          "Система оркестрации контейнеров",
          "Веб-сервер",
          "Язык программирования"
        ],
        correctAnswer: 1
      },
      {
        question: "Что такое CI/CD?",
        options: [
          "Continuous Integration/Continuous Deployment",
          "Continuous Integration/Continuous Development",
          "Continuous Implementation/Continuous Deployment",
          "Continuous Implementation/Continuous Development"
        ],
        correctAnswer: 0
      },
      {
        question: "Что такое Git?",
        options: [
          "Система управления базами данных",
          "Система контроля версий",
          "Веб-сервер",
          "Язык программирования"
        ],
        correctAnswer: 1
      },
      {
        question: "Что такое Jenkins?",
        options: [
          "База данных",
          "Сервер автоматизации",
          "Веб-сервер",
          "Язык программирования"
        ],
        correctAnswer: 1
      }
    ]
  }

  const templates = questionTemplates[professionId] || questionTemplates.frontend
  questions.value = []
  
  // Generate 10 questions by repeating and modifying templates
  for (let i = 0; i < 10; i++) {
    const template = templates[i % templates.length]
    questions.value.push({
      ...template,
      id: i
    })
  }
}

const startTimer = () => {
  timeLeft.value = 60
  timer.value = setInterval(() => {
    timeLeft.value--
    if (timeLeft.value <= 0) {
      submitAnswer()
    }
  }, 1000)
}

const stopTimer = () => {
  if (timer.value) {
    clearInterval(timer.value)
    timer.value = null
  }
}

const selectAnswer = (index) => {
  selectedAnswer.value = index
}

const submitAnswer = async () => {
  if (selectedAnswer.value === null || isSubmitting.value) return

  isSubmitting.value = true
  stopTimer()

  const currentQ = currentQuestion.value
  const userAnswer = currentQ.options[selectedAnswer.value]
  const correctAnswer = currentQ.options[currentQ.correctAnswer]

  // Check if API key is available
  if (!import.meta.env.VITE_OPENAI_API_KEY) {
    console.error('OpenAI API key not found. Please check your .env file.')
    // Fallback validation
    const isCorrect = selectedAnswer.value === currentQ.correctAnswer
    if (isCorrect) {
      correctAnswers.value++
    }
    questionResults.value.push({
      question: currentQ.question,
      userAnswer,
      correctAnswer,
      isCorrect,
      explanation: isCorrect ? 'Правильный ответ!' : 'Неправильный ответ.'
    })
    isSubmitting.value = false
    selectedAnswer.value = null
    if (currentQuestionIndex.value < questions.value.length - 1) {
      currentQuestionIndex.value++
      startTimer()
    } else {
      submitFinalResults()
    }
    return
  }

  // Call ChatGPT API to validate answer
  try {
    const response = await axios.post('https://api.openai.com/v1/chat/completions', {
      model: 'gpt-3.5-turbo',
      messages: [
        {
          role: 'system',
          content: 'Ты эксперт по программированию. Проверь правильность ответа на вопрос. Верни JSON с полями: isCorrect (boolean), explanation (string)'
        },
        {
          role: 'user',
          content: `Вопрос: ${currentQ.question}\nВарианты ответов: ${currentQ.options.join(', ')}\nПравильный ответ: ${correctAnswer}\nОтвет пользователя: ${userAnswer}\nПроверь, правильный ли ответ дал пользователь.`
        }
      ],
      temperature: 0.3
    }, {
      headers: {
        'Authorization': `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
        'Content-Type': 'application/json'
      }
    })

    const result = response.data.choices[0].message.content
    let isCorrect = false
    let explanation = ''

    try {
      const parsed = JSON.parse(result)
      isCorrect = parsed.isCorrect
      explanation = parsed.explanation
    } catch (e) {
      // Fallback: check if answer matches correct answer
      isCorrect = selectedAnswer.value === currentQ.correctAnswer
      explanation = isCorrect ? 'Правильный ответ!' : 'Неправильный ответ.'
    }

    if (isCorrect) {
      correctAnswers.value++
    }

    questionResults.value.push({
      question: currentQ.question,
      userAnswer,
      correctAnswer,
      isCorrect,
      explanation
    })

  } catch (error) {
    console.error('Error validating answer:', error)
    // Fallback validation
    const isCorrect = selectedAnswer.value === currentQ.correctAnswer
    if (isCorrect) {
      correctAnswers.value++
    }
    questionResults.value.push({
      question: currentQ.question,
      userAnswer,
      correctAnswer,
      isCorrect,
      explanation: isCorrect ? 'Правильный ответ!' : 'Неправильный ответ.'
    })
  }

  isSubmitting.value = false
  selectedAnswer.value = null

  if (currentQuestionIndex.value < questions.value.length - 1) {
    currentQuestionIndex.value++
    startTimer()
  } else {
    // Test completed - submit results to Getform and show results
    submitFinalResults()
  }
}

const submitFinalResults = async () => {
  try {
    // Submit results to Getform
    await axios.post('https://getform.io/f/bllzmwmb', {
      email: userEmail.value,
      profession: selectedProfession.value.name,
      correctAnswers: correctAnswers.value,
      totalQuestions: questions.value.length,
      percentage: Math.round((correctAnswers.value / questions.value.length) * 100),
      results: JSON.stringify(questionResults.value)
    })

    currentStep.value = 'results'
  } catch (error) {
    console.error('Error submitting results:', error)
    // Still show results even if submission fails
    currentStep.value = 'results'
  }
}

const submitEmail = async () => {
  if (!userEmail.value || isSubmittingEmail.value) return

  isSubmittingEmail.value = true

  try {
    // Submit initial email to Getform
    await axios.post('https://getform.io/f/bllzmwmb', {
      email: userEmail.value,
      step: 'started_test',
      timestamp: new Date().toISOString()
    })

    // Move to profession selection
    currentStep.value = 'profession'
  } catch (error) {
    console.error('Error submitting email:', error)
    // Still continue even if submission fails
    currentStep.value = 'profession'
  }

  isSubmittingEmail.value = false
}

const formatTime = (seconds) => {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins}:${secs.toString().padStart(2, '0')}`
}

// Cleanup
onUnmounted(() => {
  stopTimer()
})
</script>

<style scoped>
#app {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.email-container {
  max-width: 600px;
  margin: 0 auto;
  text-align: center;
}

.email-form {
  margin-top: 30px;
}

.email-form input {
  width: 100%;
  padding: 15px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 1.1em;
  margin-bottom: 20px;
}

.email-form button {
  background: #007bff;
  color: white;
  border: none;
  padding: 15px 30px;
  border-radius: 8px;
  font-size: 1.1em;
  cursor: pointer;
  transition: background 0.3s ease;
}

.email-form button:hover:not(:disabled) {
  background: #0056b3;
}

.profession-selection {
  text-align: center;
}

.professions-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
  margin-top: 30px;
}

.profession-card {
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  padding: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.profession-card:hover {
  border-color: #007bff;
  transform: translateY(-5px);
  box-shadow: 0 5px 15px rgba(0,0,0,0.1);
}

.test-container {
  max-width: 800px;
  margin: 0 auto;
}

.test-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 10px;
}

.timer {
  font-size: 1.2em;
  font-weight: bold;
  color: #dc3545;
}

.progress {
  color: #6c757d;
}

.question-container {
  background: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.options {
  margin: 20px 0;
}

.option {
  padding: 15px;
  margin: 10px 0;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.option:hover {
  border-color: #007bff;
  background: #f8f9fa;
}

.option.selected {
  border-color: #007bff;
  background: #007bff;
  color: white;
}

.submit-btn {
  background: #007bff;
  color: white;
  border: none;
  padding: 15px 30px;
  border-radius: 8px;
  font-size: 1.1em;
  cursor: pointer;
  transition: background 0.3s ease;
}

.submit-btn:hover:not(:disabled) {
  background: #0056b3;
}

.submit-btn:disabled {
  background: #6c757d;
  cursor: not-allowed;
}

.results-container {
  max-width: 800px;
  margin: 0 auto;
}

.results-summary {
  background: #f8f9fa;
  padding: 20px;
  border-radius: 10px;
  margin: 20px 0;
}

.question-results {
  margin-top: 30px;
}

.question-result {
  background: white;
  padding: 20px;
  margin: 15px 0;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  border-left: 5px solid #6c757d;
}

.question-result.correct {
  border-left-color: #28a745;
  background: #f8fff9;
}

.question-result.incorrect {
  border-left-color: #dc3545;
  background: #fff8f8;
}

.question-result h4 {
  margin-top: 0;
  color: #333;
}

.question-result p {
  margin: 5px 0;
}

.question-result strong {
  color: #555;
}
</style>
