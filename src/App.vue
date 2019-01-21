<template>
    <div id="app">
        <Calculator></Calculator>
    </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Calculator from './components/Calculator.vue';

@Component({
    components: {
        Calculator,
    }
})
export default class App extends Vue {}
</script>

<style lang="less">
html, body, #app {
    width: 100%;
    height: 100%;
    overflow: hidden;
}
html {
    font-family: 'local-file', 'Arial', 'Verdana', '微软雅黑', '黑体', 'serif';
    color: #111;
}
#app {
    background-image: url('./assets/background.jpg');
    background-size: cover;
}
html, body, div, p, h1, h2, h3, h4 {
    margin: 0;
}
</style>
