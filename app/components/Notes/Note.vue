<template>
  <div>
    <div
      v-if="loadModal"
      class="
        absolute
        inset-0
        flex
        items-center
        justify-center
        bg-gray-700 bg-opacity-50
        z-10
      "
    >
      <div class="card card-sm rounded-lg w-fit bg-neutral text-neutral-content">
        <div class="card-body">
          <div class="relative">
            <editor-content :editor="titleEditor" />
            <a
              class="btn btn-sm btn-circle absolute right-0 top-0"
              @click="toggleModal"
            >
              ✕
            </a>
          </div>
          <div class="bg-gray-700 rounded p-4">
            <NotesTipTapBar :editor="editor" />
            <div class="divider" />
            <editor-content :editor="editor" />
          </div>
          <div class="flex space-x-2">
            <div
              v-for="tag in tags"
              :key="tag.id"
            >
              <div class="group badge badge-primary mt-4 p-3">
                {{ tag.name }}
                <button
                  class="hidden group-hover:flex btn btn-circle btn-outline btn-xs btn-error ml-1"
                  @click="deleteNoteTag(tag.id)"
                >
                  <Icon name="ri:close-line" class="h-4 w-4" />
                </button>
              </div>
            </div>
            <select
              v-model="selectedTag"
              class="select select-info select-xs w-24 mt-4"
              @change="addNoteTag($event)"
            >
              <option
                :key="'default'"
                disabled
                selected
              >
                Add Tag
              </option>
              <option
                v-for=" tag in otherTags "
                :key="tag.id"
                :value="tag.id"
              >
                {{ tag.name }}
              </option>
            </select>
          </div>
          <div class="card-actions justify-end mr-0">
            <button
              class="btn btn-sm btn-primary rounded-lg"
              @click="submitEdit"
            >
              <Icon name="ri:save-fill" />
            </button>
            <button
              class="btn btn-sm btn-error rounded-lg"
              @click="$emit('deleteNote', id)"
            >
              <Icon name="ri:delete-bin-7-fill" />
            </button>
          </div>
        </div>
      </div>
    </div>
    <div
      v-else
      class="card card-sm rounded-lg w-96 bg-neutral text-neutral-content v-0"
    >
      <div class="card-body">
        <div class="flex">
          <editor-content :editor="titleEditor" />
        </div>
        <editor-content :editor="editor" />
        <div class="flex space-x-2">
          <div
            v-for=" tag in tags "
            :key="tag.id"
          >
            <div class="badge badge-primary">
              {{ tag.name }}
            </div>
          </div>
        </div>
        <div class="card-actions justify-end mr-0">
          <button
            class="btn btn-sm btn-success rounded-lg"
            @click="toggleModal"
          >
            <Icon name="ri:edit-box-fill" />
          </button>
          <button
            class="btn btn-sm btn-primary rounded-lg"
            @click="submitEdit"
          >
            <Icon name="ri:save-fill" />
          </button>
          <button
            class="btn btn-sm btn-error rounded-lg"
            @click="$emit('deleteNote', id)"
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

const props = defineProps({
  id: {
    type: Number,
    default: null
  },
  title: {
    type: String,
    default: ''
  },
  body: {
    type: String,
    default: ''
  },
})
defineEmits(['deleteNote'])
const loadModal = ref(false)
const selectedTag = ref('Add Tag')
const noteID = props.id
const noteTitle = props.title
const noteBody = props.body

let { data: noteTags } = await client.from('note-tags').select('tags(id, name, color)').eq('note_id', noteID)
if (noteTags) {
  noteTags = noteTags.map(function (tag) {
    return tag['tags']
  })
}
const tagFilter = '(' + noteTags.map(tag => tag.id).join(',') + ')';
const { data: allTags } = await client.from('tags').select('id, name, color').not('id', 'in', tagFilter)
const tags = ref(noteTags)
const otherTags = ref(allTags)

const titleEditor = ref(useEditor({
  content: noteTitle,
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

const editor = ref(useEditor({
  content: noteBody,
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

function toggleModal() {
  loadModal.value = !loadModal.value
}

async function submitEdit() {
  if (titleEditor.value && editor.value) {
    const supabase = useSupabaseClient()
    await supabase
      .from('notes')
      .update({ title: titleEditor.value.getHTML(), body: editor.value.getHTML() })
      .eq('id', noteID)
  }
  // Flash success here
}
async function addNoteTag(event: any) {
  const tag_id: number = parseInt(event.target.value)
  const { error } = await client
    .from('note-tags')
    .insert([
      { note_id: noteID, tag_id: tag_id },
    ])
  if (error) console.log(error)
  tags.value.push(otherTags.value.find(tag => tag.id === tag_id))
  otherTags.value = otherTags.value.filter(i => i.id != tag_id)
  selectedTag.value = 'default'
}


async function deleteNoteTag(tagID: number) {
  const { error } = await client.from('note-tags').delete().match({ 'note_id': noteID, 'tag_id': tagID })
  otherTags.value.push(tags.value.find(tag => tag.id === tagID))
  tags.value = tags.value.filter(i => i.id != tagID)
  if (error) console.log(error)
}
</script>
