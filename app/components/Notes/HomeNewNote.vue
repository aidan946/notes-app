<template>
  <div class="editor border rounded-lg">
    <div class="card card-sm rounded-lg bg-neutral text-neutral-content">
      <div class="card-body">
        <NotesTipTapBar :editor="homeEditor" />
        <div class="divider" />
        <editor-content :editor="titleEditor" />
        <editor-content
          class="mt-4"
          :editor="homeEditor"
        />
        <div class="flex space-x-2" />
        <div class="card-actions justify-end mr-0">
          <button
            class="btn btn-sm btn-primary rounded-lg"
            @click="saveNote"
          >
            <Icon name="ri:save-fill" />
          </button>
          <button
            class="btn btn-sm btn-error rounded-lg"
            @click="resetNote"
          >
            <Icon name="ri:delete-bin-7-fill" />
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import Document from '@tiptap/extension-document'
import Paragraph from '@tiptap/extension-paragraph'
import Heading from '@tiptap/extension-heading'
import Highlight from '@tiptap/extension-highlight'
import StarterKit from '@tiptap/starter-kit'
import Text from '@tiptap/extension-text'
import TaskItem from '@tiptap/extension-task-item'
import TaskList from '@tiptap/extension-task-list'
import { useEditor, EditorContent } from '@tiptap/vue-3'

const client = useSupabaseClient()
const user = useSupabaseUser()

if (!user.value) {
  navigateTo('/')
}

const titleEditor = ref(useEditor({
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
}))

const homeEditor = ref(useEditor({
  content: "Body",
  extensions: [
    Document,
    Highlight,
    StarterKit,
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
}))

async function saveNote() {
  const { error } = await client
    .from('notes')
    .insert([
      { title: titleEditor.value.getHTML(), body: editor.value.getHTML(), user_id: user.value.id },
    ])
    .select('id, title, body')
  if (error) {
    console.log('Error saving note:', error)
  }
  if (titleEditor.value) {
    titleEditor.value.chain().focus().setContent("Title").run()
  }
  if (homeEditor.value) {
    homeEditor.value.chain().focus().setContent("Body").run()
  }
}

async function resetNote() {
  if (titleEditor.value) {
    titleEditor.value.chain().focus().setContent("Title").run()
  } else {
    console.log('HomeNewNote titleEditor undefined')
  }
  if (homeEditor.value) {
    homeEditor.value.chain().focus().setContent("Body").run()
  } else {
    console.log('HomeNewNote Editor undefined')
  }
}
</script>
