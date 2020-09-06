<template>
  <div class="create-task-modal" @click.stop="closeDropDown()">
      <div class="modal-container">
        <div class="close-btn" @click="closeModal()" title="Закрыть">
          <img src="../../assets/close.png" alt="Close">
        </div>
        <div class="content-container">
          <div class="title-task">
            <span>Наименование задачи:</span>
            <input type="text" v-model="task.title" placeholder="Введите название задачи">
          </div>
          <div class="title-task">
            <span>Группа:</span>

            <!-- Список задач-->
            <div class="group-select" v-if="Object.keys(groupsList).length" @click.stop="openDropDown($event.target)">
              {{ task.group.title ? task.group.title : 'Выберите группу' }}
            </div>
            <!-- Если нет задач -->
            <input type="text" placeholder="Нет созданных групп" readonly v-else>
          </div>
        </div>

        <!-- Выпадающий список с группами -->
        <div class="group-list" ref="groupsDropDown" v-show="activeDropDown">
          <div class="group-option"
               v-for="(group,index) in groupsList"
               @click.stop="selectGroup(group)"
               :key="'option' + index"
               :title="group.title"
          >
            {{ group.title }}
          </div>
        </div>
        <div class="create-btn" @click="createTask()">
          <span>Создать</span>
        </div>
      </div>
  </div>
</template>

<script>
export default {
  name: "CreateTaskModal",
  props: ['taskObj', 'groupsList'],
  data() {
    return {
      activeDropDown: false, // Активная выпадашка для задач
      task: {},  // объект задачи для заполнения данными
    }
  },
  methods: {

    /* Выбрать задачу */
    selectGroup(group) {
      // В массив добавляем выранную задачу
      this.task.group = JSON.parse(JSON.stringify(group));
      // Закрываем выпадашку
      this.closeDropDown();
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
    openDropDown(target) {
      // Получаем координты элемента, по которому нажали
      let coords = this.getCoords(target);
      // Позиционируем выпадашку ниже этого элемента
      this.$refs.groupsDropDown.style.left = coords.left + 'px';
      this.$refs.groupsDropDown.style.top = (coords.top + coords.height) + 'px';
      this.activeDropDown = true;
    },

    // Закрыть выпадающий список
    closeDropDown() {
      this.activeDropDown = false;
    },
    // Закрыть модальное окно
    closeModal() {
      this.$emit('closeModal');
    },
    /* Создать задачу */
    createTask() {
      if (!this.task.title) {
        console.log('Введите название задачи')
        return;
      }
      this.$emit('createTask', this.task);
    }
  },
  created() {
    // Избавляемся от ссылки
    this.task = JSON.parse(JSON.stringify(this.taskObj));
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
    @include flex-center;
    flex-direction: column;
    font-size: 14px;

    .title-task, .title-task {
      display: flex;
      flex-direction: column;
      font-size: 12px;
      margin-top: 10px;

      input, .group-select {
        width: 245px;
        height: 30px;
        padding-left: 5px;
        padding-right: 5px;
        border: 2px solid #dedede;
        outline: none;
      }

      .group-select {
        display: flex;
        align-items: center;
        font-size: 13px;
        background-color: #fff;
        cursor: pointer;
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


  .group-list {
    position: absolute;
    width: 245px;
    max-height: 100px;
    overflow-y: auto;
    background-color: #fff;
    border: 1px solid #dedede;
    box-shadow: 2px 3px 3px -1px rgba(0, 0, 0, .5);

    .group-option {
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