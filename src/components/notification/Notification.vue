<template>
  <alert
    class="fade"
    :class="customClass"
    :style="styles"
    :type="type"
    :duration="duration"
    :dismissible="dismissible"
    @dismissed="onDismissed"
  >
    <div class="media" style="margin: 0">
      <div v-if="icons" class="media-left">
        <span :class="icons" style="font-size: 1.5em"></span>
      </div>
      <div class="media-body">
        <div v-if="title" class="media-heading">
          <b>{{ title }}</b>
        </div>
        <div v-if="html" v-html="content"></div>
        <div v-else>{{ content }}</div>
      </div>
    </div>
  </alert>
</template>

<script>
import { addClass, removeClass } from '../../utils/dom.utils';
import { isString } from '../../utils/object.utils';
import Alert from '../alert/Alert.vue';
import { TYPES, PLACEMENTS } from '../../constants/notification.constants';

const IN_CLASS = 'in';
const ICON = 'glyphicon';
const WIDTH = 300;
const TRANSITION_DURATION = 300;

export default {
  components: { Alert },
  props: {
    title: { type: String, default: undefined },
    content: { type: String, default: undefined },
    html: {
      type: Boolean,
      default: false,
    },
    duration: {
      type: Number,
      default: 5000,
    },
    dismissible: {
      type: Boolean,
      default: true,
    },
    type: { type: String, default: undefined },
    placement: { type: String, default: undefined },
    icon: { type: String, default: undefined },
    customClass: { type: null, default: undefined },
    cb: {
      type: Function,
      required: true,
    },
    queue: {
      type: Array,
      required: true,
    },
    offsetY: {
      type: Number,
      default: 15,
    },
    offsetX: {
      type: Number,
      default: 15,
    },
    offset: {
      type: Number,
      default: 15,
    },
  },
  data() {
    return {
      height: 0,
      top: 0,
      horizontal:
        this.placement === PLACEMENTS.TOP_LEFT ||
        this.placement === PLACEMENTS.BOTTOM_LEFT
          ? 'left'
          : 'right',
      vertical:
        this.placement === PLACEMENTS.TOP_LEFT ||
        this.placement === PLACEMENTS.TOP_RIGHT
          ? 'top'
          : 'bottom',
    };
  },
  computed: {
    styles() {
      const queue = this.queue;
      const thisIndex = queue.findIndex((vm) => vm._.uid === this._.uid);
      return {
        position: 'fixed',
        [this.vertical]: `${this.getTotalHeightOfQueue(queue, thisIndex)}px`,
        width: `${WIDTH}px`,
        transition: `all ${TRANSITION_DURATION / 1000}s ease-in-out`,
      };
    },
    icons() {
      if (isString(this.icon)) {
        return this.icon;
      }
      switch (this.type) {
        case TYPES.INFO:
        case TYPES.WARNING:
          return `${ICON} ${ICON}-info-sign`;
        case TYPES.SUCCESS:
          return `${ICON} ${ICON}-ok-sign`;
        case TYPES.DANGER:
          return `${ICON} ${ICON}-remove-sign`;
        default:
          return null;
      }
    },
  },
  created() {
    // get prev notifications total height in the queue
    this.top = this.getTotalHeightOfQueue(this.queue);
  },
  mounted() {
    const el = this.$el;
    el.style[this.vertical] = this.top + 'px';
    this.$nextTick(() => {
      el.style[this.horizontal] = `-${WIDTH}px`;
      this.height = el.offsetHeight;
      el.style[this.horizontal] = `${this.offsetX}px`;
      addClass(el, IN_CLASS);
    });
  },
  // unmounted() {
  //   console.log('unmounted')
  // },
  methods: {
    getTotalHeightOfQueue(queue, lastIndex = queue.length) {
      let totalHeight = this.offsetY;
      for (let i = 0; i < lastIndex; i++) {
        totalHeight += queue[i].height + this.offset;
      }
      return totalHeight;
    },
    onDismissed() {
      removeClass(this.$el, IN_CLASS);
      setTimeout(this.cb, TRANSITION_DURATION);
    },
  },
};
</script>
