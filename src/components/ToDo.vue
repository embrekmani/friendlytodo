<template>
    <div>
        <input type="text" class="todo-input" placeholder="What's on the agenda?" v-model="newToDo" @keyup.enter="addToDo">
        <transition-group name="fade" enter-active-class="animated fadeInDown" leave-active-class="animated fadeOut">
        <div v-for="(todo, index) in todos" :key="todo.id" class="todo-item">
            <div class="main-todo">
                <div class="todo-item-left">
                    <div>
                        <input class="check" type="checkbox" v-model="todo.complete" @click="completeToDo(todo); removeToDo(index);">
                    </div>
                    <div v-if="!todo.editing" @dblclick="editToDo(todo)" class="todo-item-label">{{ todo.title }}</div>
                    <input v-else type="text" v-model="todo.title" class="todo-edit" v-focus 
                    @focus="$event.target.select()"
                    @blur="finishEditing(todo)" 
                    @keyup.enter="finishEditing(todo)" 
                    @keyup.escape="cancelEdit(todo)">
                </div>
                <div class="buttons">
                    <div v-if="!todo.isHelp" class="add-sub-todo todo-button" @click="addSubTodo(todo)">
                        &plus;
                    </div>
                    <div class="remove-todo todo-button" @click="removeToDo(index)">
                        &times;
                    </div>
                </div>
            </div>
            <hr v-if="calcPercentageComplete(todo) > 0" class="progress-bar" v-bind:style="{ width: calcPercentageComplete(todo) + '%' }">
                <div class="sub-todo-item" v-for="(subTodo, subIndex) in todo.subTodos">
                    <div class="main-todo sub-todo">
                        <div class="todo-item-left">
                            <div>
                                <input class="check" type="checkbox" v-model="subTodo.complete">
                            </div>
                            <div v-if="!subTodo.editing" @dblclick="editToDo(subTodo)" v-bind:class="{ complete: subTodo.complete }" class="todo-item-label">{{ subTodo.title }}</div>
                            <input v-else type="text" v-model="subTodo.title" class="todo-edit" v-focus 
                            @focus="$event.target.select()"
                            @blur="finishEditing(subTodo)" 
                            @keyup.enter="finishEditing(subTodo)" 
                            @keyup.escape="cancelEdit(subTodo)">
                        </div>
                        <div class="buttons">
                            <div class="remove-todo todo-button" @click="removeSubTodo(todo, subIndex)">
                                &times;
                            </div>
                        </div>
                    </div>
                </div>
        </div>
        </transition-group>
        <hr>
        <transition-group name="fade" enter-active-class="animated fadeIn">
            <div v-for="(done, doneIndex) in done" :key="done.id" class="complete-items">
                <div class="complete">{{ done.title }}</div>
            </div>
        </transition-group>
        <div @click="sideMenuOpen = true" class="burger"><i class="fas fa-bars"></i></div>
        <div class="sidebar" v-bind:class="{ active: sideMenuOpen }">
            <h1 @click="sideMenuOpen = false">&times;</h1>
            <h2>This is the friendly todo app.</h2>
            <p>When you're hard at work, it's easy to forget the habits of a healthy, more productive lifestyle. The friendly todo app reminds you what you need to do whenever you finish a task.</p>
            <br>
            <p><strong>Add something you need to be reminded of every now and then:</strong></p>
            <input type="text" placeholder="e.g. Take the dog for a walk." v-model="newHelp" @keyup.enter="addHelp"></input>
            <p style="padding-top: .5rem;"><strong>List:</strong></p>
            <ul>
                <li v-for="tip in tips">{{ tip }}</li>
            </ul>
        </div>
    </div>
</template>

<script>

export default {
    name: 'todo',
    data () {
        return {
            sideMenuOpen: false,
            newToDo: '',
            newHelp: '',
            idForToDo: 1,
            titleCache: '',
            todos: [],
            tips: ['Drink a glass of water.', 'Stand up and stretch.', 'Take a 5 minute break.', 'Go for a short walk.', 'Have a healthy snack.', 'Meditate for 5 or 10 minutes.', "You've earned yourself a cup of tea or coffee."],
            done: [],
        }
    },
    directives: {
        focus: {
            inserted: function(el) {
                el.focus();
            }
        }
    },
    methods: {
        addToDo() {
            if (this.newToDo.trim().length == 0) { // checks if input is empty 
                return; // add error message
            }
            var todo = this.todos.unshift({
                id: this.idForToDo,
                idForSubTodo: 1,
                newSubTodo: '',
                title: this.newToDo,
                subTodos: [],
                complete: false,
                isHelp: false,
                editing: false,
            })
            this.newToDo = '';
            this.idForToDo++;
        },
        addSubTodo(todo) {
            todo.subTodos.push({
                id: todo.idForSubTodo,
                title: '',
                complete: false,
                isHelp: false,
                editing: true,
            })
            todo.idForSubTodo++;
        },
        addHelp() {
            if (this.newHelp.trim().length == 0) { // checks if input is empty 
                return; // add error message
            }
            this.tips.push(this.newHelp)
            this.newHelp = '';
        },
        removeToDo(index) {
            this.todos.splice(index, 1);
        },
        removeSubTodo(todo, index) {
            todo.subTodos.splice(index, 1);
        },
        editToDo(todo) {
            this.titleCache = todo.title;
            todo.editing = true;
        },
        finishEditing(todo) {
            if (todo.title.trim().length == 0) {
                todo.title = this.titleCache;
            }
            todo.editing = false;
        },
        cancelEdit(todo) {
            todo.editing = false;
            todo.title = this.titleCache;
        },
        completeToDo(todo) {
            if (!todo.isHelp) {
                this.todos.push({
                    id: this.idForToDo,
                    title: this.tips[Math.floor(Math.random() * this.tips.length)],
                    complete: false,
                    isHelp: true,
                    subTodos: [],
                })
                this.idForToDo++;
            }
            this.done.push({
                id: todo.id,
                title: todo.title,
            });
        },
        calcPercentageComplete(todo) {
            var subTodoCount = todo.subTodos.length;
            var completeSubTodoCount = 0;
            var percentage;
            for (var i = 0; i < subTodoCount; i++) {
                if (todo.subTodos[i].complete) {
                    completeSubTodoCount++;
                }
            }
            percentage = (completeSubTodoCount * 100) / subTodoCount;
            return percentage;
        }
    }
}
</script>

<style lang="scss">

@import url("https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.0/animate.min.css");

.todo-input {
    width: 100%;
    padding: 0 0 0.5rem 0.5rem;
    font-size: 1.4rem !important;
    margin-bottom: 1.5rem;
    
    &:focus {
        outline: 0;
    }
}

.progress-bar {
    border: 1px solid lightgreen !important;
    left: 0;
    margin-right: auto !important;
    margin-left: 0 !important;
    transition: width 0.1s;
    margin-top: 8px !important;
    margin-bottom: 6px !important;
    padding: 0 !important;
}

.todo-item {
    font-family: 'Source Code Pro', monospace;
    font-size: 1.5rem;
    margin-bottom: 1rem;
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-direction: column;
}

.sub-todo-item {
    width: 100%;
    font-size: 1.3rem;
}
.sub-todo {
    padding-left: 2rem;
}

.remove-todo {
    cursor: pointer;
    &:hover {
        color: red;
    }
}

.add-sub-todo {
    cursor: pointer;
    &:hover {
        color: lightgreen;
    }
}

.buttons {
    display: flex !important;
    flex-direction: row;
    justify-content: flex-end !important;
    font-size: 1.6rem;
    width: 3rem;
    height: 1rem;
    font-weight: bold;
    padding-bottom: 6px;
    .remove-todo {
        padding-left: .8rem;
    }
}

.check {
    margin-top: 8px;
}


.main-todo {
    display: flex; 
    flex-direction: row;
    width: 100%;
}
.todo-item-left {
    display: flex !important;
    flex-direction: row;
    flex-wrap: nowrap;
    align-items: center;
    align-content: space-between;
    height: 100%;
    width: 90%;
}

.todo-item-label {
    display: flex;
    margin-left: 1rem;
    max-width: 100%;
    padding-right: 2rem;
    width: 100%;
    overflow: hidden;
}

.todo-edit {
    font-size: 1.3rem !important;
    border: none;
    border-bottom: 1px solid #FFDEA6;
    margin-left: 1rem;
    padding-bottom: 0.25rem;
    width: 90%;
    &focus {
        outline: none;
    }
}

.complete {
    text-align: left;
    text-decoration: line-through;
    color: #FFDEA6;
    font-family: 'Source Code Pro', monospace;
}

.complete-items {

}

.complete-items {
    overflow-x: hidden;
    text-align: left;
    .complete {
        padding-bottom: 1rem;
        padding-top: 0;
        font-size: 1.5rem;
    }
}

hr {
    border: none !important;
    border-top: 1px solid #FFDEA6 !important;
    background-color: transparent !important;
    margin: 2rem 0 0 0 !important;
    padding-top: 1.5rem !important; 
}

.popup {
    border-radius: 12px;
    padding: .75rem 1.5rem 1rem 1.5rem;
    position: fixed;
    display: block;
    width: 25%;
    bottom: 2rem;
    right: 2rem;



}

.fadeInDown {
    -webkit-animation: fadeIn 1s; /* Safari 4+ */
    -moz-animation:    fadeIn 0.25s; /* Fx 5+ */
    -o-animation:      fadeIn 1s; /* Opera 12+ */
    animation:         fadeIn 0.25s; /* IE 10+, Fx 29+ */
}
.fadeOut {
    -webkit-animation: fadeOut 1s; /* Safari 4+ */
    -moz-animation:    fadeOut 0.25s; /* Fx 5+ */
    -o-animation:      fadeOut 1s; /* Opera 12+ */
    animation:         fadeOut 0.25s; /* IE 10+, Fx 29+ */
}

 /* The side navigation menu */
.sidebar {
    box-shadow: 0 0 8px 0 #FFDEA6;
    height: 100%; /* 100% Full-height */
    width: 0; /* 0 width - change this with JavaScript */
    position: fixed; /* Stay in place */
    z-index: 1; /* Stay on top */
    top: 0; /* Stay at the top */
    right: 0;
    background-color: #FFDEA6;
    overflow-x: hidden; /* Disable horizontal scroll */
    transition: all 0.3s ease-out; /* 0.5 second transition effect to slide in the sidenav */
    overflow-y: auto;
    color: #828DAB;
    text-align: left;
    h1 {
        padding: 0;
        line-height: 1rem;
        font-size: 3.5rem !important;
        padding-top: 1rem;
        padding-bottom: 1rem;
        cursor: pointer;
    }
    h2 {
        font-weight: 700;
        font-size: 2.5rem;
        padding-bottom: 1.5rem;
    }
    p {
        font-family: 'Source Code Pro', monospace;
    }
    strong {
        color: #828DAB;
    }
    input {
        width: 100%;
        background-color: transparent;
        color: #828DAB;
        border-bottom: 1px solid #828DAB;
        padding-bottom: 4px;
        margin-bottom: .5rem;
        margin-top: .5rem;
    }
    a {
        padding: 8px 8px 8px 32px;
        text-decoration: none;
        font-size: 25px;
        color: #818181;
        display: block;
        transition: 0.3s;
        &:hover {  
            color: #f1f1f1;
        }
    }
    ul {
        padding-top: .5rem;
        padding-bottom: 1rem;
        font-family: 'Source Code Pro', monospace;
        list-style-type: disc;
        list-style-position: inside;
        
    }
}
.sidebar.active {
    width: 450px;
    padding: 1.5rem 2rem 1rem 2rem;
}
/* Position and style the close button (top right corner) */
.sidebar .closebtn {
  position: absolute;
  top: 0;
  right: 25px;
  font-size: 36px;
  margin-left: 50px;
}
.burger {
    position: fixed;
    top: 2rem;
    right: 3rem;
    font-size: 3rem;
    cursor: pointer;
}

@media screen and (max-width: 600px) {
    .sidebar.active {
        width: 100%;
    }
    .wrapper {
        width: 100% !important;
        max-width: 100% !important;
        padding: 1.5rem;
    }
    .todo-item-left {
        width: 80%;
    }
    .buttons {
        width: 4rem !important;
    }
    .burger {
        right: 2rem;
        top: 1rem;
    }
}
</style>
