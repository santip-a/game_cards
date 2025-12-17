<script setup>
import CardItem from './cardItem.vue';
import { ref, watch, computed } from 'vue';

const props = defineProps({
  step: Number,
  quantityOpenCard: Number,
  arrCardMix: Array,
})

const emit = defineEmits(['update:step', 'update:quantityOpenCard'])

const arrCard = ref(props.arrCardMix);
const openCards = ref([]);


watch(openCards, (newVal) => {
  if (newVal === openCards.value[0] || newVal === openCards.value[0]) {
    return
  }

  if (openCards.value[0]) {
    arrCard.value[openCards.value[0].index].isOpen = true;
  }
  if (openCards.value[1]) {
    arrCard.value[openCards.value[1].index].isOpen = true;
  }
  if (openCards.value.length === 2 && openCards.value[0].id === openCards.value[1].id) {

    arrCard.value[openCards.value[0].index].found = true;
    arrCard.value[openCards.value[1].index].found = true;

    openCards.value = [];
    emit('update:step', props.step + 1);
    emit('update:quantityOpenCard', props.quantityOpenCard + 1)
  } else if (openCards.value.length === 2) {
    setTimeout(() => {
      arrCard.value[openCards.value[0].index].isOpen = false;
      arrCard.value[openCards.value[1].index].isOpen = false;

      openCards.value = [];
      emit('update:step', props.step + 1);
    }, 500)
  }
});


watch(
  () => props.arrCardMix,
  (newVal) => {
    arrCard.value = [...newVal]; // новый массив
    openCards.value = [];
  },
  { immediate: true }
);

/* ---------- 1️⃣  Вычисляем количество колонок  ---------- */
const columnsCount = computed(() => {
  // Минимум 2, максимум 6 (чтобы не получилось 1‑колоночный «список»)
  const n = arrCard.value.length;               // количество карточек, уже перемешанных
  const maxCols = 6;
  // При 6‑18 карточках лучше не превышать 6 колонок
  const cols = Math.min(maxCols, Math.ceil(Math.sqrt(n)));
  // Если карточек < 4, делаем 2 колонки (чтобы не получились «узкие» строки)
  return Math.max(2, cols);
});

</script>

<template>

  <div class="list cards-grid" :style="{ '--cols': columnsCount }" >
    <CardItem v-for="(elem, index) in arrCard" :key="`card-${index}-${elem.id || elem.icon}`" :icon="elem.icon"
      :id="elem.id" :index="index" :isFound="elem.found" :isOpen="elem.isOpen" v-model:openCards="openCards" />
  </div>
</template>





<style scoped>
.list {
  /* Становимся flex‑контейнером только для центрирования, а не для раскладки */
  display: flex;
  justify-content: center;
  align-items: stretch;
  padding: 0;
  /* padding будет задаваться в .cards-grid */
}

/* --------- Гибкая сетка --------- */
.cards-grid {
  /* Сетка растягивается на всю высоту/ширину контейнера .content */
  width: 100%;
  height: 100%;
  display: grid;

  grid-template-columns: repeat(var(--cols, 4), minmax(clamp(60px, 12vw, 150px), 1fr));

  /* Высота строк будет такой же, как ширина (aspect‑ratio 3/4 у карточки). */
  /* Мы задаём автоматическую высоту, а внутри карточки закрепляем aspect‑ratio. */
  grid-auto-rows: 1fr;

  /* Адаптивные отступы и промежутки, которые тоже «сжимаются» */
  gap: clamp(4px, 1.5vmin, 12px);
  /* padding: clamp(4px, 1.5vmin, 12px); */

  /* Без скролла внутри сетки */
  overflow: hidden;
}

@media (max-aspect-ratio: 1/1) {
  .cards-grid {
    grid-template-columns: repeat(min(var(--cols, 3)),
          minmax(clamp(50px, 12vw, 120px), 1fr));
    }
  }

/* Стили карточки, которые «протягиваются» на всю ячейку */
::v-deep .card {
  width: 100%;
  height: 100%;
  /* aspect‑ratio уже указан в CardItem.vue → сохраняем */
}
</style>
