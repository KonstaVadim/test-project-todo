<template>
  <div class="create-task-modal" @click.stop="closeDropDown()">
    <!-- Модальное окно-->
    <div class="modal-container">
      <!-- Кнопка закрытия -->
      <div class="close-btn" @click="closeModal()" title="Закрыть">
        <img src="../../assets/close.png" alt="Close">
      </div>
      <!-- Блок с контентом -->
      <div class="content-container">
        <!-- Наименование группы -->
        <div class="title-task">
          <span>Наименование группы:</span>
          <input type="text" v-model="group.title" placeholder="Введите название задачи">
        </div>
        <!-- Выбор задач -->
        <div class="title-task">
          <span>Задача:</span>
          <!-- Список выбранных задач-->
            <div class="task-container"
                 v-for="(selectedTask, index) in group.selectedTasks"
                 :key="'selectedTask' + selectedTask.id"
            >

              <!-- Список задач-->
              <div class="task-select" v-if="Object.keys(tasksList).length"
                   @click.stop="openDropDown(index, $event.target)">
                {{ selectedTask.title ? selectedTask.title : 'Выберите задачу' }}
              </div>
              <!-- Если нет задач -->
              <input type="text" placeholder="Создайте задачу" readonly v-else>

              <!-- Кнопка удалить -->
              <div class="btn-del" @click="deleteTask(index)" v-if="group.selectedTasks.length > 1">
                <img src="../../assets/delete_4219.png" alt="">
              </div>

              <!-- Кнопка добавить -->
              <div class="btn-add" v-if="index == group.selectedTasks.length - 1" @click="addTask()">
                <img src="../../assets/plus.png" alt="">
              </div>
            </div>
        </div>
      </div>

      <!-- Выпадающий список с задачами -->
      <div class="tasks-list" ref="tasksDropDown" v-show="activeDropDown">
        <div class="task-option"
             v-for="(task,index) in tasksList"
             @click.stop="selectTask(task)"
             :key="'option' + index"
             :title="task.title"
        >
          {{ task.title }}
        </div>
      </div>
      <!-- Кнопка создать -->
      <div class="create-btn" @click="createGroup()">
        <span>Создать</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "CreateGroupModal",
  props: ['tasksList', 'groupObj'],
  data() {
    return {
      group: {},  // Объект для заполнения данными
      selectedTaskIndex: {}, // выбранный объект для заполнения задачей
      activeDropDown: false, // Активная выпадашка для задач
      iterator: -1,
    }
  },
  methods: {
    // Закрыть выпадающий список
    closeDropDown() {
      this.activeDropDown = false;
      this.selectedTaskIndex = null;
    },
    // Получить координаты элемента
    getCoords(elem) {
      let box = elem.getBoundingClientRect();

      return {
        top: box.top + pageYOffset,
        left: box.left + pageXOffset,
        height: box.height
      };
    },

    /* Открыть выпадающий список */
    openDropDown(selectedTaskIndex, target) {
      // Получаем координты элемента, по которому нажали
      let coords = this.getCoords(target);
      // Позиционируем выпадашку ниже этого элемента
      this.$refs.tasksDropDown.style.left = coords.left + 'px';
      this.$refs.tasksDropDown.style.top = (coords.top + coords.height) + 'px';
      this.activeDropDown = true;
      // Запоминаем индекс этого места
      this.selectedTaskIndex = selectedTaskIndex;
    },
    /* Удалить задачу */
    deleteTask(index) {
      this.group.selectedTasks.splice(index, 1);
    },
    /* Добавить задачу */
    addTask() {
      this.$emit('iterator');
      this.group.selectedTasks.push({id: this.iterator--});
    },
    /* Выбрать задачу */
    selectTask(task) {
      let isRepeat = this.validateSelectedList(this.group.selectedTasks, task);
      console.log(isRepeat)
      if (isRepeat) return;
      // В массив добавляем выранную задачу
      this.group.selectedTasks.splice(this.selectedTaskIndex, 1, task);
      // Закрываем выпадашку
      this.closeDropDown();
    },
    // Закрыть модальное окно
    closeModal() {
      this.$emit('closeModal');
    },
    // Проверка, что выбранная задача не повторяется
    validateSelectedList(array, task) {
      let isRepeat = false
      array.forEach((item) => {
        if (item.title == task.title) isRepeat = true;
      })
      return isRepeat;
    },
    // Проверка выбранных задач, что задачи действительно были выбраны
    checkSelectedTasks(array) {
      array.forEach((item, index) => {
        if (!item.title) {
          array.splice(index, 1);
        }
      })
    },
    /* Создать группу */
    createGroup() {
      if (!this.group.title) {
        console.log('Введите название группы');
        return;
      }
      this.checkSelectedTasks(this.group.selectedTasks);
      this.$emit('createGroup', this.group);
    }
  },
  created() {
    // Избавляемся от ссылки
    this.group = JSON.parse(JSON.stringify(this.groupObj));
  }
}
</script>

<style lang="scss" scoped>

@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin input {
  border-radius: 5px;
  border-color: #1b1210;
  outline: none;
}

.create-task-modal {
  @include flex-center;
  position: absolute;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
  background-color: rgba(0, 0, 0, .5);
}

.modal-container {
  display: flex;
  flex-direction: column;
  width: 350px;
  background-color: #f0f0f0;
  border-radius: 10px;
  overflow: hidden;

  .close-btn {
    align-self: flex-end;
    margin-top: 5px;
    margin-right: 5px;
    cursor: pointer;
  }

  .content-container {
    margin-left: 20px;
    max-height: 240px;
    overflow-y: auto;
    font-size: 14px;

    .title-task, .title-task {
      display: flex;
      flex-direction: column;
      font-size: 12px;
      margin-top: 10px;

      input, .task-select {
        position: relative;
        width: 245px;
        height: 30px;
        padding-left: 5px;
        padding-right: 5px;
        border: 2px solid #dedede;
        outline: none;
      }

      .task-select {
        display: flex;
        align-items: center;
        font-size: 13px;
        background-color: #fff;
        cursor: pointer;
      }
    }

    .task-container {
      display: flex;
      align-items: center;
      margin-bottom: 10px;

      input {
        cursor: pointer;
      }

      .btn-del, .btn-add {
        @include flex-center;
        margin-left: 10px;
        font-size: 23px;
        cursor: pointer;

        img {
          width: 20px;
          height: 20px;
        }
      }
    }
  }

  .create-btn {
    @include flex-center;
    height: 30px;
    margin-top: 10px;
    font-size: 14px;
    background-color: #a79086;
    cursor: pointer;
    color: #fff;

    &:hover {
      background-color: #6d031c;
    }
  }

  .tasks-list {
    position: absolute;
    width: 245px;
    max-height: 100px;
    overflow-y: auto;
    background-color: #fff;
    border: 1px solid #dedede;
    box-shadow: 2px 3px 3px -1px rgba(0, 0, 0, .5);

    .task-option {
      display: block;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      width: 100%;
      height: 25px;
      background-color: #fff;
      cursor: pointer;
      padding: 2px 10px;
      font-size: 14px;

      &:hover {
        background-color: #eaeaea;
      }

      &:not(:last-child) {
        border-bottom: 2px solid #c6c6c6;
      }
    }
  }
}
</style>