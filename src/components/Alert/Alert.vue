<script setup lang="ts">
import './alert.css';
import { computed } from 'vue';
import Icon from '../Icon/Icon.vue';

export interface Props {
  /**
   * Type of the Alert
   * @type 'success' | 'error' | 'warning' | 'info'
   * @default 'info'
   * @example
   * <Alert type="success" />
   */
  type: 'success' | 'error' | 'warning' | 'info';

  /**
   * Title of the Alert
   * @type string
   * @default ''
   * @example
   * <Alert title="Title" />
   */
  title?: string;

  /**
   * Description of the Alert
   * @type string
   * @default ''
   * @example
   * <Alert description="Description" />
   */
  description?: string;

  /**
   * Size of the Alert
   * @type 'small' | 'medium' | 'large'
   * @default 'medium'
   * @example
   * <Alert size="small" />
   */
  size?: 'small' | 'medium' | 'large';

  /**
   * Allow to close the Alert
   * @type boolean
   * @default true
   * @example
   * <Alert closable />
   */
  closable?: boolean;
}

const emit = defineEmits(['close']);
const props = withDefaults(defineProps<Props>(), {
  type: 'info',
  title: '',
  description: '',
  size: 'medium',
  closable: true,
});
const classes = computed(() => {
  return {
    alert: true,
    [`alert--${props.type}`]: true,
    [`alert--${props.size}`]: true,
  };
});
const icon = computed(() => {
  return {
    success: 'check_circle',
    error: 'error',
    warning: 'warning',
    info: 'info',
  }[props.type];
});
const iconSize = computed(() => {
  return {
    small: 12,
    medium: 16,
    large: 20,
  }[props.size];
});
const close = () => {
  emit('close');
};
</script>
<template>
  <div :class="classes">
    <div class="icon">
      <Icon :name="icon" :size="iconSize" />
    </div>
    <div class="texts">
      <p class="texts__title">{{ props.title }}</p>
      <p class="texts__description">{{ props.description }}</p>
    </div>
    <div v-if="props.closable" class="close">
      <Icon
        v-if="!$slots['close']"
        class="close__button"
        name="close"
        size="16"
        @click="close"
      />
      <slot v-else name="close" />
    </div>
  </div>
</template>
