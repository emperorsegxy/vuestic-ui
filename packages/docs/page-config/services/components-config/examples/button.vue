<template>
  <div class="flex flex-col">
    <VaCheckbox
      v-for="(_, name) in buttonProps"
      :key="name"
      v-model="buttonProps[name]"
      :label="name"
      class="mb-3"
    />
    <VaButton
      class="w-max"
      @click="reset"
    >
      Reset config
    </VaButton>
  </div>
</template>

<script setup>
import { computed, ref, watch } from "vue";

import { useGlobalConfig } from "vuestic-ui";

const getDefaultButtonProps = () => ({
  round: false,
  gradient: false,
  plain: false,
});

const buttonProps = ref(getDefaultButtonProps());

const componentsConfig = computed(() => ({ VaButton: buttonProps.value }));

const { mergeGlobalConfig } = useGlobalConfig();

watch(
  componentsConfig,
  (newValue) => {
    mergeGlobalConfig({
      components: newValue,
    });
  },
  { deep: true }
);

const reset = () => {
  buttonProps.value = getDefaultButtonProps();
};
</script>
