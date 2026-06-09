<template>
  <div class="flex text-lg h-8">
    <div v-for="(item, index) in items" :key="item.icon" class="p-1">
      <div v-if="item.type === 'divider'" :key="`divider${index}`">
        |
      </div>
      <button class="menu-item" :class="{ 'is-active': item.isActive ? item.isActive() : null }" :title="item.title"
        @click="item.action">
        <Icon :name="`ri:${item.icon}`" />
      </button>
    </div>
  </div>
</template>

<script setup lang='ts'>
const props = defineProps({ editor: { type: Object, required: true } })
let editor = ref(props.editor)
let items = ref([])
if (editor.value) {
  items.value = [
    {
      icon: 'bold',
      title: 'Bold',
      action: () => editor.value.chain().focus().toggleBold().run(),
      isActive: () => editor.value.isActive('bold'),

    },
    {
      icon: 'italic',
      title: 'Italic',
      action: () => editor.value.chain().focus().toggleItalic().run(),
      isActive: () => editor.value.isActive('italic'),

    },
    {
      icon: 'strikethrough',
      title: 'Strike',
      action: () => editor.value.chain().focus().toggleStrike().run(),
      isActive: () => editor.value.isActive('strike'),

    },
    {
      icon: 'code-view',
      title: 'Code',
      action: () => editor.value.chain().focus().toggleCode().run(),
      isActive: () => editor.value.isActive('code'),

    },
    {
      icon: 'mark-pen-line',
      title: 'Highlight',
      action: () => editor.value.chain().focus().toggleHighlight().run(),
      isActive: () => editor.value.isActive('highlight'),

    },
    {
      type: 'divider',
    },
    {
      icon: 'h-1',
      title: 'Heading 1',
      action: () => editor.value.chain().focus().toggleHeading({ level: 1 }).run(),
      isActive: () => editor.value.isActive('heading', { level: 1 }),

    },
    {
      icon: 'h-2',
      title: 'Heading 2',
      action: () => editor.value.chain().focus().toggleHeading({ level: 2 }).run(),
      isActive: () => editor.value.isActive('heading', { level: 2 }),

    },
    {
      icon: 'paragraph',
      title: 'Paragraph',
      action: () => editor.value.chain().focus().setParagraph().run(),
      isActive: () => editor.value.isActive('paragraph'),

    },
    {
      icon: 'list-unordered',
      title: 'Bullet List',
      action: () => editor.value.chain().focus().toggleBulletList().run(),
      isActive: () => editor.value.isActive('bulletList'),

    },
    {
      icon: 'list-ordered',
      title: 'Ordered List',
      action: () => editor.value.chain().focus().toggleOrderedList().run(),
      isActive: () => editor.value.isActive('orderedList'),

    },
    {
      icon: 'list-check-2',
      title: 'Task List',
      action: () => editor.value.chain().focus().toggleTaskList().run(),
      isActive: () => editor.value.isActive('taskList'),

    },
    {
      icon: 'code-box-line',
      title: 'Code Block',
      action: () => editor.value.chain().focus().toggleCodeBlock().run(),
      isActive: () => editor.value.isActive('codeBlock'),

    },
    {
      type: 'divider',
    },
    {
      icon: 'double-quotes-l',
      title: 'Blockquote',
      action: () => editor.value.chain().focus().toggleBlockquote().run(),
      isActive: () => editor.value.isActive('blockquote'),

    },
    {
      icon: 'separator',
      title: 'Horizontal Rule',
      action: () => editor.value.chain().focus().setHorizontalRule().run(),
    },
    {
      type: 'divider',
    },
    {
      icon: 'text-wrap',
      title: 'Hard Break',
      action: () => editor.value.chain().focus().setHardBreak().run(),
    },
    {
      icon: 'format-clear',
      title: 'Clear Format',
      action: () => editor.value.chain().focus().clearNodes().unsetAllMarks().run(),
    },
    {
      type: 'divider',
    },
    {
      icon: 'arrow-go-back-line',
      title: 'Undo',
      action: () => editor.value.chain().focus().undo().run(),
    },
    {
      icon: 'arrow-go-forward-line',
      title: 'Redo',
      action: () => editor.value.chain().focus().redo().run(),
    },
  ]
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
