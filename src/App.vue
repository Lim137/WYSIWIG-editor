<template>
  <div class="main-page" ref="mainPage">
    <div class="page-conteiner">
      <buttons-block
        @undo="undo"
        @redo="redo"
        @convertToHeading="convertToHeading"
        @formatParagraph="formatParagraph"
        @addImageFromPrompt="addImageFromPrompt"
        @copyHTML="copyHTML"
      />
      <div
        class="page-conteiner__editor-content"
        contenteditable
        @input="updateForEdit"
        ref="editor"
      >
        <div v-if="content">
          <div :innerHTML="content"></div>
        </div>
        <edit-content-block-vue v-else />
      </div>
    </div>
  </div>
</template>

<script>
import Clipboard from "clipboard";
import focusDirective from "@/directives/focus.js";
import ButtonsBlock from "@/components/ButtonsBlock.vue";
import EditContentBlockVue from "@/components/EditContentBlock.vue";
export default {
  name: "App",
  directives: {
    focus: focusDirective,
  },
  data() {
    return {
      content: "",
      history: [],
      historyIndex: -1,
      cursorPosition: null,
      posTop: 0,
      buttonBlockOffset: 0,
    };
  },
  components: {
    ButtonsBlock,
    EditContentBlockVue,
  },
  mounted() {
    this.updateContent();
    window.addEventListener("scroll", this.handleScroll);
  },
  beforeUnmount() {
    // Удаляем обработчик события scroll перед размонтированием компонента
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
      this.posTop = window.pageYOffset || document.documentElement.scrollTop;
      const threshold = 77;
      if (this.posTop > threshold) {
        this.buttonBlockOffset = this.posTop - threshold;
      } else {
        this.buttonBlockOffset = 0;
      }
    },
    updateContent() {
      if (this.historyIndex < this.history.length - 1) {
        this.history.splice(this.historyIndex + 1);
      }
      this.content = this.$refs.editor.innerHTML;
      // this.restoreCursorPosition();
      this.history.push(this.content);
      this.historyIndex = this.history.length - 1;
    },
    updateForEdit() {
      if (this.historyIndex < this.history.length - 1) {
        this.history.splice(this.historyIndex + 1);
      }
      // this.content = this.$refs.editor.innerHTML;
      this.history.push(this.$refs.editor.innerHTML);
      this.historyIndex = this.history.length - 1;
    },
    undo() {
      if (this.historyIndex > 0) {
        --this.historyIndex;
      }
      this.content = this.history[this.historyIndex];
    },
    redo() {
      if (this.historyIndex < this.history.length - 1) {
        this.historyIndex++;
        this.content = this.history[this.historyIndex];
      }
    },
    formatParagraph() {
      const selection = window.getSelection();
      if (selection.rangeCount > 0) {
        const range = selection.getRangeAt(0);
        const paragraph = document.createElement("p");
        paragraph.classList.add("page-conteiner__text");
        range.surroundContents(paragraph);
        if (
          paragraph.closest(".page-conteiner__text") ||
          paragraph.closest(".page-conteiner__title")
        ) {
          // Если элемент является потомком, добавляем класс "page-conteiner__texttext"
          paragraph.classList.add("page-conteiner__inserted-text");
        }
      }
      this.updateContent();
    },
    convertToHeading() {
      const selection = window.getSelection();
      if (selection.rangeCount > 0) {
        const range = selection.getRangeAt(0);
        const heading = document.createElement("h1");
        heading.classList.add("page-conteiner__title");
        range.surroundContents(heading);
        if (
          heading.closest(".page-conteiner__text") ||
          heading.closest(".page-conteiner__title")
        ) {
          heading.classList.add("page-conteiner__inserted-title");
        }
        this.updateContent();
      }
    },
    addImageFromPrompt() {
      const imageUrl = prompt("Введите URL изображения:");

      if (imageUrl) {
        const img = document.createElement("img");
        img.src = imageUrl;
        img.classList.add("page-conteiner__image");
        // Добавляем изображение в редактор
        const editor = this.$refs.editor;
        if (editor) {
          editor.focus();
          const selection = window.getSelection();
          const range = selection.getRangeAt(0);
          range.collapse(false);
          range.insertNode(img);
          if (
            img.closest(".page-conteiner__text") ||
            img.closest(".page-conteiner__title")
          ) {
            img.classList.add("page-conteiner__inserted-image");
          }
          this.updateContent();
        }
      }
    },
    copyHTML() {
      const clipboard = new Clipboard("button", {
        text: () => this.history[this.historyIndex],
      });
      clipboard.on("success", () => {
        clipboard.destroy();
        alert("HTML скопирован в буфер обмена.");
      });
      clipboard.on("error", () => {
        clipboard.destroy();
        alert("Не удалось скопировать HTML. Пожалуйста, скопируйте вручную.");
      });
    },
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,400;0,700;1,400&display=swap");
.main-page {
  display: flex;
  align-items: center;
  justify-content: center;
}
.page-conteiner {
  width: 960px;
  height: 100%;
  background-color: #1e1e1e;
  display: flex;
  flex-direction: column;
  padding: 77px 107px 107px;
}

.page-conteiner__editor-content {
  display: flex;
  flex-direction: column;
  color: #eaeaea;
}
/* не работают кнопки вперед и назад после выноса в одтельный компонент контента (есть смысл попробовать вынести в отдельный компонент только блок v-else) */
</style>
