<template>
    <div id="app">
        <Launcher />
    </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Launcher from "./components/Launcher.vue";

@Component({
    components: {
        Launcher,
    },
})
export default class App extends Vue {
    gameTitle: string = "";

    readTextFile() {
        const txtFile = new XMLHttpRequest();
        txtFile.open("GET", "gameData.json", true);
        txtFile.responseType = "json";

        txtFile.onload = () => {
            document.title = txtFile.response.gameTitle;
        };

        txtFile.send();
    }
    mounted() {
        this.readTextFile();
    }

  
}
</script>



<style lang="scss">
</style>
