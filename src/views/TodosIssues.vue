<template>
  <div>
    <h1>todoリスト</h1>
    <!-- todo入力フォーム -->
    <form @submit.prevent="addTodo()">
      <el-input placeholder="todo" v-model="todo"></el-input>
    </form>
    <el-row :gutter="12">
      <!-- todo表示エリア -->
      <TodoItem 
        v-for="(todo, index) in todos" 
        :key="index" 
        :todo="todo"
        :index="index"
        @removeTodo="removeTodo"
      />
      <!-- issue表示エリア -->
      <TodoItem 
        v-for="(issue, index) in issues" 
        :key="issue.id" 
        :todo="issue.title"
        :index="index"
        @removeTodo="closeIssue"
      />
    </el-row>
  </div>
</template>

<script>
import axios from 'axios';
import TodoItem from '@/components/TodoItem';

const client = axios.create({
  baseURL: 'https://api.github.com/repos/diveintocode-corp/vue_seriese_api',
  headers: {
    'Authorization': `token ${process.env.VUE_APP_GITHUB_TOKEN}`,
    'Accept': 'application/vnd.github.v3+json',
    'Content-Type':'application/json',
  },
})

export default {
  name: 'TodosIssues',
  components: {
    TodoItem
  },
  data () {
    return {
      todo: '',
      todos: [],
      issues: []
    }
  },
  methods: {
    addTodo(){
      this.todos.push(this.todo);
      this.todo= '';
    },
    removeTodo(index){
      this.todos.splice(index, 1);
    },
    closeIssue(index){
      const target = this.issues[index];
      client.patch(`/issues/${target.number}`, {
            state: 'closed'
          })
        .then(() => {
          this.issues.splice(index, 1);
        })
    },
    getIssues() {
      client.get('/issues')
        .then((res) => {
          this.issues = res.data;
        })
    }
  },
  created() {
    this.getIssues();
  }
}
</script>

