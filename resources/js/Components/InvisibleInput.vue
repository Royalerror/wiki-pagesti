<template>
  <input
    class="
      bg-transparent
      border-0
      focus:outline-none
      focus:ring-0
      placeholder-gray-500 placeholder-opacity-50
    "
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
    ref="input"
  />
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  props: ["modelValue"],

  emits: ["update:modelValue"],

  methods: {
    focus() {
      this.$refs.input.focus();
    },
  },
});
</script>
