<script setup>
import { onMounted, ref } from 'vue';
import axios from 'axios';
import TaskCard from './TaskCard.vue';
import AddIcon from '../assets/icons/add_icon.png';
import CloseIcon from '../assets/icons/close_icon.png';

const {title, subTitle} = defineProps({title: String, subTitle: String})

const api_url = 'http://localhost:3000/tasks';

const addTaskFormModal = ref(false);
const updateTaskFormModal = ref(false);
const tasks = ref([]);
const task_name = ref("");
const task_description = ref("");
const task_due_date = ref("");
const updated_task_id = ref(null);
const task_status = ref("");

// Reset Task Form Modal
const resetTaskFormModal = () => {
    task_name.value = "";
    task_description.value = "";
    task_due_date.value = "";
}

// Add Task Data
const openAddTaskFormModal = () => {
    addTaskFormModal.value = true;
    document.body.classList.add('overflow_hidden');
};

const closeAddTaskFormModal = () => {
    addTaskFormModal.value = false;
    resetTaskFormModal();
    document.body.classList.remove('overflow_hidden');
};

const addTaskData = async () => {
    try {
        if(task_name.value.trim() !== "" && task_description.value.trim() !== "") {
            const new_task = {
                task_name: task_name.value.trim(),
                task_description: task_description.value.trim(),
                task_status: 'pending',
                task_due_date: task_due_date.value,
                task_due_date_updated: new Date().toISOString()
        };
            const res = await axios.post(api_url, new_task);
            tasks.value.push(res.data);
            
            sortTasks();
            resetTaskFormModal();
            addTaskFormModal.value = false;
            document.body.classList.remove('overflow_hidden');
        }
    }
    catch (error) {
        console.log("Error adding task data", error);
    }
}

// Update Task Data
const openUpdateTaskFormModal = (task_id) => {
    const task = tasks.value.find(t => t.id === task_id);
    if(task) {
        updated_task_id.value = task_id;
        task_name.value = task.task_name;
        task_description.value = task.task_description;
        task_due_date.value = task.task_due_date;
        task_status.value = task.task_status.toLowerCase();
        updateTaskFormModal.value = true;
        document.body.classList.add('overflow_hidden');
    }
}

const closeUpdateTaskFormModal = () => {
    updateTaskFormModal.value = false;
    resetTaskFormModal();
    document.body.classList.remove('overflow_hidden');
};

const updateTaskData = async () => {
    try {
        if(updated_task_id.value && task_name.value.trim() !== "" && task_description.value.trim() !== "")
        {
            const update_task = {
                task_name: task_name.value.trim(),
                task_description: task_description.value.trim(),
                task_status: task_status.value.trim().toLowerCase(),
                task_due_date: task_due_date.value,
                task_due_date_updated: new Date().toISOString()
            }
            const res = await axios.put(`${api_url}/${updated_task_id.value}`, update_task);

            const index = tasks.value.findIndex(t => t.id === updated_task_id.value);
            if(index !== -1) {
                tasks.value[index] = res.data;
            }
            sortTasks();
            closeUpdateTaskFormModal();
            document.body.classList.remove('overflow_hidden');
        }

    } catch (error) {
        console.log("Error editing task data", error);
    }
}

// Delete Task Data
const deleteTaskData = async (task_id) => {
    try {
        await axios.delete(`${api_url}/${task_id}`);
        tasks.value = tasks.value.filter(task => task.id !== task_id);
    } catch(error) {
        console.log("Error deleting task data", error);
    }
}

// Fetch Tasks Data
const fetchTasks = async () => {
  try {
    const res = await axios.get(api_url);
    tasks.value = res.data;
    sortTasks();
  } catch (error) {
    console.log("Error fetching tasks data", error);
  }
}

const onDropTaskCard = (event, status) => {
    const task_id = event.dataTransfer.getData('task_id');
    const task = tasks.value.find(t => t.id === task_id);
    if (task) {
        updated_task_id.value = task_id;
        task_name.value = task.task_name;
        task_description.value = task.task_description;
        task_due_date.value = task.task_due_date;
        task_status.value = status;
        updateTaskData(status);
    }
}

const sortTasks = () => {
    tasks.value.sort((a, b) => new Date(b.task_due_date_updated) - new Date(a.task_due_date_updated));
}   

const filtered_tasks = (status) => {
    return tasks.value.filter(task => task.task_status.toLowerCase() === status.toLowerCase());
}

onMounted(fetchTasks);
</script>

<template>
  <main class="main">
    <div class="main_container">
        <header class="header">
            <div class="header_text">
                <h1 class="header_title">{{ title }}</h1>
            <p class="header_description">{{ subTitle }}</p>
            </div>
            <button class="add_task_btn" @click="openAddTaskFormModal"><img :src="AddIcon" alt="add_icon" class="add_icon"> <span class="add_task_text">Add Task</span></button>
        </header>
        <div class="tasks_status">
            <div class="tasks_status_item pending_tasks"  @dragover.prevent @drop="(event) => onDropTaskCard(event, 'pending')">
                <h2 class="tasks_status_item_title">Pending Tasks</h2>
                <ul class="task_card_list">
                    <TaskCard v-for="task in filtered_tasks('pending')" :key="task.id" :task="task" @update_task="openUpdateTaskFormModal" @delete_task="deleteTaskData"/>
                </ul>
            </div>
            <div class="tasks_status_item in_progress_tasks" @dragover.prevent @drop="(event) => onDropTaskCard(event, 'in progress')">
                <h2 class="tasks_status_item_title">In Progress Tasks</h2>
                <ul class="task_card_list">
                    <TaskCard v-for="task in filtered_tasks('in progress')" :key="task.id" :task="task" @update_task="openUpdateTaskFormModal" @delete_task="deleteTaskData"/>
                </ul>
            </div>
            <div class="tasks_status_item completed_tasks" @dragover.prevent @drop="(event) => onDropTaskCard(event, 'completed')">
                <h2 class="tasks_status_item_title">Completed Tasks</h2>
                <ul class="task_card_list">
                    <TaskCard v-for="task in filtered_tasks('completed')" :key="task.id" :task="task" @update_task="openUpdateTaskFormModal" @delete_task="deleteTaskData"/>
                </ul>
            </div>
            <div class="tasks_status_item cancelled_tasks" @dragover.prevent @drop="(event) => onDropTaskCard(event, 'cancelled')">
                <h2 class="tasks_status_item_title">Cancelled Tasks</h2>
                <ul class="task_card_list">
                    <TaskCard v-for="task in filtered_tasks('cancelled')" :key="task.id" :task="task" @update_task="openUpdateTaskFormModal" @delete_task="deleteTaskData"/>
                </ul>
            </div>
        </div>
    </div>
  </main>

   <!-- Modal Add Task -->
   <div class="overlay add_task_form_modal" v-show="addTaskFormModal">
        <form @submit.prevent="addTaskData" class="modal_form">
            <header class="modal_header">
                <h1 class="modal_header_title">Add Task</h1>
                <button class="modal_close_btn" @click="closeAddTaskFormModal"><img :src="CloseIcon" alt="close_icon" class="modal_close_icon"></button>
            </header>
            <input type="text" v-model.trim="task_name" class="modal_task_input" placeholder="Input Task Name">
            <input type="text" v-model.trim="task_description" class="modal_task_input" placeholder="Input Task Description">
            <input type="date" v-model="task_due_date" class="modal_task_input">
            <button type="submit" class="modal_submit_btn">Add Task</button>
        </form>
    </div>

    <!-- Modal Update Task -->
    <div class="overlay update_task_form_modal" v-show="updateTaskFormModal">
        <form @submit.prevent="updateTaskData" class="modal_form">
            <header class="modal_header">
                <h1 class="modal_header_title">Update Task</h1>
                <button class="modal_close_btn" @click="closeUpdateTaskFormModal"><img :src="CloseIcon" alt="close_icon" class="modal_close_icon"></button>
            </header>
            <input type="text" v-model.trim="task_name" placeholder="Input Task Name" class="modal_task_input">
            <input type="text" v-model.trim="task_description" placeholder="Input Task Description" class="modal_task_input"> 
            <input type="date" v-model="task_due_date" class="modal_task_input">
            <select v-model="task_status" class="modal_select_status">
                <option value="pending" class="modal_status_select_option pending_option">Pending</option>
                <option value="in progress" class="modal_status_select_option in_progress_option">In Progress</option>
                <option value="completed" class="modal_status_select_option completed_option">Completed</option>
                <option value="cancelled" class="modal_status_select_option cancelled_option">Cancelled</option>
            </select>
            <button type="submit" class="modal_submit_btn">Update Task</button>
        </form>
    </div>

</template>

<style scoped>
.main .main_container 
{
    padding-inline: 40px;
    padding-block: 20px; 
}

.main .main_container .header
{
    margin-bottom: 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 20px;
}

.main .main_container .header .header-text .header_title
{
    font-size: var(--fs-title);
    font-weight: var(--fw-bold);
}

.main .main_container .header .add_task_btn
{
    padding: 10px;
    background-color: rgb(var(--clr-green));
    color:  rgb(var(--clr-light));
    display: flex;
    justify-content: center;
    align-items: center;
    column-gap: 5px;
    z-index: 999;
}

.main .main_container .header .add_task_btn .add_icon 
{
    filter: invert(100%);
    width: 20px;
    transition-duration: 0.5s;
}

.main .main_container .header .add_task_btn:hover .add_icon 
{
    transform: scale(0.8);
}

.main .main_container .header .add_task_btn .add_task_text
{
    font-weight: var(--fw-bold);
}

.tasks_status 
{
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;
}

.main .main_container .tasks_status .tasks_status_item
{
    border-radius: 10px;
    width: 100%;
    height: auto;
    padding: 20px 15px;
}

.main .main_container .tasks_status .pending_tasks
{
    background-color: rgb(var(--clr-light-orange));
}

.main .main_container .tasks_status .in_progress_tasks
{
    background-color: rgb(var(--clr-light-blue));
}

.main .main_container .tasks_status .completed_tasks
{
    background-color: rgb(var(--clr-light-green));
}

.main .main_container .tasks_status .cancelled_tasks
{
    background-color: rgb(var(--clr-light-red));
}

.main .main_container .tasks_status .tasks_status_item_title
{
    font-size: var(--fs-base);
    font-weight: var(--fw-bold);
    margin-bottom: 10px;
}

.main .main_container .tasks_status .tasks_status_item .task_card_list
{
    display: flex;
    flex-direction: column;
    row-gap: 20px;
}




/* Modal */
.overlay
{
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0,0,0,0.8);
    display: flex;
    align-items: center;
    justify-content: center;
    padding-inline: 20px;
    z-index: 9999;
}

.overlay .modal_form 
{
    background-color: rgb(var(--clr-light));
    padding: 20px;
    border-radius: 20px;
    display: flex;
    flex-direction: column;
    row-gap: 20px;
    width: 100%;
    max-width: 500px;
    animation: slide_down 0.3s ease-in-out;
}

@keyframes slide_down {
    0%
    {
        transform: translateY(-100%);
    }
    100% {
        transform: translateY(0%);
    }
}

.overlay .modal_form .modal_header
{
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.overlay .modal_form .modal_header .modal_header_title 
{
    color: rgb(var(--clr-green));
}

.overlay .modal_form .modal_header .modal_close_btn
{
    padding: 5px;
    border-radius: 5px;
    background-color: rgb(var(--clr-red));
    color: rgb(var(--clr-light));
    display: flex;
    justify-content: center;
    align-items: center;
}

.overlay .modal_form .modal_header .modal_close_btn .modal_close_icon
{
    width: 20px;
    filter: invert(100%);
    transition-duration: 0.5s;
}

.overlay .modal_form .modal_header .modal_close_btn:hover .modal_close_icon
{
    transform: scale(0.8);
}

.overlay .modal_form .modal_task_input
{
    padding: 15px 20px;
    width: 100%;
    background-color: rgb(var(--clr-gray));
}

.overlay .modal_form .modal_select_status 
{
    padding: 15px 20px;
    max-width: 200px;
    background-color: rgb(var(--clr-gray));
}

.overlay .modal_form .modal_select_status .modal_status_select_option {
    font-size: var(--fs-base);
}

.overlay .modal_form .modal_select_status .pending_option 
{
    background-color: rgb(var(--clr-light-orange));
}

.overlay .modal_form .modal_select_status .in_progress_option 
{
    background-color: rgb(var(--clr-light-blue));
}

.overlay .modal_form .modal_select_status .completed_option 
{
    background-color: rgb(var(--clr-light-green));
}

.overlay .modal_form .modal_select_status .cancelled_option 
{
    background-color: rgb(var(--clr-light-red));
}

.overlay .modal_form .modal_submit_btn
{
    background-color: rgb(var(--clr-green));
    color: rgb(var(--clr-light));
    padding: 10px;
    font-weight: var(--fw-bold);
}

@media (max-width: 1023px)
{
    .main .main_container .header .add_task_btn
    {
        position: fixed;
        right: 20px;
        bottom: 20px;
        box-shadow: rgba(0, 0, 0, 0.16) 0px 3px 6px, rgba(0, 0, 0, 0.23) 0px 3px 6px;
    }

    .main .main_container .header .add_task_btn .add_task_text
    {
        display: none;
    }

    .tasks_status
    {
        grid-template-columns: 1fr;
    }
    .main .main_container 
    {
        padding-inline: 20px;
    }
}

</style>
