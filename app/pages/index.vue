<template>
  <div class="w-screen" data-theme="night">
    <form class="row flex-center flex" @submit.prevent="login">
      <div class="flex min-h-screen justify-center items-center ml-auto mr-auto">
        <div class="card bg-neutral shadow-2xl p-24 mr-32">
          <h1 class="font-mono text-8xl">Note-It!</h1>
        </div>
        <div class="flex min-h-screen justify-center items-center ml-auto mr-auto">
          <div class="form-widget card bg-neutral shadow-2xl">
            <div class="card-body">
              <input v-model="email" class="input" type="email" placeholder="Your email" />
              <br />
              <input v-model="password" class="input" type="password"
                placeholder="Your password" />
              <br />
              <span class="text-error">{{ supabaseError }}</span>

              <div class="card-actions">
                <button class="btn btn-primary" type="submit">Login</button>
                <button class="btn btn-outline btn-primary ml-2" type="button" @click="signup">
                  Sign Up
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </form>
  </div>
</template>

<script setup lang="ts">
const client = useSupabaseClient();
const user = useSupabaseUser();

if (user.value) navigateTo("/notes/home");

const email = ref("");
const password = ref("");
const supabaseError = ref("");

async function login() {
	console.log("Hit");
	const { error } = await client.auth.signInWithPassword({
		email: email.value,
		password: password.value,
	});
	if (error) console.log(error);
	if (error) {
		supabaseError.value = error.message;
	} else {
		navigateTo("/notes/home");
	}
}
async function signup() {
	try {
		const { error } = await client.auth.signUp({
			email: email.value,
			password: password.value,
		});
		if (error) throw error;
	} catch (error: unknown) {
		supabaseError.value = error.error_description || error.message;
	}
}
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Noto+Sans&display=swap");

body {
  font-family: "Noto Sans", sans-serif;
  background-color: rgb(15 23 41);
}

body,
html {
  padding: 0;
  margin: 0;
  width: 100%;
  min-height: 100vh;
}
</style>
