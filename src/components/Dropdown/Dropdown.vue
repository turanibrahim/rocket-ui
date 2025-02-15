<script setup lang="ts">
import { computed, reactive, ref, defineEmits, watch } from 'vue';
import Chip from '../Chips/Chip.vue';
import Icon from '../Icon/Icon.vue';
import './dropdown.css';
export interface Option {
  value: string | number;
  label: string;
}
export interface SelectProps {
  /**
   * Placeholder Dropdown
   * @type InputHTMLAttributes['placeholder'];
   * @default ''
   * @example
   * <Dropdown placeholder="Placeholder" />
   * @link https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#placeholder
   */
  placeholder?: string;

  /**
   * Options of the Dropdown
   * @type Option[]
   * @default []
   * @example
   * <Dropdown
   *  :options="[
   *   {
   *    value: '1',
   *    label: 'Option 1',
   *   },
   *   {
   *    value: '2',
   *    label: '2',
   *   },
   *  ]"
   * />
   */
  options: Option[];

  /**
   * Allow to create new options
   * @type boolean
   * @default false
   * @example
   * <Dropdown taggable />
   */
  taggable?: boolean;

  /**
   * Allow to select multiple options
   * @type boolean
   * @default false
   * @example
   * <Dropdown multiple />
   */
  multiple?: boolean;

  /**
   * Disable the Dropdown
   * @type boolean
   * @default false
   * @example
   * <Dropdown disabled />
   */
  disabled?: boolean;

  /**
   * Loading state of the Dropdown
   * @type boolean
   * @default false
   * @example
   * <Dropdown loading />
   */
  loading?: boolean;

  /**
   * Icon to prepend
   * @type string
   * @default ''
   * @example
   * <Dropdown prependIcon="search" />
   */
  prependIcon?: string;

  /**
   * Icon to append
   * @type string
   * @default 'expand_less'
   * @example
   * <Dropdown appendIcon="expand_more" />
   */
  appendIcon?: string;

  /**
   * Value of the Dropdown
   * @type string | number | Option | Option[]
   * @default ''
   * @example
   * <Dropdown v-model="model" />
   */
  modelValue?: string | number | Option | Option[];
}
const props = withDefaults(defineProps<SelectProps>(), {
  options: () => [],
  placeholder: '',
  taggable: false,
  multiple: false,
  disabled: false,
  loading: false,
  prependIcon: '',
  appendIcon: 'expand_less',
  modelValue: '',
});
const selected = ref('');
const selectedMultiple = reactive<Option[]>([]);
const active = ref(false);
const inputModel = ref('');
const emit = defineEmits(['update:modelValue']);

/**
 * @description - HTML elements references
 */
const select = ref<HTMLElement | null>(null);
const input = ref<HTMLInputElement | null>(null);
/**
 * @description - Handles the appearance of the select list
 * @param e Click event
 * @returns void
 */
const setActive = (e: MouseEvent) => {
  if (props.disabled) return;
  e.stopPropagation();
  active.value = !active.value;
  if (active.value) {
    select.value?.classList.add('select--active');
    select.value?.focus();
    input.value?.focus();
    document.addEventListener('click', setActive);
    return;
  }
  select.value?.classList.remove('select--active');
  select.value?.blur();
  input.value?.blur();
  document.removeEventListener('click', setActive);
};
/**
 * @description - Selects an option
 * @param e Click event
 * @param option Selected option
 * @returns void
 */
const selectOption = (e: MouseEvent, option: Option) => {
  if (props.multiple || props.taggable) {
    e.stopPropagation();
    if (!selectedMultiple.includes(option)) {
      selectedMultiple.push(option);
    } else {
      selectedMultiple.splice(selectedMultiple.indexOf(option), 1);
    }
    inputModel.value = '';
    input.value?.focus();
    return;
  } else {
    emit('update:modelValue', option.value);
    selected.value = option.label;
    inputModel.value = option.label;
  }
};
/**
 * @description - Removes an option from the selected options
 * @param option Option to remove
 * @returns void
 */
const removeOption = (e: MouseEvent | KeyboardEvent, option: Option) => {
  if (e instanceof KeyboardEvent && e.key !== 'Backspace') return;
  if (inputModel.value !== '') return;
  e.stopPropagation();
  const index = selectedMultiple.findIndex((opt) => opt.value === option.value);
  selectedMultiple.splice(index, 1);
};
/**
 * @description - Handles the not existing options
 * @returns void
 */
const createTag = (e: KeyboardEvent) => {
  if (!props.taggable) return;
  e.stopPropagation();
  const value = inputModel.value;
  if (value === '') return;
  const option = props.options.find((opt) => opt.label === value);
  if (!option) {
    selectedMultiple.push({ value, label: value });
    inputModel.value = '';
    input.value?.focus();
  }
};
/**
 * @description - Search for options
 * @returns {Option[]} - Returns an array of options
 */
const searchedOptions = computed(() => {
  const result = props.options.filter((option) => {
    return option.label.toLowerCase().includes(inputModel.value.toLowerCase());
  });
  return result;
});
/**
 * @description - Watch the selected multiple options
 * @returns void
 */
watch(selectedMultiple, (value) => {
  emit('update:modelValue', value);
});
</script>
<template>
  <div class="dropdown">
    <div
      ref="select"
      :class="{ select: true, 'select--disabled': props.disabled }"
      @click="setActive"
    >
      <div
        :class="{
          'select__prepend-icon': true,
          'select__prepend-icon--active': active,
        }"
      >
        <slot name="prepend-icon" />
        <Icon v-if="!$slots['prepend-icon']" :name="props.prependIcon" />
      </div>
      <div v-if="props.taggable" class="select__tags">
        <Chip
          v-for="(option, index) in selectedMultiple"
          :key="index"
          appendIcon="close"
          class="select__tags__chip"
          :label="option.label"
          variant="primary"
          @click-icon="removeOption($event, option)"
        />
      </div>
      <div v-if="props.multiple" class="select__multiple">
        <p v-for="(option, index) in selectedMultiple" :key="index">
          {{ option.label + ',' }}
        </p>
      </div>
      <input
        id="select"
        ref="input"
        v-model="inputModel"
        class="select__input"
        :disabled="props.disabled"
        :placeholder="props.placeholder"
        type="text"
        @keydown.backspace="
          removeOption($event, selectedMultiple[selectedMultiple.length - 1])
        "
        @keydown.enter="createTag($event)"
      />
      <div
        :class="{
          'select__append-icon': true,
          'select__append-icon--active': active,
        }"
      >
        <slot name="append-icon" />
        <Icon v-if="!$slots['append-icon']" :name="props.appendIcon" />
      </div>
    </div>
    <ul :class="{ 'select-options': true, 'select-options--active': active }">
      <li
        v-for="option in searchedOptions"
        :key="option.value"
        :class="{
          'select-options__option': true,
          'select-options__option--active':
            option.label === selected || selectedMultiple?.includes(option),
        }"
        @click="selectOption($event, option)"
      >
        <div style="display: flex; align-items: center">
          <slot name="option-prepend" />
          <Icon
            v-if="!$slots['option-prepend']"
            :class="{
              'select-options__option__prepend-icon': true,
              'select-options__option__prepend-icon--active':
                option.label === selected || selectedMultiple?.includes(option),
            }"
            name="face"
          />
          <p
            :class="{
              'select-options__option__label': true,
              'select-options__option__label--active':
                option.label === selected || selectedMultiple?.includes(option),
            }"
          >
            {{ option.label }}
          </p>
        </div>
        <Icon
          v-if="option.label === selected || selectedMultiple?.includes(option)"
          :class="{
            'select-options__option__append-icon': true,
            'select-options__option__append-icon--active':
              option.label === selected || selectedMultiple?.includes(option),
          }"
          name="check"
        />
      </li>
    </ul>
  </div>
</template>
