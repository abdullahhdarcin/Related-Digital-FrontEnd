<template>
  <form @submit.prevent="handleSubmit">
    <div v-if="modal" class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">
          <label class="header">Title</label>
          <input class="input" type="text" v-model="title" required />
          <input type="checkbox" v-model="completed" />
          <p>Completed</p>
          <button class="primary-btn" type="submit" @click="save()">save</button>
          <button class="secondary-btn" @click="modalAction">cancel</button>
        </div>
      </div>
    </div>
  </form>

  <table class="flex-table">
    <thead>
      <tr>
        <th>#</th>
        <div class="title">Title</div>
        <th>Assignee</th>
        <th @click="sortBy('completed')">Status</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(table,index) in pageList" :key="table.id">
        <td>{{ table.id }}</td>
        <div class="titleinfo">{{ table.title }}</div>
        <td>{{ users.find((user) => user.id === table.userId).name }}</td>
        <td>
          <div v-if="table.completed">Done</div>
          <div v-else>In Progress</div>
        </td>

        <td>
          <button class="primary-btn" @click="modalAction(table.id)">
            Edit
          </button>
          <button class="secondary-btn" @click="deleteList(table.id,index)">
            Delete
          </button>
        </td>
      </tr>
    </tbody>
    <tbody></tbody>
  </table>
    <div class="pagination-container">
        <div class="pagination">
            <div v-for="(page, index) in pageNumbers" :key="index">    
                <button class="pagination-button" @click="changePage(index)"  v-if="index>0">
                    {{ index }}
                </button>
            </div>
        </div>
   </div>
</template>

<script>
export default {
  data() {
    return {
      list: [],
      pageList:[],
      users: [],
      pageNumbers: [],
      modal: false,
      title: '',
      tempId: null,
      completed: null
    };
  },
  mounted() {
    fetch("https://jsonplaceholder.typicode.com/todos")
      .then((res) => res.json())
      .then(
        (data) => {
          this.list = data;
          this.pageList = this.list.slice(0,10);
          this.setPages()
        }
      )
      .catch((err) => console.log(err.message));

    fetch("https://jsonplaceholder.typicode.com/users")
      .then((res) => res.json())
      .then((data) => (this.users = data))
      .catch((err) => console.log(err.message));

    fetch("https://jsonplaceholder.typicode.com/todos/")
      .then((res) => res.json())
      .then((data) => {
        this.title = data.title;
        this.completed = data.completed;
      })
      .catch(console.log("ok"));
  },
  methods: {
    modalAction(id) {
      this.tempId = id;
      if (this.modal == false) {
        this.modal = true;
      } else {
        this.modal = false;
      }
    },
    deleteList(id,index) {
      fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
        method: "DELETE",
      })
        .then(() => this.pageList.splice((index%10),1),
        console.log(this.pageList)
           )
        .catch((err) => console.log(err));
    },

    save() {
      fetch(`https://jsonplaceholder.typicode.com/todos/${this.tempId}`, {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          title: this.title,
          completed: this.completed,
        })
      })
      .catch((err) => console.log(err));
      this.modal = false;
    },
    sortBy() {
      this.pageList.sort(function (x, y) {
        return x.completed === y.completed ? 0 : x ? -1 : 1;
      });
    },
    setPages() {
      this.pageNumbers = Array.from({length: this.list.length/10}, (_, i) => i + 1)
      
    },
    changePage(pageNumber) {
        this.pageList = this.list.slice(10*(pageNumber-1),pageNumber*10)

    }
  },
};
</script>
<style>
</style>