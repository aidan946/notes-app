<template>
  <div class="note">
    <h1>{{ title }}</h1>
    <h3>{{ tag }}</h3>
    <p>
      {{ body }}
    </p>
    <div class="centre-buttons">
      <button @click="editNote">Edit</button>
      <button @click="$emit('deleteNote')">Delete</button>
    </div>
  </div>
</template>

<script>
export default{
  name: "Note",
  props: {
    title: String, 
    tag: String,
    body: String
  },
  methods: {
    submit() {
      this.$emit('deleteNote')
    },
    editNote() {

    }
  }
}
</script>

<style>
  body{
    max-width: 30rem;
  }
  .centre-buttons{
    text-align: center;
  }
  .centre-buttons button{
  margin: 1rem;
  }

  .note{
    padding: 0.5rem;
    border: 1px solid black;
  }
  
</style>