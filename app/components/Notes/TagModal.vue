<template>
  <div class="card card-sm rounded-lg w-96 bg-neutral text-neutral-content">
    <div class="card-body">
      <div class="relative">
        <h1 class="text-2xl">
          Tags
        </h1>
        <a
          class="btn btn-sm btn-circle absolute right-0 top-0"
          @click="$emit('addTag')"
        >
          ✕
        </a>
      </div>
      <div class="bg-gray-700 rounded p-4">
        <div>
          Add new tag:
          <div class="mt-4 flex justify-center">
            <input
              v-model="tagName"
              type="text"
              placeholder="Type here"
              class="input w-48 max-w-xs"
              @keyup.enter="createNewTag(tagName)"
            />
            <button
              class="btn btn-sm btn-success rounded-lg ml-4 mt-2"
              @click="createNewTag(tagName)"
            >
              Add
            </button>
          </div>
        </div>
        <div class="divider"></div>
        <div class="pb-1">
          My Tags:
        </div>
        <ul
          v-for="tag in allModalTags"
          :key="tag.id"
          class="menu menu-sm"
        >
          <li>
            <div class="flex">
              <a>{{ tag.name }}</a>
              <button
                class="btn btn-sm btn-error rounded-lg mr-0 ml-auto"
                @click="deleteTag(tag.id)"
              >
                <Icon name="ri:delete-bin-7-fill" />
              </button>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
const client = useSupabaseClient()
const user = useSupabaseUser()
const tagName = ref('')

const { data: allModalTags } = await useAsyncData('tags', async () => {
  if (user && user.value) {
    const { data } = await client.from('tags').select('id, name').eq('user_id', user.value.id)
    if (data) {
      return data
    }
  }
})

async function createNewTag(name: string) {
  if (user && user.value) {
    const { data } = await client
      .from('tags')
      .insert([
        { name: name, user_id: user.value.id },
      ])
      .select('id, name')
    if (data && allModalTags.value) {
      allModalTags.value.push(data[0])
    }
  }
}

async function deleteTag(id: number) {
  if (allModalTags.value) {
    await client
      .from('tags')
      .delete()
      .eq('id', id)
    let newTags = allModalTags.value.filter(i => i.id != id)
    allModalTags.value = newTags
  }
}
</script>
