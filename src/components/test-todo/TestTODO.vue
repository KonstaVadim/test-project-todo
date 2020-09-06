<template>
  <div class="test-todo">

    <!-- Блок с вкладками и с фильтром -->
    <div class="header">

      <!-- Контейнер с вкладками -->
      <div class="tabs-container">
        <!-- Вкладка со всеми задачами -->
        <div class="all-tasks tab"
             :class="{'active-tab': activeTab === 1}"
             @click="activeTab = 1"
        >
          <span>Все задания</span>
        </div>

        <!-- Вкладка с сгруппированными задачами-->
        <div class="grouped-tasks tab"
             :class="{'active-tab': activeTab === 2}"
             @click="activeTab = 2"
        >
          <span>Сгруппированные задачи</span>
        </div>
      </div>

      <!-- Фильтр -->
      <div class="filter">
        <div class="tasks-filter" v-if="activeTab == 1">
          <label for="all">Все</label>
          <input type="radio" value="all" id="all" v-model="statusFilter">
          <label for="successful">Выполнено</label>
          <input type="radio" value="successful" id="successful" v-model="statusFilter">
          <label for="bad">Не выполнено</label>
          <input type="radio" value="bad" id="bad" v-model="statusFilter">
          <input type="text" placeholder="Введите задачу" v-model="searchTextTask">
        </div>
        <input type="text" placeholder="Введите группу" v-model="searchTextGroup" v-else>
      </div>
    </div>

    <!-- Блок с задачами -->
    <transition-group name="block" tag="div" class="content-container" :class="{'column': activeTab === 2}">
      <!-- Карточки с задачами-->
      <template v-if="activeTab === 1">
        <card-task v-for="(task) in filteredTasksList"
                   @deleteTask="deleteTask(task.id)"
                   @changeStatus="changeStatus(task.id)"
                   :taskObj="tasksList[task.id]"
                   :key="'task' + task.id"/>
      </template>

      <!-- Карточки с группами -->
      <template v-if="activeTab === 2">
        <card-group v-for="(group, index) in filteredGroupsList"
                    @deleteGroup="deleteGroup(group.id)"
                    @editGroup="editGroup(group)"
                    :groupObj="group"
                    :key="'group' + index"/>
      </template>
    </transition-group>

    <!-- Блок с кнопками -->
    <div class="footer">
      <div class="right-btn">
        <div class="create-task" @click="openModal('create-task-modal')">
          <span>Создать задачу</span>
        </div>
        <div class="create-group" @click="openModal('create-group-modal')">
          <span>Создать группу</span>
        </div>
      </div>
    </div>
    <transition name="fade">
      <create-task-modal
          v-if="activeModal === 'create-task-modal'"
          :taskObj="taskObj"
          :groupsList="groupsList"
          @createTask="createTask($event)"
          @closeModal="closeModal()"
      />
      <create-group-modal
          v-if="activeModal === 'create-group-modal'"
          :tasksList="tasksList"
          :groupObj="groupObj"
          @createGroup="createGroup($event)"
          @closeModal="closeModal()"
      />
    </transition>
  </div>
</template>

<script>

import CardTask from "@/components/test-todo/Cards/CardTask";
import CardGroup from "@/components/test-todo/Cards/CardGroup";
import CreateTaskModal from "@/components/modals/CreateTaskModal";
import CreateGroupModal from "@/components/modals/CreateGroupModal";

export default {
  name: "TestTODO",
  components: {
    CreateGroupModal,
    CardTask,
    CreateTaskModal,
    CardGroup
  },
  data() {
    return {
      statusFilter: 'all',  // Фильтр по статусу
      searchTextGroup: '',  // введенный текст для поиска группы
      searchTextTask: '',  // введенный текст для поиска задачи
      activeTab: 1,     // активаная вкладка
      activeModal: '',  // Активное модальное окно
      tasksList: {},  // Список созданных задач
      groupsList: {},  // Список созданных групп
      iterator: 0,    // итератор
      groupObj: {},    // объект для создания/изменения группы
      taskObj: {},    // объект для создания/изменения задачи
    };
  },
  computed: {
    /* Отфильтрованный массив с задачами */
    filteredTasksList() {
      let tasksList = Object.values(JSON.parse(JSON.stringify(this.tasksList)));
      /* Фильтрация*/
      if (this.searchTextTask.length || this.statusFilter !== 'all') {
        tasksList = tasksList.filter(task => {
          let isValidation = [];
          let taskTitle = task.title.trim().toLowerCase(),
              searchTextTask = this.searchTextTask.trim().toLowerCase(),
              statusFilter = this.statusFilter == 'successful' ? true : false;

          // Ищем по наименованию группы
          if (searchTextTask) {
            if (taskTitle.indexOf(searchTextTask) > -1) {
              isValidation.push(true);
            } else {
              isValidation.push(false);
            }
          }
          // Ищем по статусу
          if (statusFilter !== 'all') {
            if (statusFilter == task.status) {
              isValidation.push(true);
            } else {
              isValidation.push(false);
            }
          }
          return isValidation.indexOf(false) < 0;
        })
      }
      return tasksList.reverse();
    },
    /* Отфильтрованный массив с группами */
    filteredGroupsList() {
      let groupsList = Object.values(JSON.parse(JSON.stringify(this.groupsList)));
      /* Фильтрация*/
      if (this.searchTextGroup.length) {
        groupsList = groupsList.filter(group => {
          let groupTitle = group.title.trim().toLowerCase(),
              searchTextGroup = this.searchTextGroup.trim().toLowerCase();

          // Ищем по наименованию группы
          if (groupTitle.indexOf(searchTextGroup) > -1) {
            return true
          } else return false;
        })
      }
      return groupsList.reverse();
    },

  },
  methods: {
    /* Открыть модалку */
    openModal(name) {
      this.groupObj = {
        title: '',
        selectedTasks: [{id: this.iterator++}]
      };
      this.iterator++;
      this.taskObj = {
        title: '',
        group: {
          id: '',
          title: '',
        },
        description: '',
        status: false,
      }
      this.activeModal = name;
    },
    /* Закрыть модальное окно */
    closeModal() {
      this.activeModal = '';
    },
    /* Создать группу */
    createGroup(groupObj) {
      if (!groupObj.id) {
        groupObj.id = this.iterator;
      }
      this.$set(this.groupsList, groupObj.id, groupObj);
      this.closeModal();
    },
    /* Создать задачу */
    createTask(taskObj) {
      taskObj.id = this.iterator;
      let obj = JSON.parse(JSON.stringify(taskObj));
      this.$set(this.tasksList, this.iterator++, obj);
      this.closeModal();
    },
    /* Изменить группу */
    editGroup(group) {
      this.groupObj = JSON.parse(JSON.stringify(group));
      if (!this.groupObj.selectedTasks.length) {
        this.groupObj.selectedTasks.push({id: this.iterator++});
      }
      this.activeModal = 'create-group-modal';
    },
    /* Удалить группу */
    deleteGroup(id) {
      this.$delete(this.groupsList, id);
    },
    /* Удалить задачу */
    deleteTask(id) {
      this.$delete(this.tasksList, id);
    },
    /* Изменить статус задачи */
    changeStatus(id) {
      console.log(this.tasksList)
      let task = this.tasksList[id];
      task.status = !task.status;
    },
  }
}
</script>

<style lang="scss" scoped>

@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin btn($width, $height, $bgColor, $deg, $minusDeg) {
  @include flex-center;
  width: $width;
  height: $height;
  background-color: $bgColor;
  transform: skew($deg);
  cursor: pointer;

  span {
    transform: skew($minusDeg);
  }
}

$pepper: #AF4425;
$cinnamon: #662E1C;
$cream: #EBDCB2;
$caramel: #C9A66B;
$border-color: #673e37;

.test-todo {
  display: flex;
  position: relative;
  flex-direction: column;
  background-color: #f0f0f0;
  height: 100vh;
  width: 100vw;
  min-width: 800px;
  min-height: 510px;
  padding: 10px;
}

.header {
  display: flex;
  justify-content: space-between;
  font-size: 14px;
  color: #fff;
  overflow: hidden;
  border: 2px solid $border-color;
}

.tabs-container {
  display: flex;

  .tab {
    @include flex-center;
    @include btn(220px, 35px, #a79086, 45deg, -45deg);

    &:hover {
      background-color: #673e37;
    }
  }

  .active-tab {
    background-color: #673e37;
  }

  .all-tasks {
    margin-left: -20px;
  }

  .grouped-tasks {
    border-left: 3px solid #fff;
  }
}

.filter {
  @include flex-center;
  margin: 5px;
  padding: 0 10px;
  font-size: 12px;
  color: #000;

  .tasks-filter {
    @include flex-center;

    label {
      margin-right: 5px;
    }
    input[type="radio"] {
      margin-right: 10px;
    }
  }

  input[type="text"] {
    margin-left: 10px;
    width: 150px;
    height: 20px;
    outline: none;
    padding: 0 5px;
    border: 1px solid #673e37;
  }


}

.content-container {
  display: flex;
  flex-wrap: wrap;
  flex: 1;
  padding: 15px;
  overflow: auto;
  border-right: 2px solid $border-color;
  border-bottom: 2px solid $border-color;
  border-left: 2px solid $border-color;
}

.column {
  flex-direction: column;
}

.footer {
  display: flex;
  justify-content: space-between;
  font-size: 14px;
  color: #fff;
  overflow: hidden;

  .left-btn {
    @include btn(140px, 35px, #bfbfb6, 45deg, -45deg);
    margin-left: -20px;
  }
}

.right-btn {
  margin-left: auto;
  display: flex;

  .create-task {
    @include btn(170px, 35px, #6d031c, -45deg, 45deg);
    border-right: 3px solid #fff;

    &:hover {
      background-color: #48010d;
    }
  }

  .create-group {
    @include btn(170px, 35px, #6d031c, -45deg, 45deg);
    margin-right: -20px;

    &:hover {
      background-color: #48010d;
    }
  }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}

.fade-enter, .fade-leave-to {
  opacity: 0;
}


.block-enter-active, .block-leave-to {
  opacity: 0;
}

.block-leave-active {
  position: absolute;
}


</style>