<template>
  <v-sheet class="d-flex position-relative flex-nowrap">
    <label :for="id" class="font-weight-bold" v-if="label">{{ label }}</label>
    <select
      v-model="internalValue"
      :id="id"
      v-bind="$attrs"
      class="input-normal"
    >
      <option v-for="(option, idx) in options" :key="idx" :value="option.id">
        {{ option.name }}
      </option>
    </select>
    <v-icon icon="mdi-menu-down" class="position-absolute right-0 me-2" style="top: 50%; transform: translateY(-50%);"></v-icon>
  </v-sheet>
</template>

<script setup>
import { ref, watch } from "vue";

const props = defineProps({
  modelValue: { type: [String, Number], default: "" },
  label: { type: String, default: "Tipos" },
  id: { type: String, default: "select-id" },
  options: { type: Array, default: () => [] }
});

const internalValue = ref(props.modelValue);

const emit = defineEmits(["update:modelValue"]);

watch(internalValue, (newValue) => {
  emit("update:modelValue", newValue);
});
watch(
  () => props.modelValue,
  (newValue) => {
    internalValue.value = newValue;
  }
);
</script>
