<template>
  <div id="app">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.14.0/css/all.css"
      integrity="sha384-HzLeBuhoNPvSl5KYnjx0BT+WB0QEEqLprO+NBkkk5gbc67FTaL7XIGa2w1L0Xbgc" crossorigin="anonymous" />
    <div v-if="isOpenModal" class="modal">
      <div class="modal--back_layer" @click="isOpenModal = false"></div>
      <div class="modal--main_layer">
        <div class="add_todo">
          <input type="text" v-model="inputAdd" placeholder="edit me" class="add_todo--input" id="inputTodo" />
          <button :disabled="isEmptyInputTodo" @click="addItem" class="add_todo--btn">
            追加
          </button>
        </div>
      </div>
    </div>
    <div class="modal-btn">
      <div @click="isOpenModal = false" v-if="isOpenModal">×</div>
      <div @click="isOpenModal = true" v-if="!isOpenModal">＋</div>
    </div>  
    <div class="todo_list">
      <div v-for="(todo, index) in todos" :key="index" class="todo_list--contents">
        <div class="todo_text">
          <div class="todo_text--wrap_text" v-if="!todo.isEditMode" @click="doneItem(index)">
            <span class="todo_text--text" v-if="!todo.isDone">{{ todo.item }}</span>
            <span class="todo_text--text__done" v-if="todo.isDone" style="text-decoration: line-through;">{{ todo.item }}</span>
          </div>
          <input type="text" v-if="todo.isEditMode" class="input-todo-edit" :id="'editTextArea' + index" />
        </div>
        <div class="todo_item--btn">
          <div class="btn-left">
            <button @click="deleteItem(index)" class="btn delete" v-if="!todo.isEditMode">
              <i class="fas fa-trash-alt"></i>
            </button>
            <button @click="editItem(index, 'cancel')" v-if="todo.isEditMode" class="btn cancel">
              <i class="fas fa-window-close"></i>
            </button>
          </div>
          <div class="btn-right">
            <button v-if="!todo.isEditMode" @click="editItem(index, 'edit')" class="btn">
              <i class="fas fa-pencil-alt"></i>
            </button>
            <button v-if="todo.isEditMode" @click="editItem(index, 'confirm')" class="btn">
              <i class="fas fa-check-circle"></i>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data: () => {
    return {
      inputAdd: "",
      isEmptyInputTodo: true,
      isOpenModal: false,
      todos: [],
    };
  },
  created: function () {
    let jsonObj = localStorage.getItem("todosKey");
    let jsObj = JSON.parse(jsonObj);
    for (let i in jsObj) {
      let todo = {
        item: jsObj[i].item,
        isDone: jsObj[i].isDone,
        isEditMode: false,
      };
      this.todos.push(todo);
    }
  },
  watch: {
    //TODO入力ボックスが空であればボタンdisable
    inputAdd: function () {
      if (this.inputAdd) {
        this.isEmptyInputTodo = false;
      } else {
        this.isEmptyInputTodo = true;
      }
    },
  },
  methods: {
    /**
     * TODO追加
     */
    addItem() {
      let todo = {
        item: this.inputAdd,
        isDone: false,
        isEditMode: false,
      };
      this.todos.push(todo);
      this.inputLocalStrage();
      this.inputAdd = "";
    },
    /**
     * TODO削除
     * @param {number} index - 削除するTODOのindex
     */
    deleteItem(index) {
      this.todos.splice(index, 1);
      this.inputLocalStrage();
    },
    /**
     * TODO編集
     * @param {number} index - 編集中のTODOのindex
     * @param {string} editState - 編集ボタンの状態
     */
    editItem(index, editState) {
      switch (editState) {
        case "edit": {
          this.todos[index].isEditMode = true;
          break;
        }
        case "cancel": {
          this.todos[index].isEditMode = false;
          break;
        }
        case "confirm": {
          let afterText = document.getElementById("editTextArea" + index).value;
          if (afterText) {
            this.todos[index].item = afterText;
          }
          this.todos[index].isEditMode = false;
          break;
        }
      }
      this.inputLocalStrage();
    },
    /**
     *実行済みTODOの打消し線
     * @param {number} index - 打ち消すTODOのindex
     */
    doneItem(index) {
      if (this.todos[index].isDone) {
        this.todos[index].isDone = false;
      } else {
        this.todos[index].isDone = true;
      }
      this.inputLocalStrage();
    },
    /**
     * ローカルストレージにデータ格納
     */
    inputLocalStrage() {
      let obj = JSON.stringify(this.todos);
      localStorage.setItem("todosKey", obj);
    },
  },
};
</script>

<style lang="scss">
.reset-btn-style {
  background-color: transparent;
  border: none;
  cursor: pointer;
  outline: none;
  padding: 0;
  appearance: none;
}

.base-input-text {
  font-size: 17px;
}

.main-title {
  text-align: center;
}

/* モーダル */
.modal{
    position: fixed;
    width: 100vw;
    height: 100vh;
    top: 0;
    left: 0;
  &--back_layer {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background-color: #DDDDDD33;
  }
  &--main_layer {
    position: fixed;
    width: calc(100vw - 20px);
    height: 60vh;
    top: 40vh;
    left: 0;
    padding: 10px;
    background-color: #FFF;
  }
}

.modal-btn{
  position: fixed;
  bottom: 10px;
  right: 10px;
  border-radius: 50%;
  width: 50px;
  height: 50px;
}

/* モーダルの中身 */
.add_todo {
  height: 40px;
  display: flex;
  justify-content: space-between;

  &--input {
    @extend .base-input-text;
    width: 75%;
    height: 34px;
    display: inline-block;
  }

  &--btn {
    @extend .reset-btn-style;
    width: 20%;
    height: 40px;
    display: inline-block;
    padding: 0.5em 1em;
    text-decoration: none;
    background: #76ff03;
    color: #fff;
    font-weight: bold;
    border-radius: 3px;
    border-bottom: solid 2px #3e962f;
    border-right: solid 2px #3e962f;

    &:active {
      /*ボタンを押したとき*/
      border: none;
      -webkit-transform: translate(3px, 3px);
      transform: translate(3px, 3px);
    }
  }
}

/* TODO一覧部分 */
.todo_list {
  padding: 0 10px;
  width: calc(100vw - 20px);
  &--contents {
    border-bottom: solid 1px #999;
    padding: 10px;
    display: flex;
  }
}

.todo_text {
  width: 70%;
  word-break: break-all;
  &-wrap_text {
    width: 100%;
    display: block;
    &--done_check:checked + &--text {
      text-decoration: line-through;
    }
  }

  .input-todo-edit {
    @extend .base-input-text;
    width: 90%;
    height: 40px;
  }
}

.todo_item-btn {
  width: 30%;
  display: flex;
  justify-content: space-between;
  align-items: center;

  .btn-left {
    width: 45%;

    .btn {
      @extend .reset-btn-style;
      width: 100%;
      height: 30px;
      border: solid 2px #999;
      border-radius: 5px;
      color: #999;
    }
  }

  .btn-right {
    width: 45%;

    .btn {
      @extend .reset-btn-style;
      width: 100%;
      height: 30px;
      border: solid 2px #999;
      color: #76ff03;
      border-radius: 5px;
    }
  }
}
</style>
