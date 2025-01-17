<template>
  <ul class="ul-header">
    <li class="icon">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512">
        
        <path
          fill="#ffffff"
          d="M0 96C0 78.3 14.3 64 32 64l384 0c17.7 0 32 14.3 32 32s-14.3 32-32 32L32 128C14.3 128 0 113.7 0 96zM0 256c0-17.7 14.3-32 32-32l384 0c17.7 0 32 14.3 32 32s-14.3 32-32 32L32 288c-17.7 0-32-14.3-32-32zM448 416c0 17.7-14.3 32-32 32L32 448c-17.7 0-32-14.3-32-32s14.3-32 32-32l384 0c17.7 0 32 14.3 32 32z"
        />
      </svg>
    </li>
    <li>X GPT</li>
    <li class="icon">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512">
        
        <path
          fill="#ffffff"
          d="M224 256A128 128 0 1 0 224 0a128 128 0 1 0 0 256zm-45.7 48C79.8 304 0 383.8 0 482.3C0 498.7 13.3 512 29.7 512l388.6 0c16.4 0 29.7-13.3 29.7-29.7C448 383.8 368.2 304 269.7 304l-91.4 0z"
        />
      </svg>
    </li>
  </ul>
</template>

<script setup>

</script>

<style scoped>
@keyframes animateHeader {
  to {
    translate: 0px 0px;
    opacity: 1;
  }
  from {
    translate: 0px -50px;
    opacity: 0.1;
  }
}

.ul-header {
  width: 100%;
  height: 60px;
  display: flex;
  color: white;
  font-family: 'Kanit';
  font-size: 20px;
  z-index: 1;
  position: fixed;
  animation: animateHeader 0.8s ease alternate;
  background-color: #21252b;
  border-bottom: 1px solid grey;
  justify-content: space-between;
  align-items: center;
}
.icon {
    width: 20px;
    margin-left: 15px;
    margin-right: 15px;
    height: 20px;
}
</style>
