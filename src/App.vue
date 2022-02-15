<template>
  <div class="container">
    <h1>To-Do List</h1>

    <input
      class="form-control"
      type="text"
      v-model="searchText"
      placeholder="Search"
      @keyup.enter="searchTodo"
    />

    <hr />

    <TodoSimpleForm @add-todo="addTodo" />
    <div style="color: red">{{ error }}</div>

    <div v-if="!todos.length">There is nothing to display.</div>

    <TodoList
      :todos="todos"
      @toggle-todo="toggleTodo"
      @delete-todo="deleteTodo"
    />

    <hr />
    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li v-if="currentPageNo !== 1" class="page-item">
          <a
            style="cursor: pointer"
            class="page-link"
            @click="getTodos(currentPageNo - 1)"
            >Previous</a
          >
        </li>
        <li
          v-for="page in numberOfPages"
          :key="page"
          class="page-item"
          :class="currentPageNo === page ? 'active' : ''"
        >
          <a class="page-link" href="#" @click="getTodos(page)">{{ page }}</a>
        </li>
        <li v-if="numberOfPages !== currentPageNo" class="page-item">
          <a
            style="cursor: pointer"
            class="page-link"
            href="#"
            @click="getTodos(currentPageNo + 1)"
            >Next</a
          >
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
import { ref, computed, watch } from "vue";
import TodoSimpleForm from "./components/TodoSimpleForm.vue";
import TodoList from "./components/TodoList.vue";
import axios from "axios";

export default {
  components: {
    TodoSimpleForm,
    TodoList,
  },

  setup() {
    const todos = ref([]);
    const error = ref("");

    // Pagination values
    const numberOfTodos = ref(0);
    const pageLimit = 5;
    const currentPageNo = ref(1);

    const searchText = ref("");

    const numberOfPages = computed(() => {
      return Math.ceil(numberOfTodos.value / pageLimit);
    });

    const getTodos = async (page = currentPageNo.value) => {
      currentPageNo.value = page;

      error.value = "";
      try {
        const res = await axios.get(
          `http://localhost:3000/todos?_sort=id&_order=desc&subject_like=${searchText.value}&_page=${page}&_limit=${pageLimit}`
        );
        numberOfTodos.value = res.headers["x-total-count"];
        todos.value = res.data;
      } catch (err) {
        console.log(err);
        error.value = "Something went wrong";
      }
    };

    const addTodo = async (todo) => {
      error.value = "";
      try {
        await axios.post("http://localhost:3000/todos", {
          subject: todo.subject,
          completed: todo.completed,
        });
        getTodos(1);
      } catch (err) {
        console.log(err);
        error.value = "Something went wrong";
      }
    };

    // *** TODO를 저장하는 방법 1. 이방법은 'then'안에서도 비동기함수를 부를수 있기에 콜백지옥을 야기할수 있다.
    // const addTodo = (todo) => {
    //   error.value = "";

    //   //데이터 베이스에 투두 저장.
    //   //'post'만 쓰면 비동기 로 움직여져 버린다. 따라서 콜백함수로 이용해 이를 동기화 한다.
    //   //Promise는 정해진 장시간의 기능을 수행한 후,
    //   // 정상적으로 기능이 수행되었다면 성공의 메세지와 함께 결과값을 전달하고
    //   // 문제가 발생할 경우엔 에러를 전달해주게 된다.

    //   // 간단히 말하면,
    //   // 비동기에서 성공과 실패를 분리해서 메서드를 수행한다. 라고 할 수 있겠다.
    //   axios
    //     .post("http://localhost:3000/todos", {
    //       subject: todo.subject,
    //       completed: todo.completed,
    //     })
    //     .then((res) => {
    //       console.log(res);
    //       todos.value.push(todo);
    //     })
    //     .catch((err) => {
    //       console.log(err);
    //       error.value = "Something went wrong.";
    //     });
    // };

    const toggleTodo = async (index) => {
      error.value = "";

      const id = todos.value[index].id;
      try {
        await axios.patch("http://localhost:3000/todos/" + id, {
          completed: !todos.value[index].completed,
        });
        todos.value[index].completed = !todos.value[index].completed;
      } catch (err) {
        console.log(err);
        error.value = "Something went wrong.";
      }
    };

    const deleteTodo = async (index) => {
      error.value = "";
      const id = todos.value[index].id;
      try {
        await axios.delete("http://localhost:3000/todos/" + id);
        getTodos(1);
      } catch (err) {
        console.log(err);
        error.value = "Something went wrong.";
      }
    };

    let timeout = null;

    const searchTodo = () => {
      clearTimeout(timeout);
      getTodos(1);
    };

    watch(searchText, () => {
      clearTimeout(timeout);
      timeout = setTimeout(() => {
        getTodos(1);
      }, 2000);
    });

    // const filteredTodos = computed(() => {
    //   // 만일 검색단어가 공백이 아니라면...
    //   if (searchText.value) {
    //     return todos.value.filter((todo) => {
    //       return todo.subject.includes(searchText.value);
    //     });
    //   }

    //   return todos.value;
    // });

    getTodos();

    return {
      todos,
      addTodo,
      deleteTodo,
      toggleTodo,
      searchText,
      //filteredTodos,
      error,
      numberOfPages,
      currentPageNo,
      getTodos,
      searchTodo
    };
  },
};
</script>

<style>
.todo-done {
  color: "gray";
  text-decoration: line-through;
}
</style>
