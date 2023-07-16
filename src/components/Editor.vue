<script setup lang="ts">
import { ref, computed, watch, onBeforeUnmount } from "vue";
import { useEditor, EditorContent } from "@tiptap/vue-3";
import StarterKit from "@tiptap/starter-kit";
import Document from "@tiptap/extension-document";
import TextAlign from "@tiptap/extension-text-align";
import Underline from "@tiptap/extension-underline";
import Subscript from "@tiptap/extension-subscript";
import Superscript from "@tiptap/extension-superscript";
import CharacterCount from "@tiptap/extension-character-count";
import Typography from "@tiptap/extension-typography";
import { Color } from "@tiptap/extension-color";
import TextStyle from "@tiptap/extension-text-style";
import Link from "@tiptap/extension-link";
import Highlight from "@tiptap/extension-highlight";
import FontFamily from "@tiptap/extension-font-family";
import Image from "@tiptap/extension-image";

const props = defineProps<{
  modelValue: string;
  maxLimit: number;
}>();
const emits = defineEmits(["update:modelValue"]);

const editor = useEditor({
  content: props.modelValue,
  extensions: [
    StarterKit,
    Document,
    Underline,
    Subscript,
    Superscript,
    Typography,
    CharacterCount.configure({
      limit: props.maxLimit,
    }),
    TextAlign.configure({
      types: ["heading", "paragraph"],
    }),
    TextStyle,
    Color,
    Link,
    Highlight.configure({ multicolor: true }),
    FontFamily,
    Image,
  ],
  onUpdate: () => {
    emits("update:modelValue", editor.value.getHTML());
  },
});

const currentColorFeature = ref<null | string>(null);
const showColorPicker = ref<boolean>(false);

const updateColorPicker = (event: string) => {
  currentColorFeature.value === "textStyle"
    ? editor.value.chain().focus().setColor(event).run()
    : editor.value.chain().focus().toggleHighlight({ color: event }).run();

  showColorPicker.value = false;
};

const addImage = () => {
  const url = window.prompt("URL");

  if (url) {
    editor.value.chain().focus().setImage({ src: url }).run();
  }
};

const textActions = ref([
  { slug: "bold", icon: "format-bold", active: "bold" },
  { slug: "italic", icon: "format-italic", active: "italic" },
  { slug: "underline", icon: "format-underline", active: "underline" },
  { slug: "strike", icon: "format-strikethrough", active: "strike" },
  {
    slug: "align",
    option: "left",
    icon: "format-align-left",
    active: { textAlign: "left" },
  },
  {
    slug: "align",
    option: "center",
    icon: "format-align-center",
    active: { textAlign: "center" },
  },
  {
    slug: "align",
    option: "right",
    icon: "format-align-right",
    active: { textAlign: "right" },
  },
  {
    slug: "align",
    option: "justify",
    icon: "format-align-justify",
    active: { textAlign: "justify" },
  },
  { slug: "bulletList", icon: "format-list-bulleted", active: "bulletList" },
  { slug: "orderedList", icon: "format-list-numbered", active: "orderedList" },
  { slug: "subscript", icon: "format-subscript", active: "subscript" },
  {
    slug: "superscript",
    icon: "format-superscript",
    active: "superscript",
  },
  { slug: "undo", icon: "undo", active: "undo" },
  { slug: "redo", icon: "redo", active: "redo" },
  { slug: "clear", icon: "format-clear", active: "clear" },
  { slug: "link", icon: "link", active: "link" },
  { slug: "textStyle", icon: "format-color-text", active: "textStyle" },
  { slug: "highlight", icon: "format-color-highlight", active: "highlight" },
  { slug: "image", icon: "image-outline", active: "image" },
]);

const charactersCount = computed(() => {
  return editor.value.storage.characterCount.characters();
});

const wordsCount = computed(() => {
  return editor.value.storage.characterCount.words();
});

const limitWarning = computed(() => {
  const isCloseToMax = charactersCount.value >= props.maxLimit - 20;
  const isMax = charactersCount.value === props.maxLimit;

  if (isCloseToMax && !isMax) return "warning";
  if (isMax) return "danger";

  return "";
});

watch(props.modelValue, (newValue) => {
  if (editor.value.getHTML() === newValue) return;
  editor.value.commands.setContent(props.modelValue, false);
});

onBeforeUnmount(() => {
  editor.value.destroy();
});

function onActionClick(slug: string, option: string | null) {
  const vm = editor.value.chain().focus();
  const actionTriggers = {
    bold: () => vm.toggleBold().run(),
    italic: () => vm.toggleItalic().run(),
    underline: () => vm.toggleUnderline().run(),
    strike: () => vm.toggleStrike().run(),
    bulletList: () => vm.toggleBulletList().run(),
    orderedList: () => vm.toggleOrderedList().run(),
    align: () => vm.setTextAlign(option).run(),
    subscript: () => vm.toggleSubscript().run(),
    superscript: () => vm.toggleSuperscript().run(),
    undo: () => vm.undo().run(),
    redo: () => vm.redo().run(),
    clear: () => {
      vm.clearNodes().run();
      vm.unsetAllMarks().run();
    },
    link: () => setLink(),
    textStyle: () => {
      currentColorFeature.value = "textStyle";
      showColorPicker.value = !showColorPicker.value;
    },
    highlight: () => {
      currentColorFeature.value = "highlight";
      showColorPicker.value = !showColorPicker.value;
    },
    image: () => addImage(),
  };

  actionTriggers[slug]();
}

function setLink() {
  const previousUrl = editor.value.getAttributes("link").href;
  const url = window.prompt("URL", previousUrl);

  // cancelled
  if (url === null) {
    return;
  }

  // empty
  if (url === "") {
    editor.value.chain().focus().extendMarkRange("link").unsetLink().run();

    return;
  }

  // update link
  editor.value
    .chain()
    .focus()
    .extendMarkRange("link")
    .setLink({ href: url })
    .run();
}

function onHeadingClick(index: number) {
  const vm = editor.value.chain().focus();
  vm.toggleHeading({ level: index }).run();
}
</script>

<template>
  <v-card flat id="text-editor">
    <v-card-item class="toolbar bg-grey-lighten-5" v-if="editor">
      <v-menu hide-details>
        <template v-slot:activator="{ props }">
          <v-btn
            flat
            hide-details
            rounded="sm"
            class="mr-1"
            v-bind="props"
            size="x-small"
            icon="mdi-format-header-pound"
          >
          </v-btn>
        </template>
        <v-list class="dropdown-content" flat>
          <v-list-item
            v-for="index in 6"
            :key="index"
            :class="{
              'bg-black': editor.isActive('heading', { level: index }),
            }"
            :style="{ fontSize: 20 - index + 'px' }"
            @click="onHeadingClick(index)"
            role="button"
          >
            <v-list-title> Heading-{{ index }} </v-list-title>
          </v-list-item>
        </v-list>
      </v-menu>

      <v-btn
        flat
        v-bind="props"
        v-for="({ slug, option, active, icon }, index) in textActions"
        :key="index"
        :class="{ 'bg-black': editor.isActive(active) }"
        class="mr-1"
        rounded="sm"
        @click="onActionClick(slug, option)"
        :icon="`mdi-${icon}`"
        :title="icon"
        size="x-small"
      >
      </v-btn>

      <v-menu hide-details>
        <template v-slot:activator="{ props }">
          <v-btn
            flat
            hide-details
            rounded="sm"
            class="mr-1"
            v-bind="props"
            size="x-small"
            icon="mdi-format-font"
          >
          </v-btn>
        </template>
        <v-list class="dropdown-content" flat>
          <v-list-item
            v-for="(font, index) in ['Inter', 'Serif', 'Monospace']"
            :key="index"
            @click="editor.chain().focus().setFontFamily(font).run()"
            :class="{
              'bg-black': editor.isActive('textStyle', {
                fontFamily: font,
              }),
            }"
            role="button"
          >
            <v-list-title>{{ font }}</v-list-title>
          </v-list-item>
        </v-list>
      </v-menu>
    </v-card-item>

    <v-card-text>
      <v-color-picker
        v-if="showColorPicker"
        class="ma-2"
        position="absolute"
        show-swatches
        swatches-max-height="200px"
        mode="hex"
        hide-sliders
        hide-inputs
        hide-canvas
        @update:model-value="updateColorPicker"
        :model-value="editor.getAttributes(currentColorFeature).color"
      ></v-color-picker>

      <editor-content v-model:editor="editor" />

      <div v-if="editor" class="footer">
        <span class="characters-count" :class="maxLimit ? limitWarning : ''">
          {{ charactersCount }}
          {{ maxLimit ? `/ ${maxLimit} characters` : "characters" }}
        </span>
        |
        <span class="words-count"> {{ wordsCount }} words </span>
      </div>
    </v-card-text>
  </v-card>
</template>

<style lang="scss">
.v-color-picker {
  z-index: 1 !important;
  left: 50%;
  transform: translateX(-50%);
}

#text-editor {
  border: 2px solid #808080;
  position: relative;

  img {
    max-height: 300px !important;
    max-width: 200px !important;
  }

  .toolbar {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    border-bottom: 2px solid #808080;
  }

  .footer {
    color: #808080;
    font-size: 14px;
    text-align: right;
    padding: 6px;

    .characters-count {
      &.warning {
        color: orange;
      }

      &.danger {
        color: red;
      }
    }
  }

  .ProseMirror {
    min-height: 300px;
    outline: none;
    padding: 0.5rem;

    > p:first-child {
      margin-top: 0.5em;
    }

    > h1,
    h2,
    h3,
    h4,
    h5,
    h6 {
      &:first-child {
        margin-top: 0.5em;
      }
    }
  }
}
</style>
