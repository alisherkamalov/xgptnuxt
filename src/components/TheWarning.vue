<template>
    <div class="warning-center"
    :style="{
        opacity: opacityWarning,
        translate: translateWarning
    }">
        <div class="warning">
            Сообщение не должно быть пустым!
        </div>
    </div>
    
</template>

<script setup>
import { defineProps } from 'vue';

const propsWarning = defineProps({
  opacityWarning: {
    type: Number,
    default: 0.1
  },
  translateWarning: {
    type: String,
    default: '0px -55px'
  }
});
</script>

<style>
.warning-center {
    width: 95%;
    position: absolute;
    z-index: 2;
    height: 50px;
    padding-left: 15px;
    display: flex;
    justify-content: start;
    transition: all 0.5s ease;
}
.warning {
    width: 92.5%;
    max-width: 500px;
    height: 50px;
    font-family: 'Kanit', 'Inter';
    color: white;
    display: flex;
    align-items: center;
    border-radius: 7px;
    padding-left: 15px;
    background-color: #2E2E2E;
    box-shadow: rgba(255, 255, 255, 0.226) 0px 6px 12px -2px, rgba(0, 0, 0, 0) 0px 3px 7px -3px;
}
</style>