<script setup>
import { ref, watch } from 'vue';
import Popup from '@/components/PopupComponent.vue';

const props = defineProps({
  step: Number,
  quantityOpenCard: Number,
  onMix: Function,
  arrCardMix: Array,
});

const isOpen = ref(false);
const imageNumber = ref();
const imageLoaded = ref(false)

// const emit = defineEmits(['update:isOpenModal']);


function declinateSteps(number = props.step) {
  // Убираем отрицательные числа и приводим к целому
  const n = Math.abs(Math.floor(number));

  // Последние две цифры
  const lastTwo = n % 100;
  // Последняя цифра
  const lastOne = n % 10;

  if (lastTwo >= 11 && lastTwo <= 19) {
    return `${n} шагов`;
  }

  if (lastOne === 1) {
    return `${n} шаг`;
  }

  if (lastOne >= 2 && lastOne <= 4) {
    return `${n} шага`;
  }

  // Для 0 и всех остальных случаев (5-9, 0)
  return `${n} шагов`;
}

// функция исключения дублирования картинок в попапе
function createRandomPicker() {
  let numbers = Array.from({ length: 16 }, (_, i) => i + 1); // [1, 2, ..., 16]

  return function getRandomNumber() {
    // Если числа закончились — заполняем массив заново
    if (numbers.length === 0) {
      numbers = Array.from({ length: 16 }, (_, i) => i + 1);
    }

    // Выбираем случайный индекс
    const randomIndex = Math.floor(Math.random() * numbers.length);

    // Берём число и удаляем его из массива
    const selected = numbers[randomIndex];
    numbers.splice(randomIndex, 1);

    return selected;
  };
}
// Создаём функцию-генератор
const pickNumber = createRandomPicker();


// Подгружаем картинку до открытия попапа
function preloadImage(src) {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.src = src
    img.onload = resolve
    img.onerror = reject
  })
}

watch(() => props.quantityOpenCard, async () => {
  if (props.quantityOpenCard >= props.arrCardMix.length / 2) {
    const number = pickNumber()
    const src = `/popup/${number}.webp`

    imageLoaded.value = false
    imageNumber.value = number

    await preloadImage(src)   // ⬅️ ВОТ КЛЮЧ

    imageLoaded.value = true
    isOpen.value = true
  }
})

watch(isOpen, (val) => {
  if (val) imageLoaded.value = false
})

</script>

<template>
  <div class="header">
    <h1>Игра на память</h1>
    <!-- <button @click="isOpen = true">Открыть</button> -->
    <div class="flex">
      <h3>шаги: {{ step }}</h3>
      <button class="button" @click="onMix()">Сначала</button>
      <!-- <button class="button" @click="emit('update:isOpenModal', true)">Сначала</button> -->
      <h3>найдено {{ quantityOpenCard }} / {{ props.arrCardMix.length / 2 }}</h3>
    </div>

    <Popup v-model="isOpen" title="Давай ещё?" :onMix="onMix"  >
      <h2 class="popup__title">Ай молодец какой 👋</h2>
      <p class="popup__subtitle">Ты разгадал за {{ declinateSteps() }}</p>

      <div class="img-wrapper">
        <div v-if="!imageLoaded" class="loader">Загрузка...</div>
        <!-- <img  v-show="imageLoaded" class="img" :src="`/popup/${imageNumber}.webp`" alt="" @load="imageLoaded = true"> -->
        <img  v-if="imageNumber" v-show="imageLoaded" class="img" :src="`/popup/${imageNumber}.webp`"
          @load="imageLoaded = true">
      </div>

    </Popup>
  </div>

</template>

<style scoped>
.popup__title {
  font-size: 32px;
}

.popup__subtitle {
  font-size: 28px;
}

h1 {
  text-align: center;
  margin: 0 0 16px 0;
}

.flex {
  display: flex;
  justify-content: space-between;
  margin: 0 20px;
}

@media screen and (max-width: 365px) {
  .flex {
    margin: 0 5px;
    ;
  }
}

.button {
  font-size: 22px;
  border-radius: 8px;
  height: 40px;
  cursor: pointer;
  background-color: rgb(41, 175, 41);
  color: aliceblue;
  border: none;
  text-transform: uppercase;
  padding: 6px 12px;
}

.button:hover {
  background-color: rgb(34, 141, 34);
}

.button:active {
  background-color: rgb(231, 86, 86);
}



.img {
  margin-bottom: 20px;
}

.header {
  flex: 0 0 120px;
  overflow: hidden;
}
</style>
