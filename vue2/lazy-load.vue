<script>
/**
 * 懒加载组件
 *
 * @description
 * 页面内组件很多时，不可见的组件可以不急着加载渲染
 * @example
 * <lazy-load>
 *  <your-component />
 * </lazy-load>
 */

const isSupprotIO = typeof IntersectionObserver === 'function';

export default {
  name: 'LazyLoad',
  props: {
    // 如果设置为 '0px'，组件会在视口刚可见时加载，设置为大于0的值可更快加载
    offset: {
      type: String,
      default: '200px'
    },
    // 是否启用组件懒加载
    lazy: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      ready: false,
      io: null
    };
  },
  methods: {
    observe() {
      if (!isSupprotIO) {
        console.warn(
          '浏览器不支持IntersectionObserver，懒加载将降级为常规加载'
        );
      }
      if (!this.lazy || !isSupprotIO || this.ready) return;

      this.io = new IntersectionObserver(
        entries => {
          const entry = entries?.[0];
          if (entry?.isIntersecting) {
            this.ready = true;
            this.unobserve();
          }
        },
        { rootMargin: this.offset }
      );

      this.io.observe(this.$refs?.placeholderRef);
    },
    unobserve() {
      this.io?.disconnect();
    }
  },
  mounted() {
    this.observe();
  },
  beforeDestroy() {
    this.unobserve();
  },
  render(h) {
    if (!this.lazy || !isSupprotIO || this.ready) {
      return this.$slots.default;
    }
    return h('div', {
      ref: 'placeholderRef',
      // class 只是为了标识div，无其它意义也不需要设置具体的样式
      class: 'js-lazy-load-placeholder'
    });
  }
};
</script>
