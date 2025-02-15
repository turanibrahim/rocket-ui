<script setup lang="ts">
import { onMounted, ref, watch } from 'vue';
import Button from '../Button/Button.vue';
import Icon from '../Icon/Icon.vue';
import './snackbar.css';

export interface IProps {
  /**
   * Text of the snackbar
   * @type string
   * @default ''
   * @example
   * <Snackbar text="Snackbar" />
   */
  text: string;

  /**
   * Action text of the snackbar
   * @type string
   * @default ''
   * @example
   * <Snackbar actionText="Action" />
   */
  actionText?: string;

  /**
   * Show action of the snackbar
   * @type boolean
   * @default false
   * @example
   * <Snackbar showAction />
   */
  showAction?: boolean;

  /**
   * Show close of the snackbar
   * @type boolean
   * @default false
   * @example
   * <Snackbar showClose />
   */
  showClose?: boolean;

  /**
   * Left of the snackbar
   * @type boolean
   * @default false
   * @example
   * <Snackbar left />
   */
  left?: boolean;

  /**
   * Show of the snackbar
   * @type boolean
   * @default false
   * @example
   * <Snackbar show />
   */
  show?: boolean;

  /**
   * Timeout of the snackbar
   * @type number
   * @default 0
   * @example
   * <Snackbar timeout="1000" />
   */
  timeout?: number;
}
const props = withDefaults(defineProps<IProps>(), {
  text: '',
  actionText: '',
  showAction: false,
  showClose: false,
  left: false,
  show: false,
  timeout: 0,
});
const shown = ref(false);
const emit = defineEmits(['click:action', 'click:close']);
const onClickAction = () => {
  emit('click:action');
};
const onClose = () => {
  shown.value = false;
  emit('click:close');
};
onMounted(() => {
  if (props.timeout) {
    setTimeout(() => {
      shown.value = false;
      emit('click:close');
    }, props.timeout);
  }
});
/**
 * @description Watch for changes in the show prop
 */
watch(
  () => props.show,
  (show) => {
    shown.value = show;
  },
  { immediate: true }
);
</script>
<template>
  <div
    :class="{
      snackbar: true,
      'snackbar--closed': !shown,
      [props.left ? 'snackbar--left' : 'snackbar--right']: true,
    }"
  >
    <div class="snackbar__text">{{ props.text }}</div>
    <div v-if="props.showAction" class="snackbar__action">
      <slot name="action" />
      <Button
        v-if="!$slots['action']"
        size="small"
        variant="ghost"
        @click="onClickAction"
        >{{ props.actionText }}
      </Button>
    </div>
    <div v-if="props.showClose" class="snackbar__close">
      <slot name="close" />
      <Icon
        v-if="!$slots['close']"
        name="close"
        size="14"
        @click.stop="onClose"
      />
    </div>
  </div>
</template>
