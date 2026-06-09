<template>
  <div class="navbar bg-base-100 w-screen">
    <div class="navbar-start">
    </div>
    <div class="navbar-center">
      <NuxtLink
        class="btn btn-ghost normal-case text-xl"
        to="/notes/home"
      >
        Notes
      </NuxtLink>
      <span>|</span>
      <NuxtLink
        class="btn btn-ghost normal-case text-xl"
        to="/kanban/home"
      >
        Kanban
      </NuxtLink>
    </div>
    <div class="navbar-end">
      <button class="btn btn-ghost btn-circle">
        <Icon name="ri:search-line" class="h-5 w-5" />
      </button>
      <div class="flex-none gap-2">
        <div class="dropdown dropdown-end">
          <label
            tabindex="0"
            class="btn btn-ghost btn-circle avatar"
          >
            <div class="w-10 rounded-full">
              <img src="https://avatars.githubusercontent.com/u/68595713?s=40&v=4" />
            </div>
          </label>
          <ul
            tabindex="0"
            class="mt-3 p-2 shadow menu menu-compact dropdown-content bg-base-100 rounded-box w-52"
          >
            <li>
              <NuxtLink to="/settings">
                Settings
              </NuxtLink>
            </li>
            <li><a @click="logout">Logout</a></li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
const client = useSupabaseClient()

async function logout() {
  let { error } = await client.auth.signOut()
  if (error) {
    console.log(error)
  } else {
    navigateTo('/')
  }
}
</script>
