<template>
    <div id="app">
        <CalculatorComponent />
    </div>
</template>

<script>
import CalculatorComponent from './components/CalculatorComponent.vue';

export default {
    name: 'App',
    components: {
        CalculatorComponent
    }
};
</script>

<style>
@font-face {
    font-family: 'SF Pro Display';
    src: url('@/assets/fonts/SFProDisplay-Regular.woff');
}

body {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background-color: #000;
    margin: 0;
    font-family: 'SF Pro Display', Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}

#app {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
    width: 100%;
}
</style>
