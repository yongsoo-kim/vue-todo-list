<template>
  <form v-on:submit.prevent="onSubmit">
    <div class="d-flex">
      <div class="flex-grow-1 mr-2">
        <input
          class="form-control"
          type="text"
          v-model="todo"
          placeholder="Type new to-do"
        />
      </div>
      <div>
        <button class="btn btn-primary" type="submit">Add</button>
      </div>
    </div>
    <div v-show="hasError" style="color: red">This field cannot be empty.</div>
  </form>
</template>

<script>
import { ref } from "vue";
export default {
  emits: ["add-todo"],

  setup(props, { emit }) {
    const todo = ref("");
    const hasError = ref(false);

    const onSubmit = () => {
      //e.preventDefault(); //Form의 디폴트 기능인 새로고침 기능을 막는다.
      if (todo.value === "") {
        hasError.value = true;
      } else {
        // 자식 컴포넌트에서 부모컴포넌트에서 자료를 보냄. 이때 이벤트이름, 보내고 싶은 데이터을 짝지어 emit에서 호출한다. ("add-todo")
        emit("add-todo", {
          id: Date.now(),
          subject: todo.value,
          completed: false,
        });

        hasError.value = false;
        todo.value = "";
      }
    };

    return {
      todo,
      hasError,
      onSubmit,
    };
  },
};
</script>

<style></style>
