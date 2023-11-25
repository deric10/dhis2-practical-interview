<template>
    <footer class="footer">
        <nav class="footer__nav">
            <router-link to="/weather" class="nav__item">
                <font-awesome-icon icon="fa-regular fa-sun" class="item__icon" />
                <small class="item__text">Weather</small>
            </router-link>
        </nav>
    </footer>
</template>

<script>
export default {}
</script>

<style scoped>
.footer {
    border-block-start: 1px solid var(--hover);
}

.footer__nav {
    max-width: 1200px;
    margin: 0 auto;
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-template-rows: 1;
    gap: 16px;
    padding-inline: 24px;
    padding-block: 8px;
}

.nav__item {
    border-radius: 8px;
    padding: 8px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-decoration: none;
    color: var(--font-color);
}


.item__text {
    margin-block-start: 8px;
}

.router-link-active {
    color: var(--primary)
}

@media screen and (min-width: 1024px) {
    .nav__item:hover {
        background-color: var(--hover);
    }
}
</style>