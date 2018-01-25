<template>
  <section class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input class="new-todo" v-model="newTodoTitle" @keyup.enter="createTodo()" placeholder="What needs to be done?" autofocus>
    </header>
    <!-- 如果有todos=>显示 -->
    <section class="main" v-if="todos.length">
      <input class="toggle-all" type="checkbox">
      <label for="toggle-all" @click="toggleAll">Mark all as complete</label>
      <ul class="todo-list">
        <!-- 加 class `editing` 当某项任务 completed -->
        <div v-for="(todo, index) in todosInView">
          <todo-item :todo="todo" @toggleCompleted="toggleCompleted(index)" @removeSelf="removeTodo(index)" />
        </div>
      </ul>
    </section>
    <!-- 默认隐藏的footer -->
    <todo-footer v-if="todos.length" :itemsLeft="remaining.length" :currentView="currentView" :clearCompleted="clearCompleted" />
  </section>
</template>

<script lang="ts">
import TodoFooter from './TodoFooter.vue';
import TodoHeader from './TodoHeader.vue';
import TodoItem from './TodoItem.vue';

import Vue from 'vue';
import { AppState, Todo } from '../models';

export default Vue.extend({
  components: {
    TodoItem,
    TodoFooter,
  },

  props: ['currentView'],

  data() {
    const initialState: AppState = {

      // Input box content.
      newTodoTitle: '',

      // Current todo items.
      todos: [
        { completed: false, title: 'Use Vue with TypeScript' },
      ],
    };

    return initialState;
  },
  methods: {
    /**
     * 新增Todo, 重置title.
     */
    createTodo() {
      const title = this.newTodoTitle.trim();
      if (!title) {
        return;
      }

      this.todos.push({
        completed: false,
        title,
      });

      this.newTodoTitle = '';
    },

    /**
     * 移除 Todo 给定的index.
     */
    removeTodo(index: number) {
      if (index >= this.todos.length) {
        throw new Error(`Index deletion at ${index} greater than ${this.todos.length}`);
      }

      this.todos.splice(index, 1);
    },

    /**
     * 切换 Todo's `completed`状态.
     */
    toggleCompleted(index: number) {
      const current = this.todos[index];
      this.todos.splice(index, 1, {
        ...current,
        completed: !current.completed
      });
    },

    /**
     *  切换 Todo's `completed`状态.
     */
    clearCompleted() {
      this.todos = this.remaining;
    },

    /**
     * 一次性切换多条状态
     */
    toggleAll() {
      const stateForAll = this.completed.length !== this.todos.length;

      for (const todo of this.todos) {
        todo.completed = stateForAll;
      }
    }
  },

  computed: {
    todosInView(): Todo[] {
      switch (this.currentView) {
        case 'completed':
          return this.completed;
        case 'active':
          return this.remaining;
        case 'all':
        default:
          return this.todos;
      }
    },
    completed(): Todo[] {
      return this.todos.filter(isCompleted);
    },
    remaining(): Todo[] {
      return this.todos.filter(isNotCompleted);
    },
  },
});

function isCompleted(todo: Todo) {
  return todo.completed;
}

function isNotCompleted(todo: Todo) {
  return !todo.completed;
}
</script>