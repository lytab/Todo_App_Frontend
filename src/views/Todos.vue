<template>
<div class="container mx-auto">
    <h1 class="mt-8 text-2xl">Todos</h1>

    <div class="mt-3">
        <div class="grid grid-cols-12 gap-4">
            <div class="col-span-6 space-y-4 px-1" style="height: 500px">
                <div v-for="(todo, index) in state.todos" :key="index" class="flex flex-col p-8 bg-white shadow-md rounded flex items-center justify-between" :class="{'bg-green-200': todo.done}">
<div class="flex justify-between items-center">

                    <div>
                        <div>{{ todo.text }}</div>
                        <div class="text-gray-500 text-sm">{{ todo.created_at }}</div>
                    </div>
                    <div class="space-x-2">
                        <button class="px-2 text-red-600" @click="removeTodo(todo)" title="Remove todo">&times;</button>
                        <button v-if="!todo.done" class="px-2 text-green-600" @click="markAsDone(todo)" title="Mark as done">&check;</button>
                        <button v-else class="px-2 text-orange-600" @click="markAsUndone(todo)" title="Mark as undone">&#8630;</button>
                        <button v-if="!todo.done" class="px-2 text-blue-600" @click="initializeEditable(index)" title="Open TO Edit">&#9998;</button>
                    </div>
</div>

                    <div v-if="state.editTodos[index] && state.editTodos[index].editable" class="p-2 mt-5 w-full bg-white shadow-md rounded">
                        <h2 class="text-xl">Edit a todo</h2>
                        <input type="text" v-model="state.editTodos[index].text" @keydown.enter="editTodo(index)" class="p-2 mt-4 border rounded w-full">
                    </div>
                </div>
                <div v-if="state.todos.length === 0" class="px-8 py-16 bg-white text-gray-700 shadow-md rounded text-sm">
                    You dont have any task to do.
                </div>
            </div>

            <div class="col-span-6">
                <div class="p-8 bg-white shadow-md rounded">
                    <h2 class="text-xl">Add a todo</h2>
                    <input type="text" v-model="state.todoText" @keydown.enter="addTodo" class="p-2 mt-4 border rounded w-full">
                </div>
            </div>
        </div>
    </div>
</div>
</template>

<script>
import axios from 'axios'
import { defineComponent, reactive } from 'vue'
export default defineComponent({
    setup() {
        const state = reactive({
            todos: [],
            todoText: "",

            editTodos: []
        })

        function initializeEditable(index) {
            state.editTodos[index] = {
                text: "",
                editable: true
            }
        }

        function editTodo(index) {

            axios.post(process.env.VUE_APP_BASE_API_URL + '/edit-todo', { 'id': state.todos[index].id, 'text': state.editTodos[index].text }).then(res => {
                if (res.status == 200) {
                    getTodos()
                    state.editTodos[index].editable = false;
                    state.editTodos[index].text = "";
                }
            })
        }

        function addTodo() {

            axios.post(process.env.VUE_APP_BASE_API_URL + '/store-todo', { 'text': state.todoText }).then(res => {
                if (res.status == 200) {
                    getTodos()
                    state.todoText = ""
                }
            })
        }

        function removeTodo(todo) {
            if (!confirm("Are You Sure ?")) {
                return;
            } else {
                axios.delete(process.env.VUE_APP_BASE_API_URL + '/delete-todo/' + todo.id).then(res => {
                    if (res.status == 200) {
                        getTodos()
                    }
                })

            }
        }

        function markAsDone(todo) {
            axios.post(process.env.VUE_APP_BASE_API_URL + '/mark-as-done/' + todo.id).then(res => {
                if (res.status == 200) {
                    getTodos()
                }
            })
        }

        function markAsUndone(todo) {
            axios.post(process.env.VUE_APP_BASE_API_URL + '/mark-as-undone/' + todo.id).then(res => {
                if (res.status == 200) {
                    getTodos()
                }
            })
        }

        function getTodos() {
            axios.get(process.env.VUE_APP_BASE_API_URL + '/todos').then(res => {
                state.todos = res.data;
            })
        }
        getTodos()
        return {
            state,
            addTodo,
            removeTodo,
            markAsDone,
            markAsUndone,
            getTodos,
initializeEditable,
editTodo
        }
    }
})
</script>
