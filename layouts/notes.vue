<template>
  <div>
    <HeaderBar />
    <div class="flex w-screen">
      <div class="min-w-64 w-64 max-w-64">
        <NotesMenuSideBar @navigate-page="navigatePage" @add-note="addNote" @add-tag="addTag" />
      </div>
      <div v-if="addNewNote" class="absolute
        inset-0
        flex
        items-center
        justify-center
        bg-gray-700 bg-opacity-50
        z-10">
        <div class="card-compact rounded-lg w-fit bg-neutral text-neutral-content">
          <div class="card-body">
            <div class="relative">
              <h1 class="text-2xl">Add New Note</h1>
              <a class="btn btn-sm btn-circle absolute right-0 top-0" @click="addNote">
                ✕
              </a>
            </div>
            <div class="bg-gray-700 rounded p-4">
              <div class="flex text-lg	h-8">
                <div v-for="(item, index) in items" :key="item.icon" class="p-1">
                  <div v-if="item.type === 'divider'" :key="`divider${index}`">
                    |
                  </div>
                  <button class="menu-item" :class="{ 'is-active': item.isActive ? item.isActive() : null }"
                    :title="item.title" @click="item.action">
                    <i :class="`ri-${item.icon} `" />
                  </button>
                </div>
              </div>
              <div class="divider" />
              <editor-content :editor="titleEditor" />
              <editor-content class="mt-4" :editor="editor" />
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
      <div v-if="addNewTag" class="absolute
        inset-0
        flex
        items-center
        justify-center
        bg-gray-700 bg-opacity-50
        z-10">
        <NotesTagModal @add-tag="addTag" />
      </div>
      <slot />
    </div>
  </div>
</template>

<script lang="ts">
import Document from '@tiptap/extension-document'
import Paragraph from '@tiptap/extension-paragraph'
import Heading from '@tiptap/extension-heading'
import Text from '@tiptap/extension-text'
import TaskItem from '@tiptap/extension-task-item'
import TaskList from '@tiptap/extension-task-list'
import { Editor, EditorContent } from '@tiptap/vue-3'
import 'remixicon/fonts/remixicon.css'
export default {
  name: "AddNote",
  components: {
    EditorContent,
  },
  data() {
    const addNewNote = false
    const addNewTag = false
    return {
      page: "home",
      addNewTag,
      addNewNote,
      tags: {},
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
  created() {
    const supabase = useSupabaseClient()
    useAsyncData('tags', async () => {
      const { data: { user } } = await supabase.auth.getUser()
      if (user) {
        const { data } = await supabase.from('tags').select('id, name').eq('user_id', user.id)
        if (data) {
          this.tags = data
        }
      }
    })
  },
  methods: {
    navigatePage(page: string) {
      this.page = page
    },
    addNote() {
      if (this.addNewNote == false) {
        this.addNewNote = true
      } else {
        this.addNewNote = false
      }
    },
    addTag() {
      if (this.addNewTag == false) {
        this.addNewTag = true
      } else {
        this.addNewTag = false
      }
    },
    async createNewTag() {
      const supabase = useSupabaseClient()
      const { data: { user } } = await supabase.auth.getUser()
      const { data } = await supabase
        .from('tags')
        .insert([
          { name: "", user_id: user.id },
        ])
        .select('id, name')
    },
    async deleteTag(id: number) {
      const supabase = useSupabaseClient()
      await supabase
        .from('tags')
        .delete()
        .eq('id', id)
      let newDatabaseTags = this.tags.filter((i: { id: number }) => i.id != id)
      this.tags = newDatabaseTags

    },
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

