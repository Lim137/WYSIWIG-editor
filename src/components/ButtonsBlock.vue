<template>
  <div class="buttons-block" :style="styleForButtonBlock">
    <img
      class="buttons-block__icon"
      @click="$emit('undo')"
      src="@/icons/undo.png"
      alt=""
    />
    <img
      class="buttons-block__icon"
      @click="$emit('redo')"
      src="@/icons/redo.png"
      alt=""
    />
    <img
      class="buttons-block__icon"
      @click="$emit('convertToHeading')"
      src="@/icons/formatToHeader.png"
      alt=""
    />
    <img
      class="buttons-block__icon"
      @click="$emit('formatParagraph')"
      src="@/icons/formatToParagraph.png"
      alt=""
    />
    <img
      class="buttons-block__icon"
      @click="$emit('addImageFromPrompt')"
      src="@/icons/insertImage.png"
      alt=""
    />
    <button
      @click="$emit('copyHTML')"
      class="buttons-block__text-button buttons-block__html-copy-button"
    >
      Скопировать HTML
    </button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      buttonBlockOffset: 0,
    };
  },
  emits: {
    undo: null,
    redo: null,
    convertToHeading: null,
    formatParagraph: null,
    addImageFromPrompt: null,
    copyHTML: null,
  },
  mounted() {
    window.addEventListener("scroll", this.handleScroll);
  },
  beforeUnmount() {
    window.removeEventListener("scroll", this.handleScroll);
  },
  computed: {
    styleForButtonBlock() {
      return {
        transform: this.buttonBlockOffset
          ? `translateY(${this.buttonBlockOffset}px)`
          : `translateY(0px)`,
      };
    },
  },
  methods: {
    handleScroll() {
      const scrollTop =
        window.pageYOffset || document.documentElement.scrollTop;
      const threshold = 77;
      if (scrollTop > threshold) {
        this.buttonBlockOffset = scrollTop - threshold;
      } else {
        this.buttonBlockOffset = 0;
      }
    },
  },
};
</script>

<style scoped>
.buttons-block {
  display: flex;
  gap: 12px;
  background-color: rgba(30, 30, 30);
  width: 853px;
}
.buttons-block__icon {
  width: 42px;
  height: 36px;
}
.buttons-block__text-button {
  color: #639eff;
  font-family: "Roboto", sans-serif;
  font-size: 15px;
  font-style: normal;
  line-height: 23px;
  align-self: center;
  background-color: #1e1e1e;
}
.buttons-block__html-copy-button {
  margin-left: 7px;
}
</style>
