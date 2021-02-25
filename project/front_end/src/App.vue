<template>
    <div id="app">
        <BaseMap :active_filters="filters" />
        <UIOverlay @revaluate-filters="filters = $event" />
    </div>
</template>

<script>
import BaseMap from "./components/BaseMap.vue";
import UIOverlay from "./components/UIOverlay.vue";

export default {
    name: "app",
    components: {
        BaseMap,
        UIOverlay,
    },
    data() {
        return {
            filters: null,
        };
    },
};
</script>

<style>
#app {
    position: absolute;
    left: 0px;
    top: 0px;
    font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
    color: #2c3e50;
    height: 100vh;
    width: 100vw;
}
</style>
