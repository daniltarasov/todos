<template>
  <div class="container">
    <div class="col-sm-10">
      <h1>Задачи</h1>
      <table class="table table-dark table-stripped table-hover col-sm-10">
        <thead class="thead-light">
          <tr>
            <th>Выполнено задач</th>
            <th>Не выполнено</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>?</td>
            <td>?</td>
          </tr>
        </tbody>
      </table>

      <confirmation :messag="confirmationMessage"
      :showDisAlert="showDismissibleAlert" v-on:closed="closeAlert"></confirmation>

      <button type="button" id="task-add" class="btn btn-success btn-sm
        align-left d-block"
        v-b-modal.todo-modal @click="whatToDo">Добавить задачу</button>

      <table class="table table-dark table-stripped table-hover">
        <thead class="thead-light">
          <tr>
            <th>Uid</th>
            <th>Описание</th>
            <th>Выполнена?</th>
            <th></th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(todo, index) in todos" :key="index">
            <td class="todo-uid">{{ todo.uid }}</td>
            <td>{{ todo.description }}</td>
            <td>
              <span v-if="todo.is_completed">Выполнено</span>
              <span v-else>Не выполнено</span>
            </td>
            <td>
              <div class="btn-group" role="group">
                <button type="button"
                        class="btn btn-secondary btn-sm"
                        v-b-modal.todo-update-modal
                        @click="updateTodo(todo)"
                        >
                    Обновить
                </button>
                &nbsp;
                <button type="button"
                        class="btn btn-danger
                        btn-sm" @click="deleteTodo(todo)">
                    X
                </button>
              </div>
            </td>
          </tr>
        </tbody>

      </table>

<!-- Эксперимент с формой -->
      <b-modal ref="addTodoModal"
           id="todo-modal"
           title="Добавить задачу"
           hide-footer>
        <taskForm v-bind:whatToDo="w_t_d" v-bind:addTodoForm="addTodoForm"
        v-on:submit="onSubmit" v-on:reset="onReset"></taskForm>
      </b-modal>

<!-- отключенное старое окно -->
<!--       <b-modal ref="addTodoModal"
           id="todo-modal"
           title="Добавить задачу"
           hide-footer>
        <b-form @submit="onSubmit" @reset="onReset" class="w-100">
          <b-form-group id="form-description-group"
                label="Описание:"
                label-for="form-description-input">
            <b-form-input id="form-description-input"
                  type="text"
                  v-model="addTodoForm.description"
                  required
                  placeholder="Завести задачу">
            </b-form-input>
          </b-form-group>
          <b-form-group id="form-complete-group">
            <b-form-checkbox-group v-model="addTodoForm.is_completed" id="form-checks">
              <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
            </b-form-checkbox-group>
          </b-form-group>
          <b-button-group>
            <b-button type="submit" variant="primary">Добавить</b-button>
            <b-button type="reset" variant="danger">Сброс</b-button>
          </b-button-group>
        </b-form>
      </b-modal> -->

<!-- пока оставленное окно для апдейта -->
      <b-modal ref="updateTodoModal"
         id="todo-update-modal"
         title="Update"
         hide-footer>
        <b-form @submit="onUpdateSubmit" @reset="onUpdateReset"
            class="w-100">
          <b-form-group id="form-update-description-group"
              label="Описание:"
              label-for="form-update-description-input">
            <b-form-input id="form-update-description-input"
                type="text"
                v-model="updateTodoForm.description"
                required>
            </b-form-input>
          </b-form-group>
          <b-form-group id="form-update-complete-group">
            <b-form-checkbox-group v-model="updateTodoForm.is_completed"
              id="form-update-checks">
              <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
            </b-form-checkbox-group>
          </b-form-group>
          <b-button-group>
            <b-button type="submit" variant="primary">Обновить</b-button>
            <b-button type="reset" variant="danger">Сброс</b-button>
          </b-button-group>
        </b-form>
      </b-modal>

    </div>


  </div>
</template>

<style>
button#task-add {
  margin-top: 20px;
  margin-bottom: 20px;
}
h1, td {
  text-align: left;
}
.todo-uid {
  text-align: center;
}
</style>

<script>
import axios from 'axios';
import Confirmation from './Confirmation.vue';
import TaskForm from './TaskForm.vue';

// const todoListURL = 'http://localhost:5000/api/tasks/';
// const todoAddURL = 'http://localhost:5000/api/tasks/';
const dataURL = 'http://localhost:5000/api/tasks/';

export default {
  name: 'Todo',
  data() {
    return {
      todos: [],
      addTodoForm: {
        description: '',
        is_completed: [],
      },
      updateTodoForm: {
        uid: 0,
        description: '',
        is_completed: [],
      },
      confirmationMessage: '',
      showConfirmation: true,
      showDismissibleAlert: false,
      w_t_d: '',
    };
  },
  methods: {

    whatToDo() {
      this.w_t_d = 'Добавить';
    },

    getTodos() {
      axios.get(dataURL)
        .then((response) => {
          this.todos = response.data.tasks;
        });
    },

    resetForm() {
      this.addTodoForm.description = '';
      this.addTodoForm.is_completed = [];
      this.updateTodoForm.description = '';
      this.updateTodoForm.is_completed = [];
    },

    // Как правильно?
    // onSubmit(event) {
    // onSubmit(event, sendData) {
    onSubmit(sendData) {
      // event.preventDefault();
      this.$refs.addTodoModal.hide();
      const requestData = {
        description: sendData.description,
        is_completed: sendData.is_completed,
      };
      console.log(requestData);

      // const requestData = {
      //   description: this.addTodoForm.description,
      //   is_completed: this.addTodoForm.is_completed[0],
      // };
      axios.post(dataURL, requestData)
        .then(() => {
          this.getTodos();
          this.confirmationMessage = `Задача "${requestData.description}" добавлена`;
          this.showConfirmation = true;
          this.showDismissibleAlert = true;
        });
      this.resetForm();
    },

    // onReset(event) {
    onReset(sendData) {
      // event.preventDefault();
      this.$refs.addTodoModal.hide();

      const requestData = {
        description: sendData.description,
        is_completed: sendData.is_completed,
      };
      console.log(requestData);
      this.resetForm();
    },

    updateTodo(todo) {
      this.updateTodoForm = todo;
    },

    onUpdateSubmit(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      const requestData = {
        description: this.updateTodoForm.description,
        is_completed: this.updateTodoForm.is_completed[0],
      };
      const todoURL = dataURL + this.updateTodoForm.uid;
      axios.put(todoURL, requestData)
        .then(() => {
          this.getTodos();
          this.confirmationMessage = 'Задача обновлена';
          this.showDismissibleAlert = true;
        });
    },

    onUpdateReset(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      this.resetForm();
      this.getTodos();
    },

    deleteTodo(todo) {
      const todoURL = dataURL + todo.uid;
      axios.delete(todoURL)
        .then(() => {
          this.getTodos();
          this.confirmationMessage = 'Задача удалена из списка';
          this.showDismissibleAlert = true;
        });
    },

    closeAlert() {
      this.showDismissibleAlert = false;
    },

  },

  components: {
    confirmation: Confirmation,
    taskForm: TaskForm,
  },

  created() {
    this.getTodos();
  },


};

</script>
