<template>
	<view class="box">
		<!-- 轮播图 -->
		<swiper :indicator-dots="true" circular class="image-content">
			<swiper-item class="image-content" v-for="(i,index) in imgArry" :key="index">
				<image class="ImageSize" :src="i" @click="ClickImg(i)"></image>
			</swiper-item>
		</swiper>
		<!-- 音频 -->
		<view class="audio_box">
			<view class="audios">
				<imtAudio ref="imtAudio" :src="mp3_url" :duration="mp3_url.duration" ></imtAudio>
			</view>
		</view>
	</view>
</template>

<script>
	// imtAudio 为音频UI插件
	import imtAudio from "components/imt-audio/imt-audio.vue";
	export default {
		components: {
			imtAudio
		},
		props: {
			video: Array,
		},
		data() {
			return {
				imgArry: [],
				jump: [],
				mp3_url: '',
				allKeys:[],
				imgIndexArray: []
			}
		},
		mounted() {
			this.imgArry = this.video[0].jump;
			this.getAllkeys(this.imgArry);
			if (this.video[0].hasOwnProperty("mp3")) {
				this.mp3_url = this.video[0].mp3[0].url;
			}
			let index = 0;
			for (let i in this.imgArry) {
				this.imgIndexArray[i] = index;
				index++;
			}
		},
		methods: {
			plays() {
			            this.$refs.imtAudio.play();
			        },
			switch_audio(mp3_url) {
				this.$refs.imtAudio.close();
				this.mp3_url = mp3_url;
			},
			switch_img(jump) {
				this.imgArry = jump;
			},
			// 点击图片放大p
			ClickImg(imgUrl) {
				let images = [];
				images.push(imgUrl);
				uni.previewImage({
					urls: images,
					current: 0
				})
			},
			getAllkeys(jump) {
				for (var key in jump) {
					this.allKeys.push(key);
				}
			},
		}
	}
</script>

<style>
	.box {
		width: 100vw;
		height: 88vh;
		background-color: #f5f5f5;
	}

	.image-content {
		height: 74vh;
		overflow-y: scroll;
		text-align: center;
	}

	.ImageSize {
		width: 100%;
		height: 110%;
	}

	.audio_box {
		width: 100%;
		height: 14vh;
	}
</style>
