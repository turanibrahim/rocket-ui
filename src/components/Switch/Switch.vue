<script setup lang="ts">
import {
  computed,
  reactive,
  watch,
  type HTMLAttributes,
  type InputHTMLAttributes,
  type LabelHTMLAttributes,
} from 'vue';
import './switch.css';
export interface Props {
  /**
   * id of the checkbox
   * @type HTMLAttributes['id']
   * @default ''
   * @example
   * <Checkbox id="checkbox" />
   * @link https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/id
   */
  id: HTMLAttributes['id'];

  /**
   * Input checked state
   * @type InputHTMLAttributes['checked']
   * @default false
   * @example
   * <Checkbox modelValue="true" />
   * @link https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox#checked
   */
  modelValue?: InputHTMLAttributes['checked'];

  /**
   * Input disabled state
   * @type InputHTMLAttributes['disabled']
   * @default false
   * @example
   * <Checkbox disabled="true" />
   * @link https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox#disabled
   */
  disabled?: InputHTMLAttributes['disabled'];

  /**
   * label of the checkbox
   * @type LabelHTMLAttributes['label']
   * @default ''
   * @example
   * <Checkbox label="Checkbox" />
   * @link https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label
   */
  label?: LabelHTMLAttributes['for'];

  /**
   * Hint text
   * @type string
   * @default ''
   * @example
   * <Checkbox hint="This is a hint" />
   */
  hint?: string;

  /**
   * Error message
   * @type string
   * @default ''
   * @example
   * <Checkbox errorMsg="This is an error" />
   */
  errorMsg?: string;

  /**
   * Size of the checkbox
   * @type 'small' | 'medium' | 'large'
   * @default 'medium'
   * @example
   * <Checkbox size="small" />
   */
  size?: 'small' | 'medium' | 'large';
}
const props = withDefaults(defineProps<Props>(), {
  id: 'switch',
  modelValue: false,
  size: 'medium',
  disabled: false,
  label: '',
  hint: '',
  errorMsg: '',
});
const emit = defineEmits(['update:modelValue']);
const state = reactive<{
  checked: InputHTMLAttributes['checked'];
}>({
  checked: false,
});
const classes = computed(() => {
  return {
    switch: true,
    'switch--disabled': props.disabled,
    [`switch--${props.size}`]: props.size,
    'switch--error': props.errorMsg,
  };
});
const onChange = (e: unknown) => {
  if (props.disabled) return;
  // @ts-expect-error: Unreachable code error
  state.checked = e.target.checked;
  emit('update:modelValue', state.checked);
};
watch(
  () => props.modelValue,
  (value) => {
    state.checked = value;
  },
  {
    // need immediate to set the state on first render for storybook
    // TODO: find a better way to do this
    immediate: true,
  }
);
</script>
<template>
  <div class="switch-container">
    <div :class="classes">
      <input
        :id="props.id"
        :checked="state.checked"
        class="switch__input"
        type="checkbox"
        @change="onChange"
      />
      <span class="slider round" />
    </div>
    <div
      :class="{
        'switch-texts': true,
        [`switch-texts--${props.size}`]: true,
      }"
    >
      <label :id="props.id" class="switch-texts__label" :for="props.id">
        {{ props.label }}
      </label>
      <p v-if="props.errorMsg" class="switch-texts__error">
        {{ props.errorMsg }}
      </p>
      <p v-else class="switch-texts__hint">{{ props.hint }}</p>
    </div>
  </div>
</template>
