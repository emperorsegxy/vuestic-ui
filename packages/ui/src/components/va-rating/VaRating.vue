<template>
  <div
    class="va-rating"
    :class="rootClass"
    :aria-label="tp($props.ariaLabel, { max: $props.max, value: $props.modelValue })"
  >
    <div
      class="va-rating__item-wrapper"
      @keyup.left="onArrowKeyPress(-1)"
      @keyup.right="onArrowKeyPress(1)"
      @mouseenter="onMouseEnter"
      @mouseleave="onMouseLeave"
    >
      <va-rating-item
        v-for="itemNumber in $props.max"
        :key="itemNumber"
        class="va-rating__item"
        v-bind="VaRatingItemProps"
        :aria-label="tp($props.ariaItemLabel, { max: $props.max, value: itemNumber })"
        :model-value="getItemValue(itemNumber - 1)"
        :tabindex="tabIndexComputed"
        :disabled="$props.disabled"
        :readonly="$props.readonly"
        @hover="isInteractionsEnabled && onItemHoveredValueUpdate(itemNumber - 1, $event)"
        @update:model-value="isInteractionsEnabled && onItemValueUpdate(itemNumber - 1, $event)"
        v-slot="{ value, onClick }"
      >
        <slot name="item" v-bind="{ value: value, onClick: onClick, index: itemNumber }">
          <template v-if="$props.numbers">
            <VaRatingItemNumberButton
              v-bind="VaRatingItemNumberButtonProps"
              :model-value="value"
              :item-number="itemNumber"
            />
          </template>
        </slot>
      </va-rating-item>
    </div>
    <span
      v-if="$props.texts && $props.texts.length === $props.max"
      class="va-rating__text-wrapper"
      :style="{ color: computedColor }"
    >
      {{ $props.texts[Math.round(visibleValue) - 1] }}
    </span>
  </div>
</template>

<script lang="ts">
import { defineComponent, computed, PropType } from 'vue'

import { extractComponentProps, filterComponentProps } from '../../utils/component-options'
import { useFormField, useFormFieldProps, useTranslation } from '../../composables'
import { useRating, useRatingProps } from './hooks/useRating'
import { useVaRatingColors, useVaRatingColorsProps } from './hooks/useVaRatingColors'

import { RatingValue } from './types'
import { useComponentPresetProp } from '../../composables/useComponentPreset'

import VaRatingItem from './components/VaRatingItem/VaRatingItem.vue'
import VaRatingItemNumberButton from './components/VaRatingItemNumberButton.vue'

const VaRatingItemProps = extractComponentProps(VaRatingItem, ['modelValue', 'itemNumber'])
const VaRatingItemNumberButtonProps = extractComponentProps(VaRatingItemNumberButton, ['modelValue', 'itemNumber'])

export default defineComponent({
  name: 'VaRating',
  props: {
    ...VaRatingItemNumberButtonProps,
    ...useRatingProps,
    ...useVaRatingColorsProps,
    ...useFormFieldProps,
    ...VaRatingItemProps,
    ...useComponentPresetProp,
    modelValue: { type: Number, default: 0 },
    numbers: { type: Boolean, default: false },
    halves: { type: Boolean, default: false },
    max: { type: Number, default: 5 },
    texts: { type: Array as PropType<string[]>, default: () => [] },

    ariaLabel: { type: String, default: '$t:currentRating' },
    ariaItemLabel: { type: String, default: '$t:voteRating' },
  },
  emits: ['update:modelValue'],
  components: { VaRatingItem, VaRatingItemNumberButton },
  setup (props) {
    const { computedClasses: rootClass } = useFormField('va-rating', props)
    const rating = useRating(props)
    const isInteractionsEnabled = computed(() => !props.disabled && !props.readonly)

    return {
      ...useTranslation(),
      ...useVaRatingColors(props),
      ...rating,
      rootClass,
      VaRatingItemProps: filterComponentProps(VaRatingItemProps),
      VaRatingItemNumberButtonProps: filterComponentProps(VaRatingItemNumberButtonProps),
      isInteractionsEnabled,
      tabIndexComputed: computed(() => isInteractionsEnabled.value ? 0 : undefined),
      onArrowKeyPress: (direction: 1 | -1) => {
        const step = props.halves ? RatingValue.HALF : RatingValue.FULL
        const nextStep = rating.visibleValue.value + step * direction
        const min = props.clearable ? 0 : step
        if (nextStep >= min && nextStep <= props.max) {
          rating.onItemValueUpdate(rating.visibleValue.value, step * direction)
        } else if (nextStep < min) {
          rating.onItemValueUpdate(min, 0)
        } else {
          rating.onItemValueUpdate(props.max, direction === -1 ? step * direction : 0)
        }
      },
    }
  },
})
</script>

<style lang="scss">
@import "../../styles/resources";
@import 'variables';

.va-rating {
  display: var(--va-rating-display);
  font-family: var(--va-font-family);

  &__number-item {
    @include normalize-button();

    font-size: var(--va-rating-number-item-font-size);
    margin: var(--va-rating-number-item-margin);
    font-weight: var(--va-rating-number-item-font-weight);

    @include flex-center();

    cursor: pointer;

    @at-root {
      .va-rating--disabled & {
        @include va-disabled();
      }

      .va-rating--readonly & {
        cursor: default;
      }
    }
  }

  &__item-wrapper {
    display: flex;
    cursor: var(--va-rating-item-wrapper-cursor);

    @at-root {
      .va-rating--readonly &,
      .va-rating--disabled & {
        cursor: default;
      }
    }
  }

  &-item {
    display: flex;

    @include flex-center();

    .va-rating--disabled & {
      @include va-disabled();

      &__wrapper {
        cursor: initial !important;
      }
    }

    .va-rating--readonly & &__wrapper {
      cursor: initial !important;
    }
  }

  &__text-wrapper {
    padding-left: 10px;
  }
}
</style>
