<template>
  <div>
    <HeaderBar />
    <div class="flex w-screen">
      <div class="min-w-64 w-64 max-w-64">
        <NotesMenuSideBar
          @add-note="addNote"
          @add-tag="addTag"
          @filter-notes="filterNotes"
        />
      </div>
      <div
        v-if="addNewNote"
        class="absolute
        inset-0
        flex
        items-center
        justify-center
        bg-gray-700 bg-opacity-50
        z-10"
      >
        <div class="card card-sm rounded-lg w-fit bg-neutral text-neutral-content">
          <div class="card-body">
            <div class="relative">
              <h1 class="text-2xl">
                Add New Note
              </h1>
              <a
                class="btn btn-sm btn-circle absolute right-0 top-0"
                @click="addNote"
              >
                ✕
              </a>
            </div>
            <div class="bg-gray-700 rounded p-4">
              <NotesTipTapBar :editor="editor" />
              <div class="divider" />
              <editor-content :editor="titleEditor" />
              <editor-content
                class="mt-4"
                :editor="editor"
              />
            </div>
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
      <div
        v-if="addNewTag"
        class="absolute inset-0 flex items-center justify-center bg-gray-700 bg-opacity-50 z-10"
      >
        <NotesTagModal @add-tag="addTag" />
      </div>
      <slot />
    </div>
  </div>
</template>

<script setup lang="ts">
import Document from "@tiptap/extension-document";
import Paragraph from "@tiptap/extension-paragraph";
import Heading from "@tiptap/extension-heading";
import Highlight from "@tiptap/extension-highlight";
import StarterKit from "@tiptap/starter-kit";
import Text from "@tiptap/extension-text";
import TaskItem from "@tiptap/extension-task-item";
import TaskList from "@tiptap/extension-task-list";
import { useEditor, EditorContent } from "@tiptap/vue-3";

useHead({
	title: "Note-It",
	meta: [{ name: "description", content: "Note your plans!." }],
});

useSeoMeta({
	ogImage: "/note-logo.png",
});

definePageMeta({
	middleware: "auth",
});

const client = useSupabaseClient();
const user = useSupabaseUser();

const addNewNote = ref(false);
const addNewTag = ref(false);

const notes = useState("databaseNotes");

const titleEditor = ref(
	useEditor({
		content: "Title",
		extensions: [
			Document,
			Heading.configure({
				HTMLAttributes: {
					class: "card-title text-2xl bold mb-3",
				},
			}),
			Text,
		],
		editorProps: {
			attributes: {
				class: "prose focus:outline-none",
			},
		},
	}),
);

const editor = ref(
	useEditor({
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
				HTMLAttributes: {},
			}),
			Text,
		],
		editorProps: {
			attributes: {
				class: "prose focus:outline-none pb-2",
			},
		},
	}),
);

function addNote() {
	addNewNote.value = !addNewNote.value;
}
function addTag() {
	addNewTag.value = !addNewTag.value;
}

async function saveNote() {
	if (user.value && titleEditor.value && editor.value) {
		await client
			.from("notes")
			.insert([
				{
					title: titleEditor.value.getHTML(),
					body: editor.value.getHTML(),
					user_id: user.value.id,
				},
			])
			.select("id, title, body");
		titleEditor.value.chain().focus().setContent("Title").run();
		editor.value.chain().focus().setContent("Body").run();
	}
}

async function resetNote() {
	if (titleEditor.value && editor.value) {
		titleEditor.value.chain().focus().setContent("Title").run();
		editor.value.chain().focus().setContent("Body").run();
	}
}

async function filterNotes(id: number) {
	const { data: noteTags } = await client
		.from("note-tags")
		.select("notes(id, title, body)")
		.eq("tag_id", id);
	if (noteTags) {
		notes.value = noteTags.map(function (note) {
			return note["notes"];
		});
	}
}
</script>
