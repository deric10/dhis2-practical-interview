<template>
    <button>{{ title }}</button>
</template>

<script>
export default {
    props: ['title']
}
</script>

<style scoped>
button {
    border: none;
    padding: 8px 24px;
    border-radius: 5px;
    background-color: var(--primary);
    color: var(--white);
    font-weight: bold;
    cursor: pointer;
    transition: 0.3s;
    margin: 8px;
    width: 127.36px;
    height: 37px;
    font-size: 14px;
}

button:hover {
    background-color: var(--primary-dark);
}
</style>