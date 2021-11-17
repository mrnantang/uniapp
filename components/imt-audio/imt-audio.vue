<template>
    <view class="imt-audio">
        <view class="audio-wrapper">
            <view class="audio-number">{{ format(current) }}</view>
            <slider
                class="audio-slider"
                :activeColor="color"
                block-size="12"
                :min="0"
                :value="current"
                :max="duration"
                @changing="(seek = true), (current = $event.detail.value)"
                @change="audio.seek($event.detail.value)"
            >
            </slider>
            <view class="audio-number">{{ format(duration) }}</view>
        </view>
        <view class="audio-control-wrapper" :style="{ color }">
            <view
                class="audio-control audio-control-switch"
                :class="{ audioLoading: loading }"
                :style="{ borderColor: color }"
                @click="audio.paused ? play() : audio.pause()"
            >
                {{ loading ? "&#xe600;" : paused ? "&#xe865;" : "&#xe612;" }}
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            audio: uni.createInnerAudioContext(),
            current: 0, //当前进度(s)
            duration: 0, //总时长(s)
            paused: true, //是否处于暂停状态
            loading: false, //是否处于读取状态
            seek: false, //是否处于拖动状态,
            currentImgs: "0",
        };
    },
    props: {
        src: String, //音频链接
        autoplay: false, //是否自动播放
        continue: Boolean, //播放完成后是否继续播放下一首，需定义@next事件
        control: {
            type: Boolean,
            default: true,
        }, //是否需要上一曲/下一曲按钮
        color: {
            type: String,
            default: "#169af3",
        }, //主色调
    },

    methods: {
        close() {
            console.log("销毁");
            this.paused = true;
            this.audio.stop();
            this.audio.autoplay = false;
        },

        //格式化时长
        format(num) {
            return (
                "0".repeat(2 - String(Math.floor(num / 60)).length) +
                Math.floor(num / 60) +
                ":" +
                "0".repeat(2 - String(Math.floor(num % 60)).length) +
                Math.floor(num % 60)
            );
        },
        //点击播放按钮
        play() {
            console.log("点击了播放按钮");
            this.audio.play();
            this.loading = false;
        },
        plays() {
            console.log("点击了播放按钮");
            this.audio.play();
            this.loading = false;
            this.audio.pause();
        },
    },
    created() {
        if (this.src) {
            this.audio.src = this.src;
            this.autoplay && this.play();
        }
        this.audio.obeyMuteSwitch = false;

        //音频进度更新事件
        this.audio.onTimeUpdate(() => {
            if (!this.seek) {
                this.current = this.audio.currentTime;
                this.$emit("change2", this.audio.currentTime);
            }
        });
        this.audio.onCanplay(() => {
            console.log("on can play");
            this.duration = this.audio.duration;
            this.loading = false;
        });

        this.audio.onError((res) => {
            console.log(res.errMsg);
            console.log(res.errCode);
        });
        //音频播放事件
        this.audio.onPlay(() => {
            console.log("开始播放");
            this.paused = false;
            this.loading = false;
        });
        //音频暂停事件
        this.audio.onPause(() => {
            this.paused = true;
        });
        //音频结束事件
        this.audio.onEnded(() => {
            if (this.continue) {
                this.next();
            } else {
                this.paused = true;
                this.current = 0;
            }
        });
        //音频完成更改进度事件
        this.audio.onSeeked(() => {
            this.seek = false;
        });
    },
    beforeDestroy() {
        console.log("beforeDestroy");
        this.audio.destroy();
    },
    watch: {
        src(src, old) {
            console.log("play() 前watch");
            // this.close();
            this.audio.src = src;
            this.current = 0;
            this.duration = 0;
            if (old || this.autoplay) {
                this.play();
            }
        },
    },
};
</script>

<style>
@font-face {
    font-family: "icon";
    src: url("//at.alicdn.com/t/font_1104838_fxzimc34xw.eot");
    src: url("//at.alicdn.com/t/font_1104838_fxzimc34xw.eot?#iefix")
            format("embedded-opentype"),
        url("//at.alicdn.com/t/font_1104838_fxzimc34xw.woff2") format("woff2"),
        url("//at.alicdn.com/t/font_1104838_fxzimc34xw.woff") format("woff"),
        url("//at.alicdn.com/t/font_1104838_fxzimc34xw.ttf") format("truetype"),
        url("//at.alicdn.com/t/font_1104838_fxzimc34xw.svg#iconfont")
            format("svg");
}

.imt-audio {
    padding: 20upx 0;
    background: #fff;
    border-radius: 20upx;
}

.audio-wrapper {
    display: flex;
    align-items: center;
}

.audio-number {
    width: 120upx;
    font-size: 24upx;
    line-height: 1;
    color: #333;
    text-align: center;
}

.audio-slider {
    flex: 1;
    margin: 0;
}

.audio-control-wrapper {
    margin-top: 10upx;
    display: flex;
    justify-content: center;
    align-items: center;
    font-family: "icon" !important;
}

.audio-control {
    font-size: 32upx;
    line-height: 1;
    border: 4upx solid;
    border-radius: 50%;
    padding: 16upx;
}

.audio-control-next {
    transform: rotate(180deg);
}

.audio-control-switch {
    font-size: 40upx;
    margin: 0 100upx;
}

.audioLoading {
    animation: loading 2s;
    animation-iteration-count: infinite;
    animation-timing-function: linear;
}

@keyframes loading {
    to {
        transform: rotate(360deg);
    }
}
</style>
