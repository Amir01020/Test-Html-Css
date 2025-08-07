<template>
  <!-- Регистрация -->
  <div v-if="showRegistration" class="registration-overlay">
    <div class="registration-form">
      <div class="registration-icon">📚</div>
      <h1>Тестирование знаний</h1>
      <p>Добро пожаловать! Пожалуйста, введите ваши данные для начала тестирования</p>
      
      <div class="form-group">
        <input 
          v-model="userInfo.firstName"
          type="text" 
          placeholder="Имя" 
          class="form-input"
          @keyup.enter="handleRegistration"
        />
      </div>
      
      <div class="form-group">
        <input 
          v-model="userInfo.lastName"
          type="text" 
          placeholder="Фамилия" 
          class="form-input"
          @keyup.enter="handleRegistration"
        />
      </div>
      
      <button 
        @click="handleRegistration" 
        class="start-btn"
        :disabled="!canStartTest"
      >
        Далее
      </button>
    </div>
  </div>

  <!-- Выбор теста -->
  <div v-if="showTestSelection" class="test-selection-overlay">
    <div class="test-selection-form">
      <h2>Выберите тест</h2>
      <p>Привет, {{ userInfo.firstName }}! Какой тест вы хотите пройти?</p>
      
      <div class="test-options">
        <div class="test-option" @click="selectTest('html')">
          <div class="test-icon">🏷️</div>
          <h3>HTML</h3>
          <p>30 вопросов по основам HTML</p>
        </div>
        
        <div class="test-option" @click="selectTest('htmlcss')">
          <div class="test-icon">🎨</div>
          <h3>HTML & CSS</h3>
          <p>50 вопросов по HTML и CSS</p>
        </div>
        
        <div class="test-option" @click="selectTest('python')">
          <div class="test-icon">🐍</div>
          <h3>Python</h3>
          <p>40 вопросов по основам Python</p>
        </div>
      </div>
    </div>
  </div>

  <!-- Основной тест -->
  <div v-else-if="!showRegistration && !showTestSelection" class="test-container">
    <div class="header">
      <h1>{{ testTitle }}</h1>
      <p>Привет, {{ userInfo.firstName }}! {{ testDescription }}</p>
    </div>

    <!-- Тест -->
    <div v-if="!showResults" class="quiz-container">
      <div class="question active">
        <div class="question-header">
          <div class="question-number">Вопрос {{ currentQuestion + 1 }} из {{ totalQuestions }}</div>
          <div class="progress-bar">
            <div 
              class="progress-fill" 
              :style="{ width: ((currentQuestion + 1) / totalQuestions) * 100 + '%' }"
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
            :class="currentQuestion === (totalQuestions - 1) ? 'btn-finish' : 'btn-next'"
            @click="currentQuestion === (totalQuestions - 1) ? finishQuiz() : nextQuestion()"
            :disabled="answers[currentQuestion] === undefined || isSubmitting"
          >
            {{ isSubmitting ? 'Отправка...' : (currentQuestion === (totalQuestions - 1) ? 'Завершить тест' : 'Далее') }}
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
      <div class="result-info">
        <p><strong>Участник:</strong> {{ userInfo.firstName }} {{ userInfo.lastName }}</p>
        <p><strong>Тест:</strong> {{ testTitle }}</p>
        <p><strong>Дата:</strong> {{ testDate }}</p>
      </div>
      <button class="restart-btn" @click="restartQuiz">
        Пройти тест заново
      </button>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  name: 'TestingComponent',
  setup() {
    const currentQuestion = ref(0)
    const answers = ref([])
    const showResults = ref(false)
    const showRegistration = ref(true)
    const showTestSelection = ref(false)
    const isSubmitting = ref(false)
    const userInfo = ref({ firstName: '', lastName: '' })
    const shuffledQuestions = ref([])
    const testDate = ref('')
    const selectedTestType = ref('')

    // Telegram Bot Configuration - ЗАМЕНИТЕ НА ВАШИ ДАННЫЕ
    const TELEGRAM_BOT_TOKEN = '7094810717:AAEZd01lcA-Vla4_Lf60Nxg_8foKvwcbkv8' // Замените на ваш токен бота
    const TELEGRAM_CHAT_ID = '6873895827' // Замените на ваш chat ID

    const htmlQuestions = [
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
        question: "Какой тег используется для создания неупорядоченного списка?",
        options: ["<ol>", "<ul>", "<li>", "<list>"],
        correct: 1
      },
      {
        question: "Какой атрибут HTML используется для указания альтернативного текста для изображения?",
        options: ["title", "alt", "src", "desc"],
        correct: 1
      },
      {
        question: "Какой тег используется для создания ссылки?",
        options: ["<link>", "<a>", "<href>", "<url>"],
        correct: 1
      },
      {
        question: "Какой DOCTYPE используется для HTML5?",
        options: ["<!DOCTYPE html5>", "<!DOCTYPE HTML>", "<!DOCTYPE html>", "<!DOCTYPE>"],
        correct: 2
      },
      {
        question: "Какой тег используется для создания таблицы?",
        options: ["<table>", "<tab>", "<tbl>", "<grid>"],
        correct: 0
      },
      {
        question: "Какой атрибут используется для объединения ячеек таблицы по горизонтали?",
        options: ["rowspan", "colspan", "span", "merge"],
        correct: 1
      },
      {
        question: "Какой тег используется для создания формы?",
        options: ["<form>", "<input>", "<field>", "<data>"],
        correct: 0
      },
      {
        question: "Какой тег используется для создания переноса строки?",
        options: ["<break>", "<br>", "<newline>", "<ln>"],
        correct: 1
      },
      {
        question: "Какой тег используется для выделения важного текста?",
        options: ["<important>", "<strong>", "<bold>", "<em>"],
        correct: 1
      },
      {
        question: "Какой атрибут HTML используется для указания языка страницы?",
        options: ["language", "lang", "locale", "country"],
        correct: 1
      },
      {
        question: "Какой тег используется для создания цитаты?",
        options: ["<quote>", "<blockquote>", "<cite>", "<q>"],
        correct: 1
      },
      {
        question: "Какой тег используется для создания поля ввода пароля?",
        options: ["<input type='text'>", "<input type='password'>", "<password>", "<input type='hidden'>"],
        correct: 1
      },
      {
        question: "Какой тег используется для создания области текста?",
        options: ["<input type='text'>", "<textarea>", "<textfield>", "<textbox>"],
        correct: 1
      },
      {
        question: "Какой атрибут HTML используется для открытия ссылки в новом окне?",
        options: ["target='_blank'", "new-window='true'", "open='new'", "window='new'"],
        correct: 0
      },
      {
        question: "Какой тег используется для создания выпадающего списка?",
        options: ["<dropdown>", "<select>", "<list>", "<option>"],
        correct: 1
      },
      {
        question: "Какой тег используется для группировки элементов формы?",
        options: ["<group>", "<fieldset>", "<section>", "<div>"],
        correct: 1
      },
      {
        question: "Какой атрибут HTML делает поле ввода обязательным для заполнения?",
        options: ["mandatory", "required", "must-fill", "obligatory"],
        correct: 1
      },
      {
        question: "Какой тег используется для создания заголовка таблицы?",
        options: ["<th>", "<thead>", "<header>", "<title>"],
        correct: 0
      },
      {
        question: "Какой тег используется для создания встроенного фрейма?",
        options: ["<frame>", "<iframe>", "<embed>", "<object>"],
        correct: 1
      },
      {
        question: "Какой тег используется для создания горизонтальной линии?",
        options: ["<line>", "<hr>", "<hline>", "<horizontal>"],
        correct: 1
      },
      {
        question: "Какой тег используется для создания абзаца?",
        options: ["<paragraph>", "<p>", "<para>", "<text>"],
        correct: 1
      },
      {
        question: "Какой атрибут используется для указания уникального идентификатора элемента?",
        options: ["class", "id", "name", "unique"],
        correct: 1
      },
      {
        question: "Какой тег используется для создания курсивного текста?",
        options: ["<italic>", "<i>", "<em>", "<cursive>"],
        correct: 1
      },
      {
        question: "Какой тег используется для создания подчеркнутого текста?",
        options: ["<underline>", "<u>", "<under>", "<line>"],
        correct: 1
      },
      {
        question: "Какой атрибут используется для указания заголовка элемента (всплывающая подсказка)?",
        options: ["tooltip", "title", "hint", "description"],
        correct: 1
      },
      {
        question: "Какой тег используется для создания упорядоченного списка?",
        options: ["<ul>", "<ol>", "<list>", "<ordered>"],
        correct: 1
      },
      {
        question: "Какой тег используется для указания метаданных HTML-документа?",
        options: ["<metadata>", "<meta>", "<info>", "<data>"],
        correct: 1
      }
    ]

    const htmlCssQuestions = [
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

    const pythonQuestions = [
      {
        question: "Что такое Python?",
        options: ["Язык программирования", "Текстовый редактор", "Операционная система", "База данных"],
        correct: 0
      },
      {
        question: "Какой символ используется для комментариев в Python?",
        options: ["//", "/*", "#", "<!--"],
        correct: 2
      },
      {
        question: "Какая функция используется для вывода текста в Python?",
        options: ["echo", "printf", "print", "write"],
        correct: 2
      },
      {
        question: "Какой тип данных представляет целые числа в Python?",
        options: ["int", "integer", "number", "digit"],
        correct: 0
      },
      {
        question: "Как создать список в Python?",
        options: ["list = (1, 2, 3)", "list = [1, 2, 3]", "list = {1, 2, 3}", "list = 1, 2, 3"],
        correct: 1
      },
      {
        question: "Какой оператор используется для возведения в степень в Python?",
        options: ["^", "**", "pow", "^2"],
        correct: 1
      },
      {
        question: "Как получить длину списка в Python?",
        options: ["length(list)", "size(list)", "len(list)", "count(list)"],
        correct: 2
      },
      {
        question: "Какая функция преобразует строку в число в Python?",
        options: ["int()", "number()", "convert()", "parse()"],
        correct: 0
      },
      {
        question: "Какой символ используется для объединения строк в Python?",
        options: ["&", "+", ".", "||"],
        correct: 1
      },
      {
        question: "Как проверить тип переменной в Python?",
        options: ["typeof()", "type()", "gettype()", "vartype()"],
        correct: 1
      },
      {
        question: "Какое ключевое слово используется для определения функции в Python?",
        options: ["function", "def", "func", "define"],
        correct: 1
      },
      {
        question: "Какой цикл используется для перебора элементов в Python?",
        options: ["foreach", "for", "while", "loop"],
        correct: 1
      },
      {
        question: "Как добавить элемент в конец списка в Python?",
        options: ["append()", "add()", "insert()", "push()"],
        correct: 0
      },
      {
        question: "Какое ключевое слово используется для условных операторов в Python?",
        options: ["if", "when", "condition", "check"],
        correct: 0
      },
      {
        question: "Как создать словарь в Python?",
        options: ["dict = [key: value]", "dict = (key: value)", "dict = {key: value}", "dict = <key: value>"],
        correct: 2
      },
      {
        question: "Какой оператор используется для проверки принадлежности элемента в Python?",
        options: ["in", "contains", "exists", "has"],
        correct: 0
      },
      {
        question: "Как получить символ строки по индексу в Python?",
        options: ["string.charAt(index)", "string[index]", "string.get(index)", "string(index)"],
        correct: 1
      },
      {
        question: "Какая функция возвращает случайное число в Python?",
        options: ["random()", "rand()", "rnd()", "random.random()"],
        correct: 3
      },
      {
        question: "Как импортировать модуль в Python?",
        options: ["include", "import", "require", "use"],
        correct: 1
      },
      {
        question: "Какой метод используется для разделения строки в Python?",
        options: ["split()", "divide()", "separate()", "break()"],
        correct: 0
      },
      {
        question: "Как удалить элемент из списка по значению в Python?",
        options: ["delete()", "remove()", "pop()", "clear()"],
        correct: 1
      },
      {
        question: "Какое значение возвращает функция, если не указан return в Python?",
        options: ["0", "False", "None", "null"],
        correct: 2
      },
      {
        question: "Как создать кортеж в Python?",
        options: ["tuple = [1, 2, 3]", "tuple = (1, 2, 3)", "tuple = {1, 2, 3}", "tuple = 1, 2, 3"],
        correct: 1
      },
      {
        question: "Какой метод преобразует все символы строки в верхний регистр в Python?",
        options: ["uppercase()", "upper()", "toUpper()", "capitalize()"],
        correct: 1
      },
      {
        question: "Как проверить, является ли строка числом в Python?",
        options: ["isdigit()", "isnumber()", "isnumeric()", "Все варианты верны"],
        correct: 3
      },
      {
        question: "Какая функция открывает файл в Python?",
        options: ["open()", "file()", "read()", "load()"],
        correct: 0
      },
      {
        question: "Какой оператор используется для целочисленного деления в Python?",
        options: ["/", "//", "div", "\\"],
        correct: 1
      },
      {
        question: "Как получить остаток от деления в Python?",
        options: ["mod", "%", "remainder", "//"],
        correct: 1
      },
      {
        question: "Какое ключевое слово используется для обработки исключений в Python?",
        options: ["try", "catch", "exception", "error"],
        correct: 0
      },
      {
        question: "Как создать множество в Python?",
        options: ["set = [1, 2, 3]", "set = (1, 2, 3)", "set = {1, 2, 3}", "set = <1, 2, 3>"],
        correct: 2
      },
      {
        question: "Какой метод используется для добавления элемента в множество в Python?",
        options: ["append()", "add()", "insert()", "put()"],
        correct: 1
      },
      {
        question: "Что делает функция range() в Python?",
        options: ["Создает список чисел", "Создает строку", "Создает итератор чисел", "Создает словарь"],
        correct: 2
      },
      {
        question: "Какой метод используется для получения всех ключей словаря в Python?",
        options: ["keys()", "getKeys()", "allKeys()", "keyList()"],
        correct: 0
      },
      {
        question: "Как проверить, пуст ли список в Python?",
        options: ["if list == []:", "if len(list) == 0:", "if not list:", "Все варианты верны"],
        correct: 3
      },
      {
        question: "Какое ключевое слово используется для создания класса в Python?",
        options: ["class", "object", "def", "create"],
        correct: 0
      },
      {
        question: "Какой метод используется для сортировки списка в Python?",
        options: ["sort()", "order()", "arrange()", "organize()"],
        correct: 0
      },
      {
        question: "Как получить максимальное значение из списка в Python?",
        options: ["maximum()", "max()", "largest()", "top()"],
        correct: 1
      },
      {
        question: "Какая функция возвращает абсолютное значение числа в Python?",
        options: ["absolute()", "abs()", "positive()", "unsigned()"],
        correct: 1
      },
      {
        question: "Как объединить два списка в Python?",
        options: ["list1.merge(list2)", "list1 + list2", "list1.append(list2)", "list1.join(list2)"],
        correct: 1
      },
      {
        question: "Какой символ используется для экранирования специальных символов в строке Python?",
        options: ["/", "\\", "^", "&"],
        correct: 1
      },
      {
        question: "Какая функция преобразует число в строку в Python?",
        options: ["string()", "str()", "toString()", "convert()"],
        correct: 1
      }
    ]

    // Функция перемешивания массива
    const shuffleArray = (array) => {
      const shuffled = [...array]
      for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]
      }
      return shuffled
    }

    // Computed свойства
    const getCurrentQuestion = computed(() => {
      return shuffledQuestions.value[currentQuestion.value] || {}
    })

    const canStartTest = computed(() => {
      return userInfo.value.firstName.trim() && userInfo.value.lastName.trim()
    })

    const totalQuestions = computed(() => {
      return shuffledQuestions.value.length
    })

    const testTitle = computed(() => {
      if (selectedTestType.value === 'html') return 'HTML Тест'
      if (selectedTestType.value === 'htmlcss') return 'HTML & CSS Тест'
      if (selectedTestType.value === 'python') return 'Python Тест'
      return 'Тест'
    })

    const testDescription = computed(() => {
      let count, subject
      if (selectedTestType.value === 'html') {
        count = '30'
        subject = 'HTML'
      } else if (selectedTestType.value === 'htmlcss') {
        count = '50'
        subject = 'HTML и CSS'
      } else if (selectedTestType.value === 'python') {
        count = '40'
        subject = 'Python'
      }
      return `${count} вопросов для проверки ваших знаний ${subject}`
    })

    const score = computed(() => {
      let correctAnswers = 0
      for (let i = 0; i < shuffledQuestions.value.length; i++) {
        if (answers.value[i] === shuffledQuestions.value[i].correct) {
          correctAnswers++
        }
      }
      return correctAnswers
    })

    const percentage = computed(() => {
      return Math.round((score.value / shuffledQuestions.value.length) * 100)
    })

    const getScoreClass = computed(() => {
      if (percentage.value >= 90) return 'score-excellent'
      if (percentage.value >= 70) return 'score-good'
      if (percentage.value >= 50) return 'score-average'
      return 'score-poor'
    })

    const getResultTitle = computed(() => {
      if (percentage.value >= 90) return 'Превосходно!'
      if (percentage.value >= 70) return 'Хорошо!'
      if (percentage.value >= 50) return 'Удовлетворительно'
      return 'Нужно учиться'
    })

    const getResultDescription = computed(() => {
      let subject
      if (selectedTestType.value === 'html') subject = 'HTML'
      else if (selectedTestType.value === 'htmlcss') subject = 'HTML и CSS'
      else if (selectedTestType.value === 'python') subject = 'Python'
      
      const baseText = `Вы ответили правильно на ${score.value} из ${shuffledQuestions.value.length} вопросов.`
      
      if (percentage.value >= 90) {
        return `${baseText} У вас отличные знания ${subject}!`
      } else if (percentage.value >= 70) {
        return `${baseText} Хорошие знания, но есть куда стремиться!`
      } else if (percentage.value >= 50) {
        return `${baseText} Базовые знания есть, но нужно подтянуть теорию.`
      } else {
        return `${baseText} Рекомендуем изучить основы ${subject}.`
      }
    })

    // Методы
    const handleRegistration = () => {
      if (canStartTest.value) {
        showRegistration.value = false
        showTestSelection.value = true
      }
    }

    const selectTest = (testType) => {
      selectedTestType.value = testType
      showTestSelection.value = false
      
      if (testType === 'html') {
        shuffledQuestions.value = shuffleArray(htmlQuestions)
      } else if (testType === 'htmlcss') {
        shuffledQuestions.value = shuffleArray(htmlCssQuestions)
      } else if (testType === 'python') {
        shuffledQuestions.value = shuffleArray(pythonQuestions)
      }
    }

    const sendToTelegram = async () => {
      let testName
      if (selectedTestType.value === 'html') testName = 'HTML'
      else if (selectedTestType.value === 'htmlcss') testName = 'HTML & CSS'
      else if (selectedTestType.value === 'python') testName = 'Python'
      
      const message = `
🎯 Результат теста ${testName}

👤 Участник: ${userInfo.value.firstName} ${userInfo.value.lastName}
📊 Результат: ${score.value}/${shuffledQuestions.value.length} (${percentage.value}%)
📈 Оценка: ${getResultTitle.value}

📝 Детали:
${getResultDescription.value}

📅 Дата: ${testDate.value}
      `

      try {
        const response = await fetch(`https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            chat_id: TELEGRAM_CHAT_ID,
            text: message,
            parse_mode: 'HTML'
          })
        })

        if (response.ok) {
          console.log('Результат отправлен в Telegram')
        } else {
          console.error('Ошибка отправки в Telegram')
        }
      } catch (error) {
        console.error('Ошибка:', error)
      }
    }

    const selectOption = (index) => {
      answers.value[currentQuestion.value] = index
    }

    const nextQuestion = () => {
      if (answers.value[currentQuestion.value] !== undefined && currentQuestion.value < shuffledQuestions.value.length - 1) {
        currentQuestion.value++
      }
    }

    const prevQuestion = () => {
      if (currentQuestion.value > 0) {
        currentQuestion.value--
      }
    }

    const finishQuiz = async () => {
      if (answers.value[currentQuestion.value] !== undefined) {
        isSubmitting.value = true
        testDate.value = new Date().toLocaleString('ru-RU')
        showResults.value = true
        await sendToTelegram()
        isSubmitting.value = false
      }
    }

    const restartQuiz = () => {
      currentQuestion.value = 0
      answers.value = []
      showResults.value = false
      showRegistration.value = true
      showTestSelection.value = false
      userInfo.value = { firstName: '', lastName: '' }
      selectedTestType.value = ''
      shuffledQuestions.value = []
    }

    return {
      currentQuestion,
      answers,
      showResults,
      showRegistration,
      showTestSelection,
      isSubmitting,
      userInfo,
      testDate,
      selectedTestType,
      getCurrentQuestion,
      canStartTest,
      totalQuestions,
      testTitle,
      testDescription,
      score,
      percentage,
      getScoreClass,
      getResultTitle,
      getResultDescription,
      handleRegistration,
      selectTest,
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
/* Регистрация */
.registration-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  z-index: 1000;
}

.registration-form {
  background: white;
  border-radius: 20px;
  padding: 40px;
  box-shadow: 0 25px 50px rgba(0,0,0,0.3);
  max-width: 500px;
  width: 90%;
  text-align: center;
}

.registration-icon {
  width: 80px;
  height: 80px;
  background: linear-gradient(45deg, #2196F3, #21CBF3);
  border-radius: 50%;
  margin: 0 auto 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2em;
}

.registration-form h1 {
  font-size: 2.5em;
  margin-bottom: 10px;
  color: #333;
  font-weight: bold;
}

.registration-form p {
  font-size: 1.2em;
  color: #666;
  margin-bottom: 40px;
}

.form-group {
  margin-bottom: 25px;
}

.form-input {
  width: 100%;
  padding: 15px 20px;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  font-size: 1.1em;
  outline: none;
  transition: border-color 0.3s;
  box-sizing: border-box;
}

.form-input:focus {
  border-color: #2196F3;
}

.start-btn {
  background: linear-gradient(45deg, #2196F3, #21CBF3);
  color: white;
  border: none;
  padding: 15px 40px;
  border-radius: 10px;
  font-size: 1.2em;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  width: 100%;
}

.start-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(33, 150, 243, 0.3);
}

.start-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* Выбор теста */
.test-selection-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  z-index: 1000;
}

.test-selection-form {
  background: white;
  border-radius: 20px;
  padding: 40px;
  box-shadow: 0 25px 50px rgba(0,0,0,0.3);
  max-width: 800px;
  width: 90%;
  text-align: center;
}

.test-selection-form h2 {
  font-size: 2.2em;
  margin-bottom: 10px;
  color: #333;
  font-weight: bold;
}

.test-selection-form p {
  font-size: 1.2em;
  color: #666;
  margin-bottom: 40px;
}

.test-options {
  display: flex;
  gap: 30px;
  justify-content: center;
  flex-wrap: wrap;
}

.test-option {
  background: #f8f9fa;
  border: 3px solid transparent;
  border-radius: 15px;
  padding: 30px 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 200px;
  text-align: center;
  flex: 1;
  max-width: 250px;
}

.test-option:hover {
  border-color: #2196F3;
  transform: translateY(-5px);
  box-shadow: 0 15px 30px rgba(33, 150, 243, 0.2);
}

.test-icon {
  font-size: 3em;
  margin-bottom: 15px;
}

.test-option h3 {
  font-size: 1.5em;
  margin-bottom: 10px;
  color: #333;
}

.test-option p {
  color: #666;
  margin: 0;
  font-size: 1em;
}

/* Основной тест */
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

/* Результаты */
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

.result-info {
  background: #f8f9fa;
  border-radius: 10px;
  padding: 20px;
  margin: 20px 0;
  border-left: 4px solid #2196F3;
}

.result-info p {
  margin: 10px 0;
  font-size: 1.1em;
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

/* Адаптивный дизайн */
@media (max-width: 768px) {
  .test-options {
    flex-direction: column;
    align-items: center;
  }
  
  .test-option {
    min-width: auto;
    width: 100%;
    max-width: 300px;
  }
  
  .question-header {
    flex-direction: column;
    gap: 15px;
  }
  
  .progress-bar {
    margin: 0;
    width: 100%;
  }
  
  .navigation {
    flex-direction: column;
    gap: 15px;
  }
  
  .btn {
    width: 100%;
  }
}
</style>