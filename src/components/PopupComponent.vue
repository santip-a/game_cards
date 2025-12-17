<script setup>
const props = defineProps({
  modelValue: Boolean,
  title: String,
  onMix: Function,
  summitHidden: Boolean
})

const emit = defineEmits(['update:modelValue'])

const close = () => {
  props.onMix?.();
  emit('update:modelValue', false);
}
</script>


<template>
  <Teleport to="body">
    <transition name="fade">
      <div v-if="modelValue" class="overlay" @click.self="close">
        <div class="popup">
          <slot />
          <button  class="button" :class="{hidden: summitHidden}"   @click="close">{{ title ? title : 'Закрыть'  }}</button>
        </div>
      </div>
    </transition>
  </Teleport>
</template>


<style scoped>
.overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.popup {
  background: white;
  padding: 20px;
  border-radius: 8px;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.button {
  font-size: 26px;
  display: block;
  margin: 0 auto;
}

.hidden {
  display: none;
  color: red;
}
</style>




