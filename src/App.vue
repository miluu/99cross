<script setup lang="ts">
import { computed, onBeforeUnmount, reactive } from 'vue';
import { keys, shuffle } from 'lodash-es';

const questions = reactive<Record<number, [number, number][]>>({});

const initQuestions = () => {
  for (let a = 1; a <= 9; a++) {
    for (let b = a; b <= 9; b++) {
      const res = a * b;
      if (!questions[res]) {
        questions[res] = [];
      }
      questions[res].push([a, b]);
    }
  }
};

initQuestions();

const results = computed(() => keys(questions).map((i) => Number(i)));

const currentQuestions = reactive<{
  results: number[];
  index: number;
  showAnswer: boolean;
  // right: number;
  // wrong: number;
}>({
  results: [],
  index: 0,
  showAnswer: false,
  // right: 0,
  // wrong: 0,
});
const currentQuestion = computed(() => {
  const r = currentQuestions.results[currentQuestions.index];
  const a = questions[r];
  return {
    result: r,
    answers: a,
  };
});

const resetQuestions = () => {
  currentQuestions.index = 0;
  // currentQuestions.right = 0;
  // currentQuestions.wrong = 0;
  currentQuestions.results = shuffle(results.value);
  currentQuestions.showAnswer = false;
};

const isEnd = () => currentQuestions.index >= currentQuestions.results.length;

const nextQuestion = () => {
  if (!isEnd()) {
    currentQuestions.index++;
  }
};

const onKeyDown = (e: KeyboardEvent) => {
  if (e.key === ' ') {
    currentQuestions.showAnswer = true;
  }
};

const onKeyUp = (e: KeyboardEvent) => {
  if (e.key === ' ') {
    currentQuestions.showAnswer = false;
    nextQuestion();
  }
  if (e.key === 'Enter') {
    resetQuestions();
  }
};

resetQuestions();

document.addEventListener('keydown', onKeyDown);
document.addEventListener('keyup', onKeyUp);

onBeforeUnmount(() => {
  document.removeEventListener('keydown', onKeyDown);
  document.removeEventListener('keyup', onKeyUp);
});
</script>

<template>
  <div class="wrap">
    <div
      :class="[
        'info',
        {
          hidden: isEnd(),
        },
      ]"
    >
      <span
        >第{{ currentQuestions.index + 1 }}/{{
          currentQuestions.results.length
        }}题</span
      >
    </div>
    <div class="res">
      <span v-if="isEnd()">通关了</span>
      <span v-else>{{ currentQuestion.result }}</span>
    </div>
    <div :class="['answer', { hidden: !currentQuestions.showAnswer }]">
      <span v-for="(item, i) in currentQuestion.answers" :key="i">
        {{ item[0] }}×{{ item[1] }}={{ currentQuestion.result }}
      </span>
    </div>
  </div>
  <div class="btns">
    <div
      class="btn"
      @click="nextQuestion"
      @mousedown="currentQuestions.showAnswer = true"
      @mouseup="currentQuestions.showAnswer = false"
    >
      下一题（空格）
    </div>
    <div class="btn" @click="resetQuestions">重新开始（回车）</div>
  </div>
</template>

<style lang="scss" scoped>
.wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.res {
  font-size: 300px;
  flex: auto;
}
.info,
.answer {
  flex: none;
  display: flex;
  column-gap: 50px;
  justify-content: center;
  &.hidden {
    visibility: hidden;
  }
}
.btns {
  position: fixed;
  bottom: 40px;
  right: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  column-gap: 20px;
}
.btn {
  padding: 20px;
  background-color: #eee;
  cursor: pointer;
  user-select: none;
}
</style>
