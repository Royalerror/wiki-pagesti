<template>
  <app-layout title="Créer un article">
    <template #header>
      <h2 class="font-semibold text-xl text-gray-800 leading-tight">
        Créer un article
      </h2>
    </template>
    <div class="py-12">
      <div class="max-w-7xl mx-auto sm:px-6 lg:px-8 grid grid-cols-12 gap-3">
        <div class="col-start-4 col-end-13">
          <invisible-input
            type="text"
            placeholder="Title"
            class="text-3xl font-bold tracking-wider bg"
          />
        </div>
        <div class="col-start-4 col-end-13 border-b border-gray-300"></div>
        <div class="col-span-3">
          <template v-for="(heading, index) in summary" :key="index">
            <div>
              {{ heading }}
            </div>
          </template>

          {{ summary }}
        </div>
        <div class="col-span-9">
          <bubble-menu
            class="bubble-menu"
            :tippy-options="{ duration: 100 }"
            :editor="editor"
            v-if="editor"
          >
            <button
              @click="editor.chain().focus().toggleBold().run()"
              :class="{ 'is-active': editor.isActive('bold') }"
            >
              <i class="ri-bold"></i>
            </button>
            <button
              @click="editor.chain().focus().toggleItalic().run()"
              :class="{ 'is-active': editor.isActive('italic') }"
            >
              <i class="ri-italic"></i>
            </button>
            <button
              @click="setLink"
              :class="{ 'is-active': editor.isActive('link') }"
            >
              <i class="ri-links-line"></i>
            </button>
            <div class="border-l border-gray-100 opacity-60"></div>
            <button
              @click="editor.chain().focus().toggleHeading({ level: 1 }).run()"
              :class="{ 'is-active': editor.isActive('heading', { level: 1 }) }"
            >
              <i class="ri-h-1"></i>
            </button>
            <button
              @click="editor.chain().focus().toggleHeading({ level: 2 }).run()"
              :class="{ 'is-active': editor.isActive('heading', { level: 2 }) }"
            >
              <i class="ri-h-2"></i>
            </button>
          </bubble-menu>
          <editor-content :editor="editor" />
        </div>
      </div>
    </div>
  </app-layout>
</template>

<script>
import { defineComponent } from "vue";
import AppLayout from "@/Layouts/AppLayout.vue";
import { Editor, EditorContent, BubbleMenu } from "@tiptap/vue-3";
import StarterKit from "@tiptap/starter-kit";
import Link from "@tiptap/extension-link";
import Placeholder from "@tiptap/extension-placeholder";
import Typography from "@tiptap/extension-typography";
import { SmilieReplacer } from "@/SmilieReplacer.ts";
import InvisibleInput from "@/Components/InvisibleInput.vue";

export default defineComponent({
  components: {
    AppLayout,
    InvisibleInput,
    EditorContent,
    BubbleMenu,
  },
  data() {
    return {
      editor: null,
      summary: [],
    };
  },
  mounted() {
    this.editor = new Editor({
      onUpdate({ editor }) {
        let jsonText = editor.getJSON();
        let heading = jsonText.content
          .filter((elem) => elem.type == "heading")
          .map((elem) => elem.content[0].text);
        this.summary = heading;
        console.log(this.summary);
      },
      editorProps: {
        attributes: {
          class:
            "prose prose-sm sm:prose lg:prose-lg xl:prose-2xl m-5 focus:outline-none",
        },
      },
      extensions: [
        StarterKit,
        Placeholder.configure({
          placeholder: "Racontez nous quelque chose…",
        }),
        Link.configure({
          openOnClick: false,
        }),
        Typography,
        SmilieReplacer,
      ],
    });
  },

  beforeUnmount() {
    this.editor.destroy();
  },

  methods: {
    setLink() {
      const previousUrl = this.editor.getAttributes("link").href;
      if (previousUrl) {
        this.editor.chain().focus().extendMarkRange("link").unsetLink().run();

        return;
      }
      const url = window.prompt("URL", previousUrl);

      // cancelled
      if (url === null) {
        return;
      }

      // empty
      if (url === "") {
        this.editor.chain().focus().extendMarkRange("link").unsetLink().run();

        return;
      }

      // update link
      this.editor
        .chain()
        .focus()
        .extendMarkRange("link")
        .setLink({ href: url })
        .run();
    },
  },
});
</script>

<style lang="scss">
/* Basic editor styles */
.ProseMirror {
  p.is-editor-empty:first-child::before {
    content: attr(data-placeholder);
    float: left;
    color: #adb5bd;
    pointer-events: none;
    height: 0;
  }

  > * + * {
    margin-top: 0.25rem;
  }

  ul,
  ol {
    padding: 0 1rem;
  }

  blockquote {
    padding-left: 1rem;
    border-left: 2px solid rgba(#0d0d0d, 0.1);
  }

  img {
    max-width: 100%;
    height: auto;

    &.ProseMirror-selectednode {
      outline: 3px solid #68cef8;
    }
  }
}

.bubble-menu {
  display: flex;
  gap: 0.5rem;
  background-color: #0d0d0d;
  padding: 0.2rem 0.5rem;
  border-radius: 0.5rem;

  button {
    border: none;
    background: none;
    color: #fff;
    font-size: 0.85rem;
    font-weight: 500;
    opacity: 0.6;

    &:hover,
    &.is-active {
      opacity: 1;
    }
  }
}
</style>