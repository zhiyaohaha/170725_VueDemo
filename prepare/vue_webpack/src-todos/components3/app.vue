<template>
  <div class="todo-container">
    <div class="todo-wrap">
      <!--<todos-header @addTodo="addTodo"></todos-header>-->
      <todos-header ref="header"></todos-header>
      <list :todos="todos"></list>
      <todos-footer>
        <input type="checkbox" v-model="isAllComplete" slot="selectAll"/>
        <span slot="size">已完成{{ completeSize }} / 全部{{ todos.length }} </span>
        <button slot="clear" class="btn btn-danger" v-show="completeSize" @click="clearAllCompleted">清除已完成任务</button>
      </todos-footer>
    </div>
  </div>
</template>

<script>
  import PubSub from 'pubsub-js'
  import header from './header.vue'
  import list from './list.vue'
  import footer from './footer.vue'

  import storageUtil from '../util/storageUtil'

  export default {
    data () {
      return {
        todos: []
      }
    },

    mounted () {
      // 从local存储中读取保存的todos
      /*const todosJson = localStorage.getItem('todos_key')
      if(!todoJson) {
        return []
      } else {
        return JSON.parse(todosJson)
      }*/
      this.todos = storageUtil.readTodos()

      // 订阅消息
      PubSub.subscribe('deleteTodo', (msg, index) => {
        this.deleteTodo(index)
      })
      // PubSub.subscribe('deleteTodo', this.deleteTodo) //不可以 参数不一致

      // 使用vue的自定义事件来绑定监听
      // this.$on('addTodo', this.addTodo)  //不可以 事件监听绑定在app组件对象上
      this.$refs.header.$on('addTodo', this.addTodo) // 给header组件对象绑定监听
    },

    methods: {
      addTodo (todo) {
        this.todos.unshift(todo)
      },

      deleteTodo (index) {
        this.todos.splice(index, 1)
      },

      // 全选/全不选
      selectAll (isCheck) {
        this.todos.forEach(todo => {
          todo.complete = isCheck
        })
      },

      // 清除所有已完成的
      clearAllCompleted () {
        this.todos = this.todos.filter(todo => !todo.complete)
      }
    },

    computed: {

      completeSize () {
        /*
         // 命令式编程
         let sum = 0
         for (var i = 0; i < this.todos.length; i++) {
         var todo = this.todos[i]
         sum = sum + (todo.complete ? 1 : 0)
         return sum
         }
         return sum
         */
        // 使用声明编程
        return this.todos.reduce((preTotal, todo) => {
          return preTotal + (todo.complete ? 1 : 0)
        }, 0)
      },

      isAllComplete: {
        get () {
          return this.completeSize === this.todos.length && this.todos.length > 0
        },
        set (value) {
          this.selectAll(value)
        }
      }
    },

    watch: {
      todos: {
        deep: true, //深度监视
        /*handler: function (value) { // value是最新的todos
          // 保存todos到local
          storageUtil.saveTodos(value)
        }*/
        handler: storageUtil.saveTodos
      }
    },

    components: {
      'todos-header': header,
      list,
      'todos-footer': footer
    }
  }
</script>

<style>
  .todo-container {
    width: 600px;
    margin: 0 auto;
  }
  .todo-container .todo-wrap {
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
  }
</style>