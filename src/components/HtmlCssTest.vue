<template>
    <div class="test-container">
      <div class="header">
        <h1>HTML & CSS Тест</h1>
        <p>50 вопросов для проверки ваших знаний</p>
      </div>
  
      <!-- Тест -->
      <div v-if="!showResults" class="quiz-container">
        <div class="question active">
          <div class="question-header">
            <div class="question-number">Вопрос {{ currentQuestion + 1 }} из 50</div>
            <div class="progress-bar">
              <div 
                class="progress-fill" 
                :style="{ width: ((currentQuestion + 1) / 50) * 100 + '%' }"
              ></div>
            </div>
          </div>
          
          <div class="question-text">{{ getCurrentQuestion.question }}</div>
          
          <div class="options">
            <div
              v-for="(option, index) in getCurrentQuestion.options"
              :key="index"
              class="option"
              :class="{ selected: answers[currentQuestion] === index }"
              @click="selectOption(index)"
            >
              {{ option }}
            </div>
          </div>
          
          <div class="navigation">
            <button 
              class="btn btn-prev" 
              @click="prevQuestion"
              :disabled="currentQuestion === 0"
            >
              Назад
            </button>
            <button 
              class="btn"
              :class="currentQuestion === 49 ? 'btn-finish' : 'btn-next'"
              @click="currentQuestion === 49 ? finishQuiz() : nextQuestion()"
              :disabled="answers[currentQuestion] === undefined"
            >
              {{ currentQuestion === 49 ? 'Завершить тест' : 'Далее' }}
            </button>
          </div>
        </div>
      </div>
  
      <!-- Результаты -->
      <div v-if="showResults" class="results">
        <div class="score-circle" :class="getScoreClass">
          <span>{{ percentage }}%</span>
        </div>
        <h2>{{ getResultTitle }}</h2>
        <p>{{ getResultDescription }}</p>
        <button class="restart-btn" @click="restartQuiz">
          Пройти тест заново
        </button>
      </div>
    </div>
  </template>
  
  <script>
  import { ref, computed } from 'vue'
  
  export default {
    name: 'HtmlCssTest',
    setup() {
      const currentQuestion = ref(0)
      const answers = ref([])
      const showResults = ref(false)
      const questions = [
          {
            question: "Что означает HTML?",
            options: ["HyperText Markup Language", "Home Tool Markup Language", "Hyperlinks and Text Markup Language", "HyperText Modern Language"],
            correct: 0
          },
          {
            question: "Какой тег используется для создания заголовка первого уровня?",
            options: ["<h1>", "<header>", "<head>", "<title>"],
            correct: 0
          },
          {
            question: "Какой атрибут используется для указания источника изображения?",
            options: ["href", "src", "alt", "link"],
            correct: 1
          },
          {
            question: "Что означает CSS?",
            options: ["Computer Style Sheets", "Creative Style Sheets", "Cascading Style Sheets", "Colorful Style Sheets"],
            correct: 2
          },
          {
            question: "Какой селектор CSS используется для выбора элемента по ID?",
            options: [".", "#", "*", "&"],
            correct: 1
          },
          {
            question: "Какое свойство CSS отвечает за цвет текста?",
            options: ["text-color", "font-color", "color", "text-style"],
            correct: 2
          },
          {
            question: "Какой тег используется для создания неупорядоченного списка?",
            options: ["<ol>", "<ul>", "<li>", "<list>"],
            correct: 1
          },
          {
            question: "Какое значение свойства display делает элемент блочным?",
            options: ["inline", "block", "flex", "grid"],
            correct: 1
          },
          {
            question: "Какой атрибут HTML используется для указания альтернативного текста для изображения?",
            options: ["title", "alt", "src", "desc"],
            correct: 1
          },
          {
            question: "Какое свойство CSS используется для изменения размера шрифта?",
            options: ["font-size", "text-size", "font-weight", "size"],
            correct: 0
          },
          {
            question: "Какой тег используется для создания ссылки?",
            options: ["<link>", "<a>", "<href>", "<url>"],
            correct: 1
          },
          {
            question: "Какое свойство CSS отвечает за отступы внутри элемента?",
            options: ["margin", "padding", "border", "spacing"],
            correct: 1
          },
          {
            question: "Какой DOCTYPE используется для HTML5?",
            options: ["<!DOCTYPE html5>", "<!DOCTYPE HTML>", "<!DOCTYPE html>", "<!DOCTYPE>"],
            correct: 2
          },
          {
            question: "Какое свойство CSS используется для скрытия элемента?",
            options: ["visibility: hidden", "display: none", "opacity: 0", "Все варианты верны"],
            correct: 3
          },
          {
            question: "Какой тег используется для создания таблицы?",
            options: ["<table>", "<tab>", "<tbl>", "<grid>"],
            correct: 0
          },
          {
            question: "Какое свойство CSS отвечает за выравнивание текста?",
            options: ["text-align", "align", "text-position", "position"],
            correct: 0
          },
          {
            question: "Какой атрибут используется для объединения ячеек таблицы по горизонтали?",
            options: ["rowspan", "colspan", "span", "merge"],
            correct: 1
          },
          {
            question: "Какое свойство CSS используется для создания границы?",
            options: ["border", "outline", "edge", "frame"],
            correct: 0
          },
          {
            question: "Какой тег используется для создания формы?",
            options: ["<form>", "<input>", "<field>", "<data>"],
            correct: 0
          },
          {
            question: "Какое свойство CSS отвечает за позиционирование элемента?",
            options: ["position", "location", "place", "align"],
            correct: 0
          },
          {
            question: "Какой тег используется для создания переноса строки?",
            options: ["<break>", "<br>", "<newline>", "<ln>"],
            correct: 1
          },
          {
            question: "Какое значение свойства position делает элемент фиксированным относительно окна браузера?",
            options: ["static", "relative", "absolute", "fixed"],
            correct: 3
          },
          {
            question: "Какой тег используется для выделения важного текста?",
            options: ["<important>", "<strong>", "<bold>", "<em>"],
            correct: 1
          },
          {
            question: "Какое свойство CSS используется для изменения прозрачности элемента?",
            options: ["transparency", "opacity", "alpha", "visible"],
            correct: 1
          },
          {
            question: "Какой атрибут HTML используется для указания языка страницы?",
            options: ["language", "lang", "locale", "country"],
            correct: 1
          },
          {
            question: "Какое свойство CSS отвечает за отступы снаружи элемента?",
            options: ["padding", "margin", "spacing", "gap"],
            correct: 1
          },
          {
            question: "Какой тег используется для создания цитаты?",
            options: ["<quote>", "<blockquote>", "<cite>", "<q>"],
            correct: 1
          },
          {
            question: "Какое свойство CSS используется для изменения типа курсора?",
            options: ["cursor", "pointer", "mouse", "hover"],
            correct: 0
          },
          {
            question: "Какой тег используется для создания поля ввода пароля?",
            options: ["<input type='text'>", "<input type='password'>", "<password>", "<input type='hidden'>"],
            correct: 1
          },
          {
            question: "Какое свойство CSS используется для создания тени у текста?",
            options: ["text-shadow", "box-shadow", "shadow", "text-effect"],
            correct: 0
          },
          {
            question: "Какой селектор CSS выбирает все элементы?",
            options: ["all", "*", "everything", "global"],
            correct: 1
          },
          {
            question: "Какое свойство CSS отвечает за жирность шрифта?",
            options: ["font-weight", "font-bold", "text-weight", "bold"],
            correct: 0
          },
          {
            question: "Какой тег используется для создания области текста?",
            options: ["<input type='text'>", "<textarea>", "<textfield>", "<textbox>"],
            correct: 1
          },
          {
            question: "Какое свойство CSS используется для создания округлых углов?",
            options: ["corner-radius", "border-radius", "round-corner", "radius"],
            correct: 1
          },
          {
            question: "Какой атрибут HTML используется для открытия ссылки в новом окне?",
            options: ["target='_blank'", "new-window='true'", "open='new'", "window='new'"],
            correct: 0
          },
          {
            question: "Какое свойство CSS отвечает за высоту строки?",
            options: ["line-height", "row-height", "text-height", "font-height"],
            correct: 0
          },
          {
            question: "Какой тег используется для создания выпадающего списка?",
            options: ["<dropdown>", "<select>", "<list>", "<option>"],
            correct: 1
          },
          {
            question: "Какое свойство CSS используется для изменения порядка flex-элементов?",
            options: ["flex-order", "order", "position", "index"],
            correct: 1
          },
          {
            question: "Какой тег используется для группировки элементов формы?",
            options: ["<group>", "<fieldset>", "<section>", "<div>"],
            correct: 1
          },
          {
            question: "Какое свойство CSS отвечает за направление flex-контейнера?",
            options: ["flex-direction", "direction", "flex-flow", "orientation"],
            correct: 0
          },
          {
            question: "Какой атрибут HTML делает поле ввода обязательным для заполнения?",
            options: ["mandatory", "required", "must-fill", "obligatory"],
            correct: 1
          },
          {
            question: "Какое свойство CSS используется для выравнивания элементов по главной оси в flex?",
            options: ["align-items", "justify-content", "flex-align", "content-align"],
            correct: 1
          },
          {
            question: "Какой тег используется для создания заголовка таблицы?",
            options: ["<th>", "<thead>", "<header>", "<title>"],
            correct: 0
          },
          {
            question: "Какое свойство CSS отвечает за переполнение содержимого?",
            options: ["overflow", "content-overflow", "text-overflow", "scroll"],
            correct: 0
          },
          {
            question: "Какой тег используется для создания встроенного фрейма?",
            options: ["<frame>", "<iframe>", "<embed>", "<object>"],
            correct: 1
          },
          {
            question: "Какое свойство CSS используется для создания столбцов текста?",
            options: ["columns", "column-count", "text-columns", "multi-column"],
            correct: 1
          },
          {
            question: "Какой псевдокласс CSS применяется при наведении мыши?",
            options: [":hover", ":focus", ":active", ":visited"],
            correct: 0
          },
          {
            question: "Какое свойство CSS отвечает за интервал между буквами?",
            options: ["letter-spacing", "char-spacing", "text-spacing", "font-spacing"],
            correct: 0
          },
          {
            question: "Какой тег используется для создания горизонтальной линии?",
            options: ["<line>", "<hr>", "<hline>", "<horizontal>"],
            correct: 1
          },
          {
            question: "Какое свойство CSS используется для изменения стиля списка?",
            options: ["list-style", "list-type", "marker-style", "bullet-style"],
            correct: 0
          }
        ]
  
      const getCurrentQuestion = computed(() => {
        return questions[currentQuestion.value] || {};
      })
  
      const score = computed(() => {
        let correctAnswers = 0;
        for (let i = 0; i < questions.length; i++) {
          if (answers.value[i] === questions[i].correct) {
            correctAnswers++;
          }
        }
        return correctAnswers;
      })
  
      const percentage = computed(() => {
        return Math.round((score.value / questions.length) * 100);
      })
  
      const getScoreClass = computed(() => {
        if (percentage.value >= 90) return 'score-excellent';
        if (percentage.value >= 70) return 'score-good';
        if (percentage.value >= 50) return 'score-average';
        return 'score-poor';
      })
  
      const getResultTitle = computed(() => {
        if (percentage.value >= 90) return 'Превосходно!';
        if (percentage.value >= 70) return 'Хорошо!';
        if (percentage.value >= 50) return 'Удовлетворительно';
        return 'Нужно учиться';
      })
  
      const getResultDescription = computed(() => {
        const baseText = `Вы ответили правильно на ${score.value} из ${questions.length} вопросов.`;
        
        if (percentage.value >= 90) {
          return `${baseText} У вас отличные знания HTML и CSS!`;
        } else if (percentage.value >= 70) {
          return `${baseText} Хорошие знания, но есть куда стремиться!`;
        } else if (percentage.value >= 50) {
          return `${baseText} Базовые знания есть, но нужно подтянуть теорию.`;
        } else {
          return `${baseText} Рекомендуем изучить основы HTML и CSS.`;
        }
      })
  
      const selectOption = (index) => {
        answers.value[currentQuestion.value] = index
      }
  
      const nextQuestion = () => {
        if (answers.value[currentQuestion.value] !== undefined && currentQuestion.value < 49) {
          currentQuestion.value++
        }
      }
  
      const prevQuestion = () => {
        if (currentQuestion.value > 0) {
          currentQuestion.value--
        }
      }
  
      const finishQuiz = () => {
        if (answers.value[currentQuestion.value] !== undefined) {
          showResults.value = true
        }
      }
  
            const testShuffle = () => {
          console.log('Текущий первый вопрос:', questions.value[0].question)
          questions.value = createShuffledQuestions()
          console.log('Новый первый вопрос:', questions.value[0].question)
        }
  
        const restartQuiz = () => {
        currentQuestion.value = 0
        answers.value = []
        showResults.value = false
      }
  
      return {
        currentQuestion,
        answers,
        showResults,
        getCurrentQuestion,
        score,
        percentage,
        getScoreClass,
        getResultTitle,
        getResultDescription,
        selectOption,
        nextQuestion,
        prevQuestion,
        finishQuiz,
        restartQuiz
      }
    }
  }
  </script>
  
  <style scoped>
  .test-container {
    max-width: 800px;
    margin: 0 auto;
    background: white;
    border-radius: 15px;
    box-shadow: 0 20px 40px rgba(0,0,0,0.1);
    overflow: hidden;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  }
  
  .header {
    background: linear-gradient(45deg, #2196F3, #21CBF3);
    color: white;
    text-align: center;
    padding: 30px;
  }
  
  .header h1 {
    font-size: 2.5em;
    margin-bottom: 10px;
    margin: 0 0 10px 0;
  }
  
  .header p {
    font-size: 1.1em;
    opacity: 0.9;
    margin: 0;
  }
  
  .quiz-container {
    padding: 40px;
  }
  
  .question {
    margin-bottom: 30px;
    animation: fadeIn 0.5s ease-in-out;
  }
  
  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(20px); }
    to { opacity: 1; transform: translateY(0); }
  }
  
  .question-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    padding-bottom: 15px;
    border-bottom: 2px solid #f0f0f0;
  }
  
  .question-number {
    background: #2196F3;
    color: white;
    padding: 8px 16px;
    border-radius: 20px;
    font-weight: bold;
  }
  
  .progress-bar {
    flex: 1;
    margin: 0 20px;
    height: 8px;
    background: #f0f0f0;
    border-radius: 4px;
    overflow: hidden;
  }
  
  .progress-fill {
    height: 100%;
    background: linear-gradient(45deg, #2196F3, #21CBF3);
    border-radius: 4px;
    transition: width 0.3s ease;
  }
  
  .question-text {
    font-size: 1.3em;
    font-weight: 600;
    color: #333;
    margin-bottom: 25px;
    line-height: 1.5;
  }
  
  .options {
    display: grid;
    gap: 15px;
  }
  
  .option {
    background: #f8f9fa;
    border: 2px solid transparent;
    border-radius: 10px;
    padding: 15px 20px;
    cursor: pointer;
    transition: all 0.3s ease;
    font-size: 1.1em;
  }
  
  .option:hover {
    background: #e3f2fd;
    border-color: #2196F3;
    transform: translateY(-2px);
  }
  
  .option.selected {
    background: #2196F3;
    color: white;
    border-color: #1976D2;
  }
  
  .navigation {
    display: flex;
    justify-content: space-between;
    margin-top: 40px;
    padding-top: 20px;
    border-top: 2px solid #f0f0f0;
  }
  
  .btn {
    padding: 12px 24px;
    border: none;
    border-radius: 8px;
    font-size: 1.1em;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
  }
  
  .btn-prev {
    background: #6c757d;
    color: white;
  }
  
  .btn-prev:hover:not(:disabled) {
    background: #5a6268;
  }
  
  .btn-next, .btn-finish {
    background: #28a745;
    color: white;
  }
  
  .btn-next:hover:not(:disabled), .btn-finish:hover:not(:disabled) {
    background: #218838;
  }
  
  .btn:disabled {
    background: #ccc;
    cursor: not-allowed;
  }
  
  .results {
    text-align: center;
    padding: 40px;
    animation: fadeIn 0.5s ease-in-out;
  }
  
  .score-circle {
    width: 200px;
    height: 200px;
    border-radius: 50%;
    margin: 0 auto 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 3em;
    font-weight: bold;
    color: white;
  }
  
  .score-excellent {
    background: linear-gradient(45deg, #28a745, #34ce57);
  }
  
  .score-good {
    background: linear-gradient(45deg, #ffc107, #ffcd39);
  }
  
  .score-average {
    background: linear-gradient(45deg, #fd7e14, #ff8c42);
  }
  
  .score-poor {
    background: linear-gradient(45deg, #dc3545, #e55353);
  }
  
  .results h2 {
    font-size: 2.5em;
    margin-bottom: 20px;
    color: #333;
  }
  
  .results p {
    font-size: 1.2em;
    color: #666;
    margin-bottom: 30px;
  }
  
  .restart-btn {
    background: #007bff;
    color: white;
    padding: 15px 30px;
    border: none;
    border-radius: 8px;
    font-size: 1.2em;
    cursor: pointer;
    transition: background 0.3s ease;
  }
  
  .restart-btn:hover {
    background: #0056b3;
  }
  </style>