<template>
  <v-sheet class="d-flex flex-nowrap">
    <label :for="id" class="font-weight-bold">{{ label }}</label>
    <input
      v-model="internalValue"
      :id="id"
      v-bind="$attrs"
      type="text"
      class="input-normal"
      :placeholder="placeholder"
    />
  </v-sheet>
</template>

<script setup>
import { ref, watch } from "vue";

const props = defineProps({
  modelValue: { type: [String, Number], default: "" },
  label: { type: String, default: "select-label" },
  id: { type: String, default: "select-id" },
  placeholder: { type: String, default: "Digite aqui"}
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