<template class="no-drag">
    <div>
        <div class="top-menu">
            score <span class="score">{{ score }}</span>
        </div>
        <div class="game-screen" v-if="gameData.gameSource !== null">
            <!-- 게임영역 -->
            <iframe
                :src="gameData.gameSource"
                width="100%"
                height="100%"
                class="in-game"
                ref="gameScreen"
                :class="isDimmed === true ? 'dimmed on' : 'dimmed off'"
            />
            <!-- 게임 로딩 후 보여지는 화면  -->
            <div
                :class="
                    isBackground === false ? 'background off' : 'loading-div'
                "
                :style="{
                    backgroundImage: 'url(' + gameData.thumbnailImg + ')',
                }"
            >
                <div
                    :class="
                        isBackground === false
                            ? 'game-info-div off'
                            : 'game-info-div game-info-img'
                    "
                >
                    <div>
                        <img
                            class="no-drag thumbnail-img"
                            :src="gameData.thumbnailImg"
                        />
                        <div class="game-title">{{ gameData.gameTitle }}</div>
                    </div>

                    <div @click="playGame" v-show="type === '@gameReady'">
                        <img
                            class="no-drag play-btn"
                            :src="playBtnSrc"
                            @mousedown="chgPlayBtn"
                            @mouseout="returnPlayBtn"
                            @mouseup="returnPlayBtn"
                            @mouseenter="playHover"
                        />
                    </div>
                    <div v-show="type !== '@gameReady'" class="spinner-div">
                        <div class="loadingio-spinner-eclipse-ebw4655yzwb">
                            <div class="ldio-13e9g3vw80s">
                                <div></div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- 게임 종료 후 보여지는 화면 -->
            <div class="fin-container" v-if="type === '@gameOver'">
                <div class="fin-wrap">
                    <div class="fin-thumb-div">
                        <img
                            class="no-drag fin-thumb-img"
                            :src="gameData.thumbnailImg"
                        />
                    </div>

                    <div class="retry-btn-container" @click="playRetry">
                        <img
                            class="no-drag retry-btn"
                            :src="retryBtnSrc"
                            @mouseout="returnBtn"
                            @mouseenter="retryHover"
                            @mousedown="chgRetryBtn"
                            @mouseup="returnBtn"
                        />
                    </div>
                </div>
            </div>
        </div>
        <!-- 하단 로고 -->
        <div :class="logon === true ? 'logo-container' : 'logo-container off'">
            <span class="no-drag" @click="moveZempiePage">
                <img
                    src="img/common/zempie-logo.png"
                    class="no-drag bottom-logo"
                />
            </span>
        </div>
    </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";

interface GameData {
    gameTitle: string;
    gameSource: string;
    thumbnailImg: string;
    endGameAd: string;
    restartGameAd: string;
    playGameAd: string;
}

@Component
export default class Launcher extends Vue {
    score = 0;
    type = "";
    // vue control
    isDimmed = false;
    isBackground = true;
    logon = true;

    //sdk, iframe 로딩 확인용 변수
    isSdkLoadDone = false;
    isGameLoadDone = false;

    gameData: GameData = {
        gameTitle: "",
        gameSource: "",
        thumbnailImg: "",
        endGameAd: "",
        restartGameAd: "",
        playGameAd: "",
    };

    playBtnSrc = "img/common/btn_play_normal.png";
    retryBtnSrc = "img/common/btn_retry_normal.png";

    // 이미지 변경 부분
    returnPlayBtn(): void {
        this.playBtnSrc = "img/common/btn_play_normal.png";
    }
    playHover(): void {
        this.playBtnSrc = "img/common/btn_play_focused.png";
    }
    chgPlayBtn(): void {
        this.playBtnSrc = "img/common/btn_play_pressed.png";
    }
    retryHover(): void {
        this.retryBtnSrc = "img/common/btn_retry_focused.png";
    }
    chgRetryBtn(): void {
        this.retryBtnSrc = "img/common/btn_retry_pressed.png";
    }
    returnBtn(): void {
        this.retryBtnSrc = "img/common/btn_retry_normal.png";
    }
    getIframeElement(): HTMLIFrameElement {
        return this.$refs.gameScreen as HTMLIFrameElement;
    }
    // load json file
    readTextFile(): void {
        const txtFile = new XMLHttpRequest();
        txtFile.open("GET", "gameData.json", true);
        txtFile.responseType = "json";

        txtFile.onload = () => {
            this.gameData.gameTitle = txtFile.response.gameTitle;
            this.gameData.gameSource = txtFile.response.gameSource;
            this.gameData.thumbnailImg = txtFile.response.thumbnailImg;
            this.gameData.endGameAd = txtFile.response.endGameAd;
            this.gameData.restartGameAd = txtFile.response.restartGameAd;
            this.gameData.playGameAd = txtFile.response.playGameAd;
        };

        txtFile.send();
    }

    //game message
    onMessage(message: MessageEvent): void {
        if (message.data.type !== undefined) {
            this.type = message.data.type;
            switch (this.type) {
                case "@gameReady": {
                    this.isDimmed = true;
                    this.isGameLoadDone = true;
                    //@ts-ignore
                    window.start()
                    break;
                }
                case "@updateScore": {
                    this.score = message.data.score;
                    this.logon = false;
                    break;
                }
                case "@gameOver": {
                    if (this.gameData.endGameAd === "true") {
                        this.isDimmed = true;
                        this.score = message.data.score;
                        this.logon = false;
                    }

                    break;
                }
            }
        }
    }
    // 게임 시작
    playGame(): void {
        this.getIframeElement().contentWindow?.postMessage(
            { type: "@gamePlay" },
            "*"
        );
        this.isBackground = false;
        this.isDimmed = false;
        this.logon = false;

        // if (this.gameData.playGameAd === "true") {
        // }
    }
    // 게임 재시작
    playRetry(): void {
        this.getIframeElement().contentWindow?.postMessage(
            { type: "@gameRetry" },
            "*"
        );
        if (this.gameData.restartGameAd === "true") {
        }
        this.isDimmed = false;
        this.logon = false;
    }
    //게임 일시정지
    pauseGame(): void {
        this.getIframeElement().contentWindow?.postMessage(
            { type: "@gamePause" },
            "*"
        );
        this.isDimmed = true;
    }

    //게임 resume
    resumeGame(): void {
        this.getIframeElement().contentWindow?.postMessage(
            { type: "@gameResume" },
            "*"
        );
        this.isDimmed = false;
        this.logon = false;
    }

    adStarted() {
        this.getIframeElement().contentWindow?.postMessage(
            { type: "@soundOff" },
            "*"
        );
        this.getIframeElement().contentWindow?.postMessage(
            { type: "@gamePause" },
            "*"
        );
    }

    adError() {
        this.getIframeElement().contentWindow?.postMessage(
            { type: "@soundOn" },
            "*"
        );
        this.getIframeElement().contentWindow?.postMessage(
            { type: "@gameResume" },
            "*"
        );
    }

    adFinished() {
        this.getIframeElement().contentWindow?.postMessage(
            { type: "@soundOn" },
            "*"
        );

        if (this.gameData.endGameAd === "false") {
            console.log("?");
            this.getIframeElement().contentWindow?.postMessage(
                { type: "@gameResume" },
                "*"
            );
        }
    }

    async mounted() {
        window.addEventListener("message", this.onMessage);

        this.readTextFile();
    }
    beforeDestroy() {
        window.removeEventListener("message", this.onMessage);
    }

    moveZempiePage(): void {
        window.open("https://zempie.com");
    }
}
</script>

<style scoped>
@import "../css/launcher.css";
</style>
