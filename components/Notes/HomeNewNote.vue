<template>
  <div v-if="editor" class="editor border rounded-lg">
    <div class="card-compact rounded-lg bg-neutral text-neutral-content">
      <div class="card-body">
        <div class="flex text-lg h-8">
          <div v-for="(item, index) in items" :key="item.icon" class="p-1">
            <div v-if="item.type === 'divider'" :key="`divider${index}`">
              |
            </div>
            <button class="menu-item" :class="{ 'is-active': item.isActive ? item.isActive() : null }" :title="item.title"
              @click="item.action">
              <i :class="`ri-${item.icon} `" />
            </button>
          </div>
        </div>
        <div class="flex mt-3">
          <editor-content :editor="titleEditor" />
        </div>
        <editor-content :editor="editor" />
        <div class="flex space-x-2">
        </div>
        <div class="card-actions justify-end mr-0">
          <button class="btn-sm btn-primary rounded-lg" @click="saveNote">
            <i class="ri-save-fill"></i>
          </button>
          <button class="btn-sm btn-error rounded-lg" @click="resetNote">
            <i class="ri-delete-bin-7-fill"></i>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Document from '@tiptap/extension-document'
import Paragraph from '@tiptap/extension-paragraph'
import Heading from '@tiptap/extension-heading'
import Text from '@tiptap/extension-text'
import TaskItem from '@tiptap/extension-task-item'
import TaskList from '@tiptap/extension-task-list'
import { Editor, EditorContent } from '@tiptap/vue-3'
import 'remixicon/fonts/remixicon.css'
export default {
  name: "HomeNote",
  components: {
    EditorContent,
  },
  data() {
    const loadModal = false
    return {
      loadModal,
      tags: [],
      titleEditor: null,
      editor: null,
      items: [
        {
          icon: 'bold',
          title: 'Bold',
          action: () => this.editor.chain().focus().toggleBold().run(),
          isActive: () => this.editor.isActive('bold'),
        },
        {
          icon: 'italic',
          title: 'Italic',
          action: () => this.editor.chain().focus().toggleItalic().run(),
          isActive: () => this.editor.isActive('italic'),
        },
        {
          icon: 'strikethrough',
          title: 'Strike',
          action: () => this.editor.chain().focus().toggleStrike().run(),
          isActive: () => this.editor.isActive('strike'),
        },
        {
          icon: 'code-view',
          title: 'Code',
          action: () => this.editor.chain().focus().toggleCode().run(),
          isActive: () => this.editor.isActive('code'),
        },
        {
          icon: 'mark-pen-line',
          title: 'Highlight',
          action: () => this.editor.chain().focus().toggleHighlight().run(),
          isActive: () => this.editor.isActive('highlight'),
        },
        {
          type: 'divider',
        },
        {
          icon: 'h-1',
          title: 'Heading 1',
          action: () => this.editor.chain().focus().toggleHeading({ level: 1 }).run(),
          isActive: () => this.editor.isActive('heading', { level: 1 }),
        },
        {
          icon: 'h-2',
          title: 'Heading 2',
          action: () => this.editor.chain().focus().toggleHeading({ level: 2 }).run(),
          isActive: () => this.editor.isActive('heading', { level: 2 }),
        },
        {
          icon: 'paragraph',
          title: 'Paragraph',
          action: () => this.editor.chain().focus().setParagraph().run(),
          isActive: () => this.editor.isActive('paragraph'),
        },
        {
          icon: 'list-unordered',
          title: 'Bullet List',
          action: () => this.editor.chain().focus().toggleBulletList().run(),
          isActive: () => this.editor.isActive('bulletList'),
        },
        {
          icon: 'list-ordered',
          title: 'Ordered List',
          action: () => this.editor.chain().focus().toggleOrderedList().run(),
          isActive: () => this.editor.isActive('orderedList'),
        },
        {
          icon: 'list-check-2',
          title: 'Task List',
          action: () => this.editor.chain().focus().toggleTaskList().run(),
          isActive: () => this.editor.isActive('taskList'),
        },
        {
          icon: 'code-box-line',
          title: 'Code Block',
          action: () => this.editor.chain().focus().toggleCodeBlock().run(),
          isActive: () => this.editor.isActive('codeBlock'),
        },
        {
          type: 'divider',
        },
        {
          icon: 'double-quotes-l',
          title: 'Blockquote',
          action: () => this.editor.chain().focus().toggleBlockquote().run(),
          isActive: () => this.editor.isActive('blockquote'),
        },
        {
          icon: 'separator',
          title: 'Horizontal Rule',
          action: () => this.editor.chain().focus().setHorizontalRule().run(),
        },
        {
          type: 'divider',
        },
        {
          icon: 'text-wrap',
          title: 'Hard Break',
          action: () => this.editor.chain().focus().setHardBreak().run(),
        },
        {
          icon: 'format-clear',
          title: 'Clear Format',
          action: () => this.editor.chain()
            .focus()
            .clearNodes()
            .unsetAllMarks()
            .run(),
        },
        {
          type: 'divider',
        },
        {
          icon: 'arrow-go-back-line',
          title: 'Undo',
          action: () => this.editor.chain().focus().undo().run(),
        },
        {
          icon: 'arrow-go-forward-line',
          title: 'Redo',
          action: () => this.editor.chain().focus().redo().run(),
        },
      ],
    }
  },
  mounted() {
    this.titleEditor = new Editor({
      content: "Title",
      extensions: [
        Document,
        Heading.configure({
          HTMLAttributes: {
            class: 'card-title text-2xl bold mb-3'
          },
        }),
        Text,
      ],
      editorProps: {
        attributes: {
          class: 'prose focus:outline-none',
        },
      },
    })
    this.editor = new Editor({
      content: "Body",
      extensions: [
        Document,
        TaskList,
        TaskItem.configure({
          nested: true,
        }),
        Paragraph.configure({
          HTMLAttributes: {
          },
        }),
        Text,
      ],
      editorProps: {
        attributes: {
          class: 'prose focus:outline-none pb-2',
        },
      },
    })
  },
  beforeUnmount() {
    this.titleEditor.destroy()
    this.editor.destroy()
  },
  methods: {
    async saveNote() {
      const supabase = useSupabaseClient()
      const { data: { user } } = await supabase.auth.getUser()
      const { data } = await supabase
        .from('notes')
        .insert([
          { title: this.titleEditor.getHTML(), body: this.editor.getHTML(), user_id: user.id },
        ])
        .select('id, title, body')
      this.titleEditor.chain().focus().setContent("Title").run()
      this.editor.chain().focus().setContent("Body").run()
    },
    async resetNote() {
      this.titleEditor.chain().focus().setContent("Title").run()
      this.editor.chain().focus().setContent("Body").run()
    }
  }
}
</script>

<style lang="scss">
body {
  max-width: 30rem;
}

.centre-buttons {
  text-align: center;
}

.centre-buttons button {
  margin: 1rem;
}

ul[data-type="taskList"] {
  list-style: none;
  padding: 0;

  p {
    margin: 0;
  }

  li {
    display: flex;

    >label {
      flex: 0 0 auto;
      margin-right: 0.5rem;
      user-select: none;
    }

    >div {
      flex: 1 1 auto;
    }
  }
}

.menu-item {
  border-radius: 0.4rem;

  &.is-active,
  &:hover {
    color: #0D0D0D;
    background-color: #FFF;
  }
}
</style>
