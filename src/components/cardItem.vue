<script setup>
import { ref, watch } from 'vue';
import { icons } from '../data/svgIcons'

const props = defineProps({
  icon: String,
  id: Number,
  index: Number,
  isFound: Boolean,
  isOpen: Boolean,
  openCards: Array,
})


const openCard = ref(props.isOpen);

const showFlash = ref(false);
let flashTimeout = null;


const emit = defineEmits(['update:openCards',]);


function clickCard() {
  if (props.stopOpen || props.isFound || props.openCards.length > 1) {
    return
  }

  if (props.openCards[0]) {
    if (props.openCards[0].index === props.index) {
      return
    }
  }


  const newOpenCards = [...props.openCards, { id: props.id, index: props.index }];

  emit('update:openCards', newOpenCards)

  openCard.value = !openCard.value
}

// Следим за изменением isFound
watch(
  () => props.isFound,
  (newVal) => {
    // Только если стало true
    if (newVal === true) {
      // Отменяем предыдущий таймер (на случай быстрых переключений)
      if (flashTimeout) {
        clearTimeout(flashTimeout)
      }

      // Включаем тень
      showFlash.value = true

      // Через 1 секунду — выключаем
      flashTimeout = setTimeout(() => {
        showFlash.value = false
        flashTimeout = null
      }, 500)
    }
  }
)



</script>

<template>

  <div class="card search-result" :class="{ cardClose: !isOpen, 'flash-shadow': showFlash }" @click="clickCard"
    :id="props.id">
    <!-- <img class="image" :class="{ imageHidden: !isOpen }" :src="props.url" alt="image"> -->
    <component :class="{ imageHidden: !isOpen }"  :is="icons[props.icon]" class="image"/>
  </div>
</template>

<style scoped>
.card {
  /* Карточка уже имеет aspect‑ratio: 3 / 4, но делаем её flex‑контейнером */
  display: flex;
  align-items: center;
  /* по вертикали центрировать */
  justify-content: center;
  /* по горизонтали центрировать */
  overflow: hidden;
  /* лишнее (если вдруг) не покажем */
  width: 100%;
  height: 100%;
  aspect-ratio: 3 / 4;
  /* сохраняем пропорцию карточки */
  border: 4px solid rgb(0, 17, 255);
  border-radius: 16px;
  background: #fff;
  padding: 6px;
  box-sizing: border-box;
  cursor: pointer;
  transition: box-shadow 0.2s ease;
}

.cardClose {
  background-color: blue;
}

/* Всё, что было в .image, оставляем, но добавим object-fit */
.image {
  /* Ограничиваем размеры, но НЕ задаём фиксированную ширину/высоту */
  max-width: 100%;
  max-height: 100%;
  width: auto;
  height: auto;

  /* Сохраняем собственные пропорции, полностью помещаем в контейнер */
  object-fit: contain;
  /* <‑‑ самое главное */
  /* (можно также использовать `scale-down`, если хотите, чтобы картинка
        показывалась лишь при достаточном размере, иначе оставалась оригинальная) */

  transition: opacity 0.2s ease;
}

.imageHidden {
  opacity: 0;
  visibility: hidden;
}

.search-result {
  transition: box-shadow 0.2s ease;
}

.flash-shadow {
  box-shadow: 0 0 60px rgba(255, 0, 179, 0.7);
}
</style>
