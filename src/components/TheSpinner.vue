<template>
    <transition name="fade">
        <span class="loader"></span>
    </transition>
</template>

<script>
export default {}
</script>

<style scoped>
.loader {
    width: 30px;
    height: 30px;
    border: 2px solid var(--primary);
    border-bottom-color: transparent;
    border-radius: 50%;
    display: inline-block;
    box-sizing: border-box;
    animation: rotation 1s linear infinite;
}



@keyframes rotation {
    0% {
        transform: rotate(0deg);
    }

    100% {
        transform: rotate(360deg);
    }
}
</style>