<template>
  <div class="h-full divide-y divide-solid">
    <div class="drawer lg:drawer-open">
      <label for="my-drawer-2" class="drawer-overlay">
      </label>
      <ul class="menu p-4 w-48 bg-base-100 text-base-content">
        <!-- Sidebar content here -->
        <li @click="$emit('addNote')">
          <a>Add Note</a>
        </li>
        <li>
          <NuxtLink to="/notes/home">Home</NuxtLink>
        </li>
        <li>
          <NuxtLink to="/notes/my-notes">My Notes</NuxtLink>
        </li>
      </ul>
      <div class="divider"></div>
      <div class="menu">
        <li class="flex">
          <a>My Tags: <button class="btn btn-ghost btn-circle avatar" @click="$emit('addTag')">+</button></a>
        </li>
        <li class="flex" v-for="tag in tags" :key="tag.id">
          <a @click="$emit('filterNotes', tag.id)">{{ tag.name }}</a>
        </li>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
export default {
  name: "MenuSideBar",
  emits: ['addNote', 'addTag', 'filterNotes'],
  data() {
    return {
      edit: false,
      tags: {},
    }
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
    submit() {
      this.$emit('addNote')
      this.$emit('addTag')
      this.$emit('filterNotes')
    }
  }
}
</script>
