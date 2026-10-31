"use strict";

document.documentElement.classList.add("js");

const LETTERS = [
  { id: "alif", name: "Алиф", glyph: "ا" },
  { id: "ba", name: "Ба", glyph: "ب" },
  { id: "ta", name: "Та", glyph: "ت" },
  { id: "tha", name: "Са", glyph: "ث" },
  { id: "jim", name: "Джим", glyph: "ج" },
  { id: "ha", name: "Ха", glyph: "ح" },
  { id: "kha", name: "Хо", glyph: "خ" },
  { id: "dal", name: "Даль", glyph: "د" }
];

const STORAGE_KEY = "harf-studied-letters-v1";

function readStudiedLetters() {
  try {
    const saved = JSON.parse(window.localStorage.getItem(STORAGE_KEY) || "[]");
    if (!Array.isArray(saved)) return [];
    return [...new Set(saved.filter((id) => LETTERS.some((letter) => letter.id === id)))];
  } catch (error) {
    return [];
  }
}

function writeStudiedLetters(ids) {
  try {
    window.localStorage.setItem(STORAGE_KEY, JSON.stringify(ids));
  } catch (error) {
    // Progress still updates for the current page when storage is unavailable.
  }
}

let studiedLetters = readStudiedLetters();

function updateProgress() {
  const count = Math.min(studiedLetters.length, 28);
  const countElement = document.querySelector("[data-progress-count]");
  const labelElement = document.querySelector("[data-progress-label]");
  const progressBar = document.querySelector("[data-progressbar]");
  const progressFill = document.querySelector("[data-progress-fill]");

  if (countElement) countElement.textContent = String(count);
  if (labelElement) labelElement.textContent = `Изучено ${count} из 28 букв`;
  if (progressBar) progressBar.setAttribute("aria-valuenow", String(count));
  if (progressFill) progressFill.style.width = `${(count / 28) * 100}%`;
}

function markLetterStudied(id) {
  if (!LETTERS.some((letter) => letter.id === id)) return;
  if (!studiedLetters.includes(id)) {
    studiedLetters = [...studiedLetters, id];
    writeStudiedLetters(studiedLetters);
  }
  updateProgress();
}

function initLetterCards() {
  document.querySelectorAll("[data-letter-card]").forEach((card) => {
    const letterId = card.dataset.letter;
    const letterName = card.dataset.letterName || "буква";
    const front = card.querySelector('[data-face="front"]');
    const back = card.querySelector('[data-face="back"]');

    card.addEventListener("click", () => {
      const isFlipped = card.classList.toggle("is-flipped");
      card.setAttribute("aria-pressed", String(isFlipped));
      card.setAttribute("aria-label", `${isFlipped ? "Скрыть" : "Открыть"} карточку: ${letterName}`);
      if (front) front.setAttribute("aria-hidden", String(isFlipped));
      if (back) back.setAttribute("aria-hidden", String(!isFlipped));
      if (isFlipped) markLetterStudied(letterId);
    });
  });
}

function shuffle(items) {
  const result = [...items];
  for (let index = result.length - 1; index > 0; index -= 1) {
    const swapIndex = Math.floor(Math.random() * (index + 1));
    [result[index], result[swapIndex]] = [result[swapIndex], result[index]];
  }
  return result;
}

function initQuiz() {
  const startButton = document.querySelector("[data-quiz-start]");
  const restartButton = document.querySelector("[data-quiz-restart]");
  const panel = document.querySelector("[data-quiz-panel]");
  const questionView = document.querySelector("[data-quiz-question]");
  const resultView = document.querySelector("[data-quiz-result]");
  const progressLabel = document.querySelector("[data-quiz-progress]");
  const glyphElement = document.querySelector("[data-quiz-glyph]");
  const optionsElement = document.querySelector("[data-quiz-options]");
  const feedbackElement = document.querySelector("[data-quiz-feedback]");
  const nextButton = document.querySelector("[data-quiz-next]");
  const scoreElement = document.querySelector("[data-quiz-score]");

  if (!startButton || !panel || !questionView || !optionsElement || !nextButton) return;

  const roundCount = 5;
  let questions = [];
  let questionIndex = 0;
  let score = 0;
  let answered = false;

  function finishQuiz() {
    questionView.hidden = true;
    resultView.hidden = false;
    if (scoreElement) scoreElement.textContent = `Ваш результат: ${score} из ${roundCount}`;
    startButton.textContent = "Пройти ещё раз";
  }

  function showQuestion() {
    if (questionIndex >= questions.length) {
      finishQuiz();
      return;
    }

    const correct = questions[questionIndex];
    const alternatives = shuffle(LETTERS.filter((letter) => letter.id !== correct.id)).slice(0, 2);
    const options = shuffle([correct, ...alternatives]);

    answered = false;
    if (progressLabel) progressLabel.textContent = `ВОПРОС ${questionIndex + 1} / ${roundCount}`;
    glyphElement.textContent = correct.glyph;
    feedbackElement.textContent = "";
    nextButton.hidden = true;
    questionView.hidden = false;
    resultView.hidden = true;
    optionsElement.replaceChildren();

    options.forEach((option) => {
      const button = document.createElement("button");
      button.type = "button";
      button.className = "quiz-option";
      button.textContent = option.name;
      button.dataset.letter = option.id;
      button.addEventListener("click", () => chooseAnswer(button, option, correct));
      optionsElement.append(button);
    });
  }

  function chooseAnswer(button, choice, correct) {
    if (answered) return;
    answered = true;

    optionsElement.querySelectorAll(".quiz-option").forEach((optionButton) => {
      optionButton.disabled = true;
      if (optionButton.dataset.letter === correct.id) optionButton.classList.add("is-correct");
    });

    if (choice.id === correct.id) {
      score += 1;
      feedbackElement.textContent = "Верно — отличная работа.";
      markLetterStudied(correct.id);
    } else {
      button.classList.add("is-incorrect");
      feedbackElement.textContent = `Пока нет. Правильный ответ: ${correct.name}.`;
    }

    nextButton.textContent = questionIndex + 1 === roundCount ? "Посмотреть результат" : "Следующий вопрос →";
    nextButton.hidden = false;
  }

  function startQuiz() {
    questions = shuffle(LETTERS).slice(0, roundCount);
    questionIndex = 0;
    score = 0;
    panel.hidden = false;
    startButton.textContent = "Перезапустить тест";
    showQuestion();
    panel.scrollIntoView({ behavior: "smooth", block: "nearest" });
  }

  startButton.addEventListener("click", startQuiz);
  if (restartButton) restartButton.addEventListener("click", startQuiz);
  nextButton.addEventListener("click", () => {
    if (!answered) return;
    questionIndex += 1;
    showQuestion();
  });
}

function initAccordions() {
  const items = [...document.querySelectorAll(".accordion-item")];

  items.forEach((item) => {
    const trigger = item.querySelector(".accordion-trigger");
    const panel = item.querySelector(".accordion-panel");
    if (!trigger || !panel) return;

    trigger.addEventListener("click", () => {
      const shouldOpen = trigger.getAttribute("aria-expanded") !== "true";

      if (shouldOpen) {
        items.forEach((otherItem) => {
          if (otherItem === item) return;
          const otherTrigger = otherItem.querySelector(".accordion-trigger");
          const otherPanel = otherItem.querySelector(".accordion-panel");
          if (!otherTrigger || !otherPanel || otherPanel.hidden) return;
          otherTrigger.setAttribute("aria-expanded", "false");
          otherPanel.classList.remove("is-open");
          window.setTimeout(() => {
            if (otherTrigger.getAttribute("aria-expanded") === "false") otherPanel.hidden = true;
          }, 380);
        });

        trigger.setAttribute("aria-expanded", "true");
        panel.hidden = false;
        panel.classList.remove("is-open");
        void panel.offsetHeight;
        window.requestAnimationFrame(() => panel.classList.add("is-open"));
      } else {
        trigger.setAttribute("aria-expanded", "false");
        panel.classList.remove("is-open");
        window.setTimeout(() => {
          if (trigger.getAttribute("aria-expanded") === "false") panel.hidden = true;
        }, 380);
      }
    });
  });
}

function initReveal() {
  const revealItems = document.querySelectorAll("[data-reveal], .accordion-item.reveal");
  const reducedMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches;

  if (reducedMotion || !("IntersectionObserver" in window)) {
    revealItems.forEach((item) => item.classList.add("is-visible"));
    return;
  }

  const observer = new IntersectionObserver((entries, currentObserver) => {
    entries.forEach((entry) => {
      if (!entry.isIntersecting) return;
      entry.target.classList.add("is-visible");
      currentObserver.unobserve(entry.target);
    });
  }, { threshold: 0.12, rootMargin: "0px 0px -35px 0px" });

  revealItems.forEach((item) => observer.observe(item));
}

updateProgress();
initLetterCards();
initQuiz();
initAccordions();
initReveal();
