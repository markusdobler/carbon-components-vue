<template>
  <button
    type="button"
    :style="data.styleObject"
    @click="toggle"
    :class="[
      `cv-tile-expandable ${carbonPrefix}--tile--expandable`,
      { [`${carbonPrefix}--tile--is-expanded`]: data.internalExpanded },
    ]"
    ref="el"
  >
    <div :class="`${carbonPrefix}--tile-content`">
      <span
        data-tile-atf
        :class="`${carbonPrefix}--tile-content__above-the-fold`"
        ref="aboveFold"
      >
        <slot>
          <!-- Above the fold content here -->
        </slot>
      </span>
      <div :class="`${carbonPrefix}--tile__chevron`">
        <span>{{ chevronLabel }}</span>
        <ChevronDown16 />
      </div>
      <span
        :class="`${carbonPrefix}--tile-content__below-the-fold`"
        ref="belowFold"
        v-show="data.internalExpanded || data.initialized"
      >
        <slot name="below">
          <!-- Rest of the content here -->
        </slot>
      </span>
    </div>
  </button>
</template>

<script setup>
import { carbonPrefix } from '../../global/settings';
import ChevronDown16 from '@carbon/icons-vue/es/chevron--down/16';
import { computed, nextTick, reactive, ref, watch } from 'vue';

const props = defineProps({
  expanded: Boolean,
  tileCollapsedLabel: String,
  tileExpandedLabel: String,
});
const data = reactive({
  styleObject: {
    maxHeight: 'initial',
  },
  initialized: false,
  internalExpanded: props.expanded,
});

watch(
  () => props.expanded,
  val => {
    if (val !== data.internalExpanded) {
      toggle(val);
    }
  }
);

const chevronLabel = computed(() => {
  return data.internalExpanded
    ? props.tileExpandedLabel
    : props.tileCollapsedLabel;
});

const el = ref(null);
const belowFold = ref(null);
function toggle(force) {
  let currentHeight = el?.value.getBoundingClientRect().height;
  if (!data.initialized) {
    data.styleObject.maxHeight = `${currentHeight}px`;
    data.initialized = true;
  }

  nextTick(() => {
    const forceType = typeof force;
    data.internalExpanded =
      forceType === 'boolean' ? force : !data.internalExpanded;

    const belowFoldHeight = belowFold?.value.getBoundingClientRect().height;

    if (data.internalExpanded) {
      currentHeight += belowFoldHeight;
    } else {
      currentHeight -= belowFoldHeight;
    }
    data.styleObject.maxHeight = `${currentHeight}px`;
  });
}
const emit = defineEmits(['expanded']);
watch(
  () => data.internalExpanded,
  val => {
    emit('expanded', val);
  }
);
</script>
