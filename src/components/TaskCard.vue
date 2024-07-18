<script setup>
import UpdateIcon from '../assets/icons/update_icon.png';
import DeleteIcon from '../assets/icons/delete_icon.png';

const {task} = defineProps(['task']);
const emits = defineEmits(['update_task', 'delete_task']);

const openUpdateTaskFormModal = () => {
    emits('update_task', task.id);
}

const deleteTaskData = () => {
    emits('delete_task', task.id);
}

const onDragStart = (event) => {
    event.dataTransfer.setData('task_id', task.id);
};

</script>

<template>
    <div class="task_card" draggable="true" @dragstart="onDragStart" data-aos="zoom-in" data-aos-duration="500" data-aos-once="true">
        <div class="task_card_text">
            <li class="task_card_title">{{ task.task_name }}</li>
            <li class="task_card_description">{{ task.task_description }}</li>
        </div>
        <div class="task_card_due_date">
            <span class="task_card_due_date_text">Due Date:</span><span class="task_card_date"> {{ task.task_due_date }}</span>
        </div>
        <div class="task_card_cta_btns">
            <button class="task_card_cta_btn task_card_update_btn" @click="openUpdateTaskFormModal">
                <img :src="UpdateIcon" alt="update_icon" class="task_card_cta_icon">
            </button>
            <button class="task_card_cta_btn task_card_delete_btn" @click="deleteTaskData" >
                <img :src="DeleteIcon" alt="delete_icon" class="task_card_cta_icon">
            </button>
        </div>
    </div>
</template>

<style scoped>
.task_card 
{
    padding: 20px;
    background-color: rgb(var(--clr-light), 0.8);
    border-radius: 5px;
    position: relative;
    cursor: grab;
}

.task_card:active
{
    cursor: grabbing;
}

.task_card .task_card_text
{
    margin-bottom: 20px;
}

.task_card .task_card_text .task_card_title
{
    margin-bottom: 10px;
    font-weight: var(--fw-bold);
}

.task_card .task_card_text .task_card_description
{
    font-size: var(--fs-base);
}

.task_card .task_card_due_date 
{
    display: flex;
    justify-content: flex-end;
    align-items: flex-end;
    font-size: var(--fs-small);
}

.task_card .task_card_due_date .task_card_due_date_text
{
    margin-right: 3px;
}

.task_card .task_card_due_date .task_card_date
{
    font-weight: var(--fw-bold);
}

.task_card .task_card_cta_btns
{
    position: absolute;
    top: -15px;
    right: -5px;
    display: flex;
    align-items: center;
    column-gap: 5px;
}

.task_card .task_card_cta_btns .task_card_cta_btn
{
    width: 35px;
    height: 35px;
    border-radius: 50%;
}

.task_card .task_card_cta_btns .task_card_update_btn
{
    background-color: rgb(var(--clr-dark-orange));
}

.task_card .task_card_cta_btns .task_card_delete_btn
{
    background-color: rgb(var(--clr-red));
}

@keyframes wiggle {
    0% {
        transform: rotate(10deg);
        transform: scale(1.1);
    }
    50% {
        transform: rotate(-10deg);
    }
    100% {
        transform: rotate(0deg);
    }
}

.task_card .task_card_cta_btns .task_card_cta_btn .task_card_cta_icon
{
    width: 18px;
    filter: invert(100%);
}

.task_card .task_card_cta_btns .task_card_cta_btn:hover .task_card_cta_icon
{
    animation: wiggle 0.5s ease-in-out;
}

</style>