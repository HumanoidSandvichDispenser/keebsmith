<template>
    <TypingTest/>
</template>

<script>
import TypingTest from "./components/TypingTest.vue"

export default {
    name: "App",
    components: {
        TypingTest
    }
};
</script>

<style>
body {
    background-color: var(--background-color);
    color: var(--primary-color);
}

#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    margin-top: 60px;
}
</style>