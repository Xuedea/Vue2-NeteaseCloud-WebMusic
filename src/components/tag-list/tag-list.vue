<template>
  <div class="hot-tags" :style="wrapStyle">
    <div class="title" v-if="title">
      {{ title }}
    </div>
    <ul class="list">
      <li
        class="item-wrap"
        v-for="tag in tags"
        :key="tag.name"
        @click="onTagClick(tag)"
      >
        <span
          class="item"
          :class="
            active && activeTag && activeTag.name === tag.name ? 'active' : ''
          "
          >{{ tag.name }}
        </span>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: "TagList",
  props: {
    tags: Array, // [{name:xxx}]
    title: String,
    active: {
      default: false,
      type: Boolean,
    },
    defaultActive: Object,
    wrapStyle: Object,
  },
  data() {
    return {
      activeTag: this.defaultActive,
    };
  },
  watch: {
    activeTag(tag) {
      this.$emit("tagChange", tag);
    },
  },
  methods: {
    onTagClick(tag) {
      this.activeTag = tag;
    },
  },
};
</script>

<style lang="scss" scoped>
.hot-tags {
  display: flex;
  font-size: $font-size-sm;
  flex-wrap: nowrap;
  margin: 8px 0;
  width: 100%;
  .title {
    min-width: 60px;
    line-height: 36px;
    height: 36px;
  }
  .list {
    display: flex;
    width: 100%;
    flex-wrap: wrap;
    .item-wrap {
      margin: 6px 0;
      padding: 0 12px;
      height: 24px;
      line-height: 24px;
      cursor: pointer;
      &:not(:last-child) {
        border-right: 1px $grey solid;
      }
      .item {
        padding: 6px;
        &.active {
          color: $white;
          background-color: $theme-color;
        }
      }
    }
  }
}
</style>
