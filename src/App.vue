<script setup>
import { ref, onMounted, watch } from 'vue';
import CardList from './components/cardList.vue';
import headerComponent from './components/headerComponent.vue';
import { cartData } from './data/svgName'
import Popup from '@/components/PopupComponent.vue';

const step = ref(0);
const quantityOpenCard = ref(0);
const quelityCards = ref();
const isOpen = ref(true);

const arrCardMix = ref([]);

function onMix(data = cartData) {
  // Создаем новый массив, удваивая элементы с помощью map()
  const doubled = [
    ...data.map(card => ({ ...card })), // Первая копия с новыми объектами
    ...data.map(card => ({ ...card }))  // Вторая копия с новыми объектами
  ];

  // Перемешиваем массив
  for (let i = doubled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [doubled[i], doubled[j]] = [doubled[j], doubled[i]];
  }

  // Добавляем уникальные id для каждого элемента (опционально)
  doubled.forEach((card, index) => {
    card.uniqueId = index; // Это поможет отличать одинаковые карточки
  });


  arrCardMix.value = doubled;
  step.value = 0;
  quantityOpenCard.value = 0;
}

function getRandomSubarray(arr, n) {
  // // Проверка входных данных
  // if (!Array.isArray(arr)) {
  //   throw new Error('Первый аргумент должен быть массивом');
  // }

  // if (!Number.isInteger(countToRemove) || countToRemove < 0) {
  //   throw new Error('Второй аргумент должен быть неотрицательным целым числом');
  // }

  // // Создаем копию исходного массива, чтобы не изменять оригинал
  // const arrayCopy = [...arr];
  // const newLength = arrayCopy.length - countToRemove;
  // const result = [];

  // // Выбираем случайные элементы
  // for (let i = 0; i < newLength; i++) {
  //   // Генерируем случайный индекс
  //   const randomIndex = Math.floor(Math.random() * arrayCopy.length);

  //   // Извлекаем элемент по случайному индексу и добавляем в результат
  //   result.push(arrayCopy[randomIndex]);

  //   // Удаляем выбранный элемент из копии, чтобы не выбирать его снова
  //   arrayCopy.splice(randomIndex, 1);
  // }

  // return result;

  if (n <= 0) return [];
  if (n >= arr.length) return arr.slice(); // если n >= 40 — вернём копию всего массива

  const copy = arr.slice();
  let len = copy.length;

  // Частично перемешиваем массив (алгоритм Фишера-Йетса)
  for (let i = len - 1; i > len - n - 1; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [copy[i], copy[j]] = [copy[j], copy[i]];
  }

  // Берём последние n элементов (они уже случайные)
  return copy.slice(-n);
}

function resset() {
  isOpen.value = true;
}

function setQuelityCards(value) {
  if (quelityCards.value ===value) {
    const f = getRandomSubarray(cartData, value);
    onMix(f);
    isOpen.value = false;
  } else {
    quelityCards.value = value
  }
}

watch(quelityCards, (newVal) => {
  isOpen.value = !isOpen.value;
  const f = getRandomSubarray(cartData, newVal);
  onMix(f)
})


onMounted(() => {
  getRandomSubarray(cartData, 6);
})
</script>


<template>
  <div class="main">
    <headerComponent :step :onMix="resset" :quantityOpenCard v-model:isOpenModal="isOpen" :arrCardMix/>
    <div class="content">
      <CardList v-model:step="step" v-model:quantityOpenCard="quantityOpenCard" ref="cardListRef"
        :arrCardMix="arrCardMix" />
    </div>
  </div>

  <Popup v-model="isOpen" title="Давай ещё раз?" :onMix :summitHidden="true" >
    <h2 class="popup__title">Сколько карточек угадываем?</h2>
    <div class="popup__button-list">
      <button class="popup__button-item" @click="setQuelityCards(3)">6</button>
      <button class="popup__button-item" @click="setQuelityCards(4)">8</button>
      <button class="popup__button-item" @click="setQuelityCards(6)">12</button>
      <button class="popup__button-item" @click="setQuelityCards(7)">14</button>
      <button class="popup__button-item" @click="setQuelityCards(8)">16</button>
      <button class="popup__button-item" @click="setQuelityCards(9)">18</button>
      <button class="popup__button-item" @click="setQuelityCards(12)">24</button>
      <button class="popup__button-item" @click="setQuelityCards(16)">32</button>
      <button class="popup__button-item" @click="setQuelityCards(20)">40</button>
      <button class="popup__button-item" @click="setQuelityCards(26)">52</button>
      <button class="popup__button-item" @click="setQuelityCards(32)">64</button>
      <button class="popup__button-item" @click="setQuelityCards(40)">80</button>
    </div>
  </Popup>
</template>

<style scoped>
.popup__title {
  font-size: 32px;
}

.popup__button-list {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 10px;
  margin: 20px auto;
  justify-content: center;
}

.popup__button-item {
  font-size: 28px;
  font-weight: bold;
  padding: 20px 4px;
  /* width: 40px; */
  width: 100%;
  cursor: pointer;
  border-radius: 8px;
  border: none;
  background-color: rgb(25, 102, 245);
  color: aliceblue;
}

.main {
  height: 100svh;
  /* width: 100%; */
  max-width: 1000px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  /* гарантируем отсутствие скролла */
  box-sizing: border-box;
  padding: 4px;
}

/* Точное вычисление высоты контейнера контента */
.content {
  flex: 1 1 0;
  /* занимает всё свободное место */
  overflow: hidden;
  /* без скролла */
  /* Высота = окно – шапка */
  height: calc(100svh - var(--header-height));
  box-sizing: border-box;
}
</style>
