<template>
    <div class="blur"></div>
</template>

<script setup>

</script>

<style scoped>
.blur {
    width: 100%;
    height: 100%;
    z-index: 2;
    position: absolute;
    background: rgba(255, 255, 255, 0.1); 
    backdrop-filter: blur(10px); 
    -webkit-backdrop-filter: blur(10px);
    transition: all 0.1s ease;
}
</style>