<template>
  <div v-for="(todo, index) in todos" v-bind:key="todo.id" class="card mt-2">
    <div class="card-body p-2 d-flex align-items-center">
      <div class="form-check flex-grow-1">
        <input
          class="form-check-input"
          type="checkbox"
          v-model="todo.completed"
        />
        <label
          :class="{ 'todo-done': todo.completed }"
          class="form-check-label"
        >
          {{ todo.subject }}
        </label>
      </div>
      <div>
        <button class="btn btn-danger btn-sm" @click="deleteTodo(index)">
          Delete
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  // props: ['todos'] 이것만 해도 되지만, 타입을 정해주고 입력필수 여부를 정해주면 더 낫다.
  props: {
    todos: {
      type: Array,
      required: true,
    },
  },
};
</script>

<style></style>
