<template>
        <div class="ui-video" @click="clickEvent">
            <video ref="video" webkit-playsinline playsinline
                   :src="src"
                   :poster="poster" @timeupdate="updateTime"
                   @playing="playingEvent" @pause="pauseEvent"></video>
            <div class="loading" v-show="isLoading">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
                    <path fill="none" d="M0 0h100v100H0z"/>
                    <rect width="7" height="20" x="46.5" y="40" fill="#E9E9E9" rx="5" ry="5"
                          transform="translate(0 -30)"/>
                    <rect width="7" height="20" x="46.5" y="40" fill="#989697" rx="5" ry="5"
                          transform="rotate(30 105.98 65)"/>
                    <rect width="7" height="20" x="46.5" y="40" fill="#9B999A" rx="5" ry="5"
                          transform="rotate(60 75.98 65)"/>
                    <rect width="7" height="20" x="46.5" y="40" fill="#A3A1A2" rx="5" ry="5"
                          transform="rotate(90 65 65)"/>
                    <rect width="7" height="20" x="46.5" y="40" fill="#ABA9AA" rx="5" ry="5"
                          transform="rotate(120 58.66 65)"/>
                    <rect width="7" height="20" x="46.5" y="40" fill="#B2B2B2" rx="5" ry="5"
                          transform="rotate(150 54.02 65)"/>
                    <rect width="7" height="20" x="46.5" y="40" fill="#BAB8B9" rx="5" ry="5"
                          transform="rotate(180 50 65)"/>
                    <rect width="7" height="20" x="46.5" y="40" fill="#C2C0C1" rx="5" ry="5"
                          transform="rotate(-150 45.98 65)"/>
                    <rect width="7" height="20" x="46.5" y="40" fill="#CBCBCB" rx="5" ry="5"
                          transform="rotate(-120 41.34 65)"/>
                    <rect width="7" height="20" x="46.5" y="40" fill="#D2D2D2" rx="5" ry="5"
                          transform="rotate(-90 35 65)"/>
                    <rect width="7" height="20" x="46.5" y="40" fill="#DADADA" rx="5" ry="5"
                          transform="rotate(-60 24.02 65)"/>
                    <rect width="7" height="20" x="46.5" y="40" fill="#E2E2E2" rx="5" ry="5"
                          transform="rotate(-30 -5.98 65)"/>
                </svg>
            </div>
            <div class="ui-video-bar" :class="{hidden:!videoBar}">
                <div class="video-controls">
                    <div class="control-button" :class="{pause:isPlay,play:!isPlay}" @click="playOrPause"></div>
                    <div class="current-time">{{format(CurrentTime)}}</div>
                    <div class="progress-container">
                        <div class="progress" ref="progressBar">
                            <div class="buffered" :style="{width:videoMetadata+'%'}"></div>
                            <div class="video-ball" :style="{left:playProgress+'%'}" @touchstart="sliderStart"
                                 @touchmove="sliderMove" @touchend="sliderEnd">
                                <div></div>
                            </div>
                            <div class="played" ref="playedBar" :style="{width:playProgress+'%'}"></div>
                        </div>
                    </div>
                    <div class="duration-time">{{format(videoTime)}}</div>
                </div>
                <div class="fullscreen-icon" @click="fullScreen"></div>
            </div>
        </div>
</template>
<script>
    export default {
        data() {
            return {
                playProgress: 0,
                CurrentTime: 0,
                videoTime: 0,
                videoDome: '',
                touch: [],
                videoMetadata: 0,
                isPlay: false,
                videoBar: true,
                hiddenBarTime: '',
                isLoading: false
            }
        },
        props: ['src','poster'],
        mounted() {
            this.videoDome = this.$refs.video;
            this.videoDome.load();
            document.addEventListener("WeixinJSBridgeReady", () => {
                this.videoDome.load()
            }, false);
            this.videoDome.addEventListener('progress', () => {
                setTimeout(() => {
                    let v = this.videoDome;
                    let r = v.buffered;
                    let total = v.duration;
                    try {
                        let end = r.end(0);
                        this.videoTime = total;
                        this.videoMetadata = (end / total) * 100;
                    } catch (e) {
                    }
                }, 100)
            }, false);
            this.videoDome.addEventListener('seeked', () => {
                if (this.videoDome.readyState >= 2) {
                    this.isLoading = false;
                }
            });
            this.videoDome.addEventListener('waiting', _ => {
                this.isLoading = true;
            })
        },
        watch: {},
        methods: {
            playingEvent() {
                this.isLoading = false;
                this.hiddenBarTime = setTimeout(_ => {
                    this.videoBar = false;
                }, 5000);
            },
            clickEvent() {
                if (!this.videoBar) {
                    this.videoBar = true;
                    this.hiddenBarTime = setTimeout(_ => {
                        this.videoBar = false;
                    }, 5000);
                }
            },
            pauseEvent() {
                clearTimeout(this.hiddenBarTime);
                this.isPlay = false;
                this.videoBar = true;
            },
            sliderStart(e) { //开始拖拽
                clearTimeout(this.hiddenBarTime);
                this.videoDome.pause();
                this.isPlay = false;
                this.touch.initiated = true;
                this.touch.startX = e.touches[0].pageX;
                this.touch.left = this.$refs.playedBar.clientWidth / (this.$refs.progressBar.clientWidth / 100);
            },
            sliderMove(e) { //拖拽中
                if (!this.touch.initiated) return;
                let oldX = e.touches[0].pageX - this.touch.startX;
                let slide = Math.min(Math.max(0, this.touch.left + oldX / (this.$refs.progressBar.clientWidth / 100)), 100);
                this.playProgress = slide;
                this.CurrentTime = this.playProgress / 100 * this.videoTime;
            },
            sliderEnd() { //拖拽完毕
                this.touch.initiated = false;
                this.videoDome.currentTime = this.playProgress / 100 * this.videoTime;
                this.videoDome.play();
                this.isPlay = true;
            },
            playOrPause() {
                this.isPlay = !this.isPlay;
                if (this.isPlay) {
                    this.videoDome.play();
                } else {
                    this.videoDome.pause();
                }
            },
            updateTime(e) {
                this.CurrentTime = e.target.currentTime;
                this.playProgress = this.CurrentTime / this.videoTime * 100;
            },
            fullScreen() {
                let dom = this.videoDome;
                this.videoDome.play();
                if (dom.requestFullscreen) {
                    dom.requestFullscreen();
                }
                else if (dom.mozRequestFullScreen) {
                    dom.mozRequestFullScreen();
                }
                else if (dom.webkitRequestFullscreen) {
                    dom.webkitRequestFullscreen();
                }
                else if (dom.webkitEnterFullscreen) {   // Safari for iOS
                    dom.webkitEnterFullscreen();
                }
            },
            // 格式化时间
            format(interval) {
                interval = interval | 0;
                const minute = interval / 60 | 0;
                const second = this._pad(interval % 60);
                return `${minute}:${second}`
            },
            _pad(num, n = 2) {
                let len = num.toString().length;
                while (len < n) {
                    num = '0' + num;
                    len++
                }
                return num
            }
        },
        components: {}
    }
</script>
<style lang="scss">
    .ui-video {
        position: relative;
        top: 0;
        left: 0;
        right: 0;
        overflow: hidden;
        video {
            width: 100%;
            display: block;
        }
        .loading {
            position: absolute;
            left: 50%;
            top: 50%;
            margin: -.5rem 0 0 -.5rem;
            width: 1rem;
            height: 1rem;
            border-radius: .2rem;
            background: rgba(0, 0, 0, .5);
            text-align: center;
            svg {
                width: .8rem;
                margin-top: .1rem;
                animation: loading 1s steps(12, end) infinite;
            }
        }
        .ui-video-bar {
            position: absolute;
            width: 100%;
            height: .9rem;
            background: rgba(0, 0, 0, .5);
            display: flex;
            padding: 0 .2rem;
            bottom: 0;
            transition: all .3s ease;
            &.hidden {
                bottom: -1rem;
            }
            .video-controls {
                position: relative;
                display: flex;
                align-items: center;
                .control-button {
                    width: .26rem;
                    height: .3rem;
                    margin: .3rem .2rem .3rem 0;
                    &.play {
                        background: url("./video_play.png") no-repeat;
                        background-size: 100%;
                    }
                    &.pause {
                        background: url("./video_pause.png") no-repeat;
                        background-size: 100%;
                    }
                }
                .current-time, .duration-time {
                    line-height: normal;
                    position: relative;
                    top: 0;
                    font-size: .26rem;
                    color: #cbcbcb;
                }
                .progress-container {
                    position: relative;
                    width: 4.4rem;
                    margin: .44rem .25rem;
                    .progress {
                        height: .04rem;
                        background: rgba(255, 255, 255, .4);
                        position: relative;
                        .buffered {
                            position: absolute;
                            height: .04rem;
                            background: rgba(255, 255, 255, .6);
                        }
                        .video-ball {
                            position: absolute;
                            width: .3rem;
                            height: .3rem;
                            padding: .3rem;
                            margin: -.43rem 0 0 -.43rem;
                            z-index: 2;
                            div {
                                width: .3rem;
                                height: .3rem;
                                background: #fff;
                                border-radius: .15rem;
                                display: block;
                            }
                        }
                        .played {
                            position: absolute;
                            height: .04rem;
                            background: #fff;
                        }
                    }
                }
            }
            .fullscreen-icon {
                width: .34rem;
                height: .34rem;
                background: url("./video_fullscreen.png") no-repeat;
                background-size: 100%;
                margin: .26rem 0 0 .3rem;
            }
        }
    }

    @keyframes loading {
        0% {
            transform: rotate3d(0, 0, 1, 0deg);
        }

        100% {
            transform: rotate3d(0, 0, 1, 360deg);
        }
    }
</style>
