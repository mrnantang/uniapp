<template>
	<view class="box">
		<!-- 切换按钮 -->
		<view class="btn">
			<button type="default" id="button0" :class="{ active1: 0 == button_index }" @click="switch_audio(0)">
				{{name1}}
			</button>
			<button type="default" id="button1" :class="{ active1: 1 == button_index }" @click="switch_audio(1)">
				{{name2}}
			</button>
		</view>
		<!-- 插件 -->
		<Song ref="Song" :video="video"/>
		<!-- <Video></Video> -->
		<!-- 底部切换按钮 -->
		<view class="foot">
			<view class="foot-item" v-for="(i,index) in item" :style="{fontSize:font_size}"
				:class="{active:index == isActive}" @click="change(index)">
				<view>{{i.name}}</view>
				<view v-if="index == 1" :class="{ active: index == isActive }">
					<text v-show="xiala" class="iconfont icon-xiala"></text>
					<text v-show="!xiala" class="iconfont icon-shang"></text>
				</view>
				<view v-if="index == 2" :class="{ active: index == isActive }">
					<text v-show="xiala2" class="iconfont icon-xiala"></text>
					<text v-show="!xiala2" class="iconfont icon-shang"></text>
				</view>
			</view>
		</view>
		<!-- 弹出栏 -->
		<view class="alert" v-show="showAlert">
			<view class="foot-item1" v-for="(i,index) in paragraph" :class="{ active: index == isclick }"
				@click="changeParagraph('Teach', index)">
				{{i.name}}
			</view>
		</view>
		<view class="alert2" v-show="showAlert2">
			<view class="foot-item1" v-for="(i,index) in paragraph1" :class="{ active: index == isclick }"
				@click="changeParagraph('Hechang', index)">
				{{i.name}}
			</view>
		</view>
	</view>
</template>


<script>
	import Song from "../../components/common/song.vue";
	import Video from "../../components/common/video.vue";

	export default {
		// 组件名称
		components: {
			Song,
			Video
		},
		data() {
			return {
				showAlert: false, //是否显示第二个按钮的子项
				showAlert2: false, //是否显示第三个按钮的子项
				xiala: true,
				xiala2: true,
				font_size: "4vw",
				isActive: 0,
				button_index: 0, //左上角切换按钮选择的序号
				isclick: -1,
				currentPage: 0,
				clickedIndex: "",
				paragraph: [],
				paragraph1: [],
				video: [],
				audio: [],
				name1: '',
				name2: '',
				item: [{
						id: "1",
						name: "听全曲",
						type: "Example",
						list: [],
					},
					{
						id: "2",
						name: "学领唱",
						type: "Teach",
						list: [],
					},
					{
						id: "3",
						name: "学合唱",
						type: "Hechang",
					},
				],
			}
		},
		onLoad() {
			this.getData()
		},
		methods: {
			switch_audio(index) {
				let mp3_url
				if (this.currentPage == 0) {
					mp3_url = this.video[0].mp3[index].url;
				} else if (this.currentPage == 1) {
					mp3_url = this.paragraph[this.clickedIndex].mp3[index].url;
				} else if (this.currentPage == 2) {
					mp3_url = this.paragraph1[this.clickedIndex].mp3[index].url;
				}
				this.$refs.Song.switch_audio(mp3_url);
				this.button_index = index;
			},
			change(index) {
				if (this.currentPage != index) {
					this.isclick = -1; //如果切换了选项卡，子项被选中的索引归-1
				}
				if (index == 1) {
					this.xiala = !this.xiala;
					this.showAlert = !this.showAlert;
					if (this.showAlert) {
						this.xiala2 = true;
						this.showAlert2 = false;
					}
				} else if (index == 2) {
					// this.showAlert = true;
					this.xiala2 = !this.xiala2;
					this.showAlert2 = !this.showAlert2;
					if (this.showAlert2) {
						this.xiala = true;
						this.showAlert = false;
					}
				} else {
					this.xiala2 = true;
					this.showAlert2 = false;
					this.xiala = true;
					this.showAlert = false;
					var mp3_url = this.video[0].mp3[0].url;
					this.name1 = this.video[0].mp3[0].name;
					this.name2 = this.video[0].mp3[1].name;
					if (this.$refs.hasOwnProperty("Song")) {
						this.$refs.Song.switch_audio(mp3_url);
						this.$refs.Song.switch_img(this.video[0].jump);
						// this.$refs.Song.plays();
					}
					this.currentPage = index;
					this.isActive = index;
				}
				this.button_index = 0;
			},
			changeParagraph(type, index) {
				this.isclick = index;
				this.clickedIndex = index;
				this.button_index = 0;
				if (type == "Teach") {
					var mp3_url = this.paragraph[index].mp3[0].url;
					this.name1 = this.paragraph[index].mp3[0].name;
					this.name2 = this.paragraph[index].mp3[1].name;
					this.$refs.Song.switch_audio(mp3_url);
					this.$refs.Song.switch_img(this.paragraph[index].jump);
					
					this.xiala2 = true;
					this.showAlert2 = false;
					this.xiala = true;
					this.showAlert = false;
					this.currentPage = 1;
					this.isActive = 1;
				} else if (type == "Hechang") {
					var mp3_url = this.paragraph1[index].mp3[0].url;
					this.name1 = this.paragraph1[index].mp3[0].name;
					this.name2 = this.paragraph1[index].mp3[1].name;

					this.$refs.Song.switch_audio(mp3_url);
					this.$refs.Song.switch_img(this.paragraph1[index].jump);

					this.xiala2 = true;
					this.showAlert2 = false;
					this.xiala = true;
					this.showAlert = false;
					this.currentPage = 2;
					this.isActive = 2;
				}
			},
			getData() {
				// 默认json读取文件
				let json_file = "rsdxh.json";
				// 根据链接后缀获取json文件
				let mapDataCollection = require('@/static/json/' + json_file);
				let res = JSON.parse(JSON.stringify(mapDataCollection));
				// 获取JSON里面的对象
				this.video = res.data.video;
				this.audio = res.data.audio;
				this.name1 = res.data.video[0].mp3[0].name;
				this.name2 = res.data.video[0].mp3[1].name;
				// 页面文档名字
				window.document.title = res.data.songName;
				// 获取学领唱子级段落数
				this.paragraph = res.data.audio;
				// 获取学合唱子级段落数
				this.paragraph1 = res.data.audio2;

			}

		}
	}
</script>

<style>
	@import url("../../static/css/iconfont.css");

	.box {
		position: relative;
	}

	.foot {
		width: 100%;
		height: 80px;
		background-color: black;
		display: flex;
		position: fixed;
		bottom: 0;
	}

	.foot-item {
		color: white;
		width: 50%;
		height: 100%;
		display: flex;
		align-items: center;
		text-align: center;
		justify-content: center;
		font-size: 4vw;
		position: relative;
		background-color: #000000;
	}

	.foot-item1 {
		color: white;
		width: 28vw;
		height: 8vh;
		display: flex;
		align-items: center;
		text-align: center;
		justify-content: center;
		font-size: 4vw;
		background-color: #000000;
	}

	.alert {
		position: absolute;
		top: 82%;
		left: 38%;
		width: 31vw;
		text-align: center;
		z-index: 999;
	}

	.alert2 {
		position: absolute;
		top: 91%;
		left: 70%;
		width: 31vw;
		text-align: center;
		z-index: 999;

	}

	.active {
		background-color: #2f2f2f;
		color: #feb301;
	}

	.active1 {
		background-color: #feb301;
		color: #ffffff;
	}

	.btn {
		position: absolute;
		opacity: 0.8;
		z-index: 9999;
	}
</style>
