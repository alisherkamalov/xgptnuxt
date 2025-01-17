<template>
  <TheWarning :opacityWarning="opacityW" :translateWarning="translateW" />
  <div class="blur" :style="{ opacity: blurOpacity }" @click="stopHoldingInCopy">
    <TheBlur />
  </div>
  <div class="container">
    <div class="center-logo" :style="{ opacity: opacitybg }">
      <img src="/logo.png" class="gygole" />
      <div class="shadow-logo"></div>
      Привет!
    </div>

    <!-- Ответы -->
    <div class="answers-column">
      <div v-for="(answer, index) in answers" :key="index" class="answer" :class="answer.class">
        <template v-for="(part, partIndex) in answer.parts" :key="partIndex">
          <!-- Обычный текст -->
          <div v-if="part.type === 'text'" class="text">{{ part.content }}</div>

          <!-- Блок кода -->
          <div v-else-if="part.type === 'code'" class="code-block">
            <pre>{{ part.content }}</pre>
            <v-btn class="copy-code" @click="copyText(part.content)"><v-icon>mdi mdi-content-copy</v-icon></v-btn>
          </div>
        </template>
      </div>
    </div>

    <!-- Поле ввода -->
    <div class="cont-inputs">
      <v-textarea v-model="inputs[0]" outlined dense placeholder="Введите свой вопрос" height="67px" clearable
        class="custom-textarea" />
      <v-button class="send-btn icon" @click="sendAnswer">
        <i class="icon2">
          <svg v-if="isSenttext" class="loading" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512">
            <path fill="#ffffff"
              d="M8 256a56 56 0 1 1 112 0A56 56 0 1 1 8 256zm160 0a56 56 0 1 1 112 0 56 56 0 1 1 -112 0zm216-56a56 56 0 1 1 0 112 56 56 0 1 1 0-112z" />
          </svg>
          <svg v-else xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512">
            <path fill="#ffffff"
              d="M438.6 278.6c12.5-12.5 12.5-32.8 0-45.3l-160-160c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3L338.8 224 32 224c-17.7 0-32 14.3-32 32s14.3 32 32 32l306.7 0L233.4 393.4c-12.5 12.5-12.5 32.8 0 45.3s32.8 12.5 45.3 0l160-160z" />
          </svg>
        </i>
      </v-button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick } from "vue";
import TheWarning from "./TheWarning.vue";
import TheBlur from "./TheBlur.vue";

// Состояния
const opacitybg = ref(1);
const inputs = ref([""]);
const answers = ref([]);
const answerRefs = ref([]);
const blurOpacity = ref(0);
const translateCopyBtn = ref("0px 50px");
const opacityCopyBtn = ref(0);
const currentlyHeldIndex = ref(null);
const isSenttext = ref(false);
const opacityW = ref(0.1);
const translateW = ref("0px -55px");

// Методы
const processAnswer = (rawText) => {
  // Убираем текст "User:" и "Assistant:" из ответа
  const cleanedText = rawText.replace(/User:.*?Assistant:/s, "").trim();

  const parts = [];
  const regex = /```(.*?)```/gs; // Ищем блоки между ```
  let match;
  let lastIndex = 0;

  while ((match = regex.exec(cleanedText)) !== null) {
    if (match.index > lastIndex) {
      // Добавляем текст до блока кода
      parts.push({ type: "text", content: cleanedText.slice(lastIndex, match.index).trim() });
    }
    // Добавляем блок кода
    parts.push({ type: "code", content: match[1].trim() });
    lastIndex = regex.lastIndex;
  }

  // Добавляем остаток текста
  if (lastIndex < cleanedText.length) {
    parts.push({ type: "text", content: cleanedText.slice(lastIndex).trim() });
  }

  return parts;
};


const sendAnswer = async () => {
  const question = inputs.value[0]?.trim();
  if (!question) {
    showWarning();
    return;
  }

  isSenttext.value = true;
  try {
    const response = await fetch("https://xgptback.vercel.app/api/ask/", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ question }),
    });

    if (!response.ok) {
      if (response.status === 504) {
        throw new Error("Сервер не отвечает, попробуйте еще раз!!");
      } else {
        throw new Error(`HTTP error! Status: ${response.status}`);
      }
    }

    const data = await response.json();
    if (data.answer) {
      const formattedParts = processAnswer(data.answer);
      answers.value.push({ parts: formattedParts, opacity: 0, translate: "0px 10px" });
      await nextTick();
      animateAnswer(answers.value.length - 1);
    }
  } catch (error) {
    console.error("Error making request:", error.message);
    alert(error.message.includes("504") ? "Сервер не отвечает, попробуйте еще раз!!" : "Произошла ошибка, попробуйте еще раз.");
  } finally {
    isSenttext.value = false;
    inputs.value = [""];
  }
};


const animateAnswer = (index) => {
  const answer = answers.value[index];
  opacitybg.value = 0;
  if (answer) {
    setTimeout(() => {
      answer.class = "active";
    }, 200);
    
  }
};

const showWarning = () => {
  opacityW.value = 1;
  translateW.value = "0px 75px";
  setTimeout(() => {
    opacityW.value = 0.1;
    translateW.value = "0px -55px";
  }, 2000);
};

const startHoldingInCopy = (index) => {
  currentlyHeldIndex.value = index;
  blurOpacity.value = 1;
  translateCopyBtn.value = "0px 0px";
  opacityCopyBtn.value = 1;
};

const stopHoldingInCopy = () => {
  blurOpacity.value = 0;
  translateCopyBtn.value = "0px 50px";
  opacityCopyBtn.value = 0;
  currentlyHeldIndex.value = null;
};

const copyText = async (text) => {
  try {
    const textToCopy = Array.isArray(text) ? text.join("\n") : text;
    await navigator.clipboard.writeText(textToCopy);
    console.log("Text copied:", textToCopy);
  } catch (error) {
    console.error("Failed to copy text:", error);
  }
};


const modifiedAnswers = computed(() =>
  answers.value.map((answer, index) => ({
    ...answer,
    class: index === currentlyHeldIndex.value ? "selected" : "",
  }))
);
</script>


<style scoped>
@keyframes load {
  from {
    opacity: 0.1;
  }

  to {
    opacity: 1;
  }
}

@keyframes logoAnimate {
  0% {
    transform: translateY(-10px);
  }

  50% {
    transform: translateY(0px);
  }

  100% {
    transform: translateY(-10px);
  }
}

.blur {
  transition: all 0.1s ease;
}
.answer.active {
  opacity: 1;
  transform: translateY(0px);
}
.revert-position {
  position: revert;
}

.custom-textarea {
  min-width: 100%;
}

.copy-btn {
  width: 80px;
  display: flex;
  padding: 2px;
  position: absolute;
  top: -40px;
  left: -1px;
  border-radius: 15px;
  font-family: "Kanit", "Inter";
  color: grey;
  justify-content: center;
  box-shadow: rgba(0, 0, 0, 1) 0px 5px 15px;
  background-color: #2e2e2e;
  transition: all 0.1s ease;
}

.copy-btn:active {
  color: white;
}

.shadow-logo {
  width: 100px;
  height: 1px;
  translate: 0px 10px;
  box-shadow: rgba(0, 255, 64, 1) 0px -20px 16px,
    rgba(0, 255, 64, 1) 0px -20px 24px, rgba(0, 255, 64, 1) 0px -20px 56px;
}

.loading {
  opacity: 0.1;
  animation: load 1s infinite alternate ease-in-out;
}

.ico-copy {
  width: 15px;
  height: 15px;
  padding: 7px 10px 10px 7px;
  translate: 0px -31px;
}

.text-code {
  max-width: 400px;
}

.copy-code {
  width: 30px;
  height: 30px;
  margin-left: 10px;
  border-radius: 5px;
  background-color: black;
  color: white;
  top: 0;
  border: 1px solid grey;
  transition: all 0.5s ease;
}


pre {
  background-color: black;
  padding: 10px;
  position: relative;
  margin: 5px;
  display: flex;
  justify-content: space-between;
  border-radius: 4px;
  white-space: pre-wrap;
}

.cont-inputs {
  width: 100%;
  display: flex;
  height: 150px;
  overflow: hidden;
  z-index: 3;
}

.answers-column {
  width: 100%;
  height: auto;
  overflow-y: auto;
  padding-top: 60px;
}

.bottom-cont {
  width: 100%;
  height: 5px;
}

.container {
  width: 100%;
  height: 100dvh;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  position: relative;
  overflow-y: auto;
}

.chat {
  height: 50px;
  border-top: 1px solid grey;
  background-color: #21252b;
  position: relative;
}

.center-logo {
  width: 100%;
  display: flex;
  position: absolute;
  top: 160px;
  gap: 15px;
  align-items: center;
  color: white;
  flex-direction: column;
  font-family: 'Inter';
  transition: all 0.5s ease;
}

.gygole {
  width: 60px;
  transition: all 0.5s ease;
  animation: logoAnimate 3s infinite ease-in-out;
}

.input-message {
  width: 100%;
  display: flex;
  gap: 10px;
  background-color: #21252b;
  height: 50px;
}

.send-btn {
  width: 60px;
  height: 30px;
  margin-top: 10px;
  margin-right: 15px;
  margin-bottom: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 7px;
  border: 0;
  position: absolute;
  bottom: 10px;
  right: 10px;
  background-color: #2e2e2e;
  transition: all 0.2s ease;
}

.send-btn:active {
  background-color: #4b4b4b;
  scale: 0.9;
}

.answer {
  width: auto;
  max-width: 70%;
  height: auto;
  padding: 10px;
  margin: 10px;
  opacity: 0;
  cursor: pointer;
  border-radius: 10px;
  color: white;
  font-family: "Kanit", "Inter";
  font-weight: 300;
  position: relative;
  transform: translateY(50px);
  white-space: normal;
  word-wrap: break-word;
  word-break: break-word;
  background-color: #2e2e2e;
  transition: all 0.5s ease;
}

.answer.active {
  opacity: 1;
  transform: translateY(0px);
}

.input-text {
  width: 100%;
  height: 30px;
  border: 0;
  color: white;
  white-space: normal;
  word-wrap: break-word;
  word-break: break-word;
  padding-left: 5px;
  font-family: "Kanit", "Inter";
  background-color: transparent;
  transition: all 0.5s ease;
}

input:focus {
  outline: none;
}

.icon {
  display: flex;
  margin-left: 10px;
  margin-top: 10px;
  width: 60px;
  height: 30px;
}

.icon2 {
  display: flex;
  width: 22.5px;
  height: 22.5px;
}
</style>
