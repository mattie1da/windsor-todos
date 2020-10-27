<template>
    <v-app>
        <v-main>
            <v-container>
                <div v-if="hasError">
                    <p>Failed to retrieve the tasks 😔</p>
                    <v-btn @click="getTasks">Try again?</v-btn>
                </div>
                <div v-else>
                    <v-row no-gutters justify="space-between">
                        <h1>Tasks</h1>
                        <v-text-field v-if="tasks.length" prepend-icon="mdi-magnify" class="col-md-4" label="Filter" v-model="taskSearch" flat solo-inverted></v-text-field>
                    </v-row>

                    <v-progress-linear v-if="tasks.length" class="mb-5" :value="progress"></v-progress-linear>
                    <p v-if="tasks.length">{{ completedTasks }} / {{ tasks.length }} complete</p>

                    <v-row v-if="!taskSearch" no-gutters>
                        <v-text-field data-test-create-input label="Create task" v-model="newTask" @keyup.enter="createTask(newTask)">
                            <v-icon
                                @click="createTask(newTask)"
                                slot="append"
                                color="green"
                                data-test-create-button
                            >
                                mdi-plus
                            </v-icon>
                        </v-text-field>
                    </v-row>

                    <p v-if="taskSearch && tasks.length" class="mt-5">✨ {{ filteredTasks.length }} tasks filtered</p>

                    <div v-if="tasks.length" data-test-tasks>
                        <v-row v-for="task in filteredTasks" :key="task.index" no-gutters align="center" data-test-task>
                            <v-checkbox @click="task.completed != task.completed" hide-details v-model="task.completed" data-test-task-checkbox></v-checkbox>
                            <v-text-field hide-details v-model="task.title" @keyup.enter="$event.target.blur()" :class="{ 'task--completed' : task.completed }" data-test-task-input>
                                <v-icon
                                    @click="deleteTask(task)"
                                    slot="append"
                                    color="red"
                                >
                                    mdi-close
                                </v-icon>
                            </v-text-field>
                        </v-row>
                    </div>
                </div>
            </v-container>
        </v-main>
    </v-app>
</template>

<style>
    .task--completed .v-text-field__slot { text-decoration: line-through; }
</style>

<script>
import axios from 'axios';

export default {
    name: 'Tasks',

    data: () => ({
        newTask: '',
        tasks: [],
        taskSearch: '',
        hasError: false,
    }),

    methods: {
        createTask(task) {
            this.tasks.unshift({
                title: task,
                completed: false
            });
            this.newTask = '';
        },

        deleteTask(task) {
            let index = this.tasks.indexOf(task);
            this.tasks.splice(index, 1)
        },

        getTasks() {
            axios.get('https://jsonplaceholder.typicode.com/todos')
                .then(response => {
                    this.tasks = response.data;
                    this.hasError = false;
                })
                .catch(error => {
                    console.log(error);
                    this.hasError = true;
                }
            );
        }
    },

    created() {
        this.getTasks();
    },

    computed: {
        filteredTasks() {
            return this.tasks.filter(task => {
                return task.title.indexOf(this.taskSearch) > -1;
            })
        },

        completedTasks() {
            return this.tasks.filter(task => {
                return task.completed === true
            }).length;
        },

        progress() {
            return this.completedTasks / this.tasks.length * 100;
        }
    }
};
</script>
