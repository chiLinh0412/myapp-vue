<template lang="html">
  <div>
    <input type="text" class="todo-input" placeholder="Enter your work !"
    v-model="newTodo" @keyup.enter="addTodo">
    <div v-for="(todo, index) in todosFiltered" :key="todo.id" class="todo-item">
      <div class="todo-item-left">
        <input type="checkbox" v-model="todo.completed">
        <div v-if="!todo.editing" @dblclick="editTodo(todo)" class="todo-item-label" :class="{ completed : todo.completed }">{{todo.title}}</div>
        <input v-else class="todo-item-edit" type="text" v-model="todo.title" @blur="doneEdit(todo)" 
        @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)" v-focus>
      </div>
      <div class="remove-item" @click="removeTodo(index)">
        &times;
      </div>
    </div>

    <div class="extra-container">
      <div><label><input type="checkbox" :checked="!anyRemaining"
      @change="checkAllTodos"> Check All </label></div>
      <div>{{ remaining}} items left</div>
    </div>

    <div class="extra-container">
      <div>
        <button :class="{ active: filter == 'all' }" @click="filter = 'all'">All</button>
        <button :class="{ active: filter == 'active' }" @click="filter = 'active'">Active</button>
        <button :class="{ active: filter == 'completed' }" @click="filter = 'completed'">Completed</button>
      </div>
      <div>
        <button v-if="showClearCompleted" @click="clearCompleted">
        Clear Completed
        </button>
      </div>
    </div>
  </div>
</template>

<script lang="js">
import axios from 'axios'

  export default  {
    name: 'todos',
    data () {
      return {
        newTodo: '',
        idForTodo: 3,
        beforeEditCache: '',
        filter: 'all',
        todos: [
          {
            'id': 1,
            'title': 'Finish vuejs',
            'completed': false,
            'editing': false,
          },
          {
            'id': 2,
            'title': 'Progress spring',
            'completed': false,
            'editing': false,
          },
        ]
      }
    },
    computed: {
      remaining() {
        return this.todos.filter(todo => todo.completed != true).length
      },
      anyRemaining() {
        return this.remaining != 0
      },
      todosFiltered() {
        if(this.filter == 'all') {
          return this.todos
        } else if (this.filter == 'active') {
          return this.todos.filter(todo => todo.completed != true)
        } else if (this.filter == 'completed') {
          return this.todos.filter(todo => todo.completed == true)
        }
        return this.todos
      },
      showClearCompleted() {
        return this.todos.filter(todo => todo.completed == true).length > 0
      } 
    },
    directives: {
      focus: {
        inserted: function (el) {
          el.focus()
        }
      }
    },
    methods: {
      addTodo() {
        if(this.newTodo.trim().length == 0){
          return
        }
        this.todos.push({
          id: this.idForTodo,
          title: this.newTodo,
          completed: false,
        })
        this.newTodo = ''
        this.idForTodo++
      },
      editTodo(todo){
        this.beforeEditCache = todo.title
        todo.editing = true
      },
      doneEdit(todo){
        if(todo.title.trim() == ''){
          todo.title = this.beforeEditCache
        }
        todo.editing = false
      },
      cancelEdit(todo) {
        todo.title = this.beforeEditCache
        todo.editing = false
      },
      removeTodo(index) {
        this.todos.splice(index, 1)
      },
      checkAllTodos() {
        this.todos.forEach((todo) => todo.completed = event.target.checked);
      },
      clearCompleted() {
        this.todos = this.todos.filter(todo => todo.completed != true)
      }
    },

    // lance db.json by command "json-server db.json"
    mounted (){
    axios
      .get('http://localhost:3000/todolist')
      .then(response => {
        this.info = response.data.id
      })
      .catch(error => {
        console.log(error)
        this.errored = true
      })
      .finally(() => this.loading = false)
    },
    created() {
    const data = { title: 'happy'}
    axios.post('http://localhost:3000/todolist', data)
    .then(response => {
        this.info = response.data.id
      })
    .catch(e => {
      this.errors.push(e)
    })
  }
  }


</script>

<style scoped lang="scss">
  .todo-input {
    width: 100%;
    padding: 10px 18px;
    font-size: 18px;
    margin-bottom: 16px;

    &:focus {
      outline: 0;
    }
  }

  .todo-item {
    margin-bottom: 12px;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .remove-item {
    cursor: pointer;
    margin-left: 14px;
    &:hover {
      color: black;
    }
  }

  .todo-item-left {
    display: flex;
    align-items: center;
  }

  .todo-item-label {
    padding: 10px;
    border: 1px solid white;
    margin-left: 12px;
  }

  .todo-item-edit {
    font-size: 12px;
    color: #2c3e50;
    margin-left: 12px;
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    font-family: 'Avenir', Arial, Helvetica, sans-serif;

    &:focus {
      outline: none;
    }
  }

  .completed {
    text-decoration: line-through;
    color: gray;
  }

  .extra-container {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 16px;
    border-top: 1px solid lightgray;
    padding-top: 14px;
    margin-bottom: 14p;
  }

  button {
    font-size: 14px;
    background-color: white;
    appearance: none;

    &:hover {
      background: lightgreen;
    }

    &:focus {
      outline: none;
    }
  }

  .active {
    background: lightgreen;
  }

</style>
