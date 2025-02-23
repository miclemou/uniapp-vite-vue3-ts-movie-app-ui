<template>
	<view class="module-block">
		<MusicTitleComponent :classifyItem="classifyItem" />
		<view class="classify-music-list">
			<view class="classify-music-item" :key="classifyItem.id + '_' + item.id" v-for="item in classifyMusicList">
				<image class="music-cover" :src="getMusicCover(item.cover)" />
				<view class="song-info">
					<text class="song-name">{{item.songName}}</text>
					<text class="song-desc">{{`${item.authorName} - ${item.albumName}`}}</text>
				</view>
				<image v-if="store.isPlaying && store.musicItem.id === item.id" @click="store.usePlay(false)" class="icon-play" src="../../../static/icon_music_playing_grey.png"/>
				<image v-else @click="usePlayMusic(item)" class="icon-play" src="../../../static/icon_music_play.png"/>
				<image class="icon-play" :src="item.isLike ? isLikeActiveIcon : isLikeIcon"/>
				<image class="icon-play" src="../../../static/icon_music_menu.png" />
			</view>
		</view>
	</view>
</template>

<script setup lang="ts">
	import { defineProps, reactive, type PropType } from 'vue';
	import type { MusicClassifyType, MusicType } from "../types";
	import { getMusicListByClassifyIdService } from '../service';
	import { getMusicCover } from '../../utils/util';
	import MusicTitleComponent from './MusicTitleComponent';
	import { useStore } from "../../stores/useStore";
	import isLikeIcon from '../../../static/icon_like.png';
	import isLikeActiveIcon from '../../../static/icon_like_active.png';
	const store = useStore();

	const { classifyItem } = defineProps({
		classifyItem: {
			type: Object as PropType<MusicClassifyType>,
			reqiure: true,
			default: {}
		}
	});

	/**
	 * @description: 播放音乐
	 * @date: 2024-05-07 22:56
	 * @author wuwenqiang
	 */
	const usePlayMusic = (musicItem:MusicType) => {
		if(store.musicItem?.id === musicItem.id && store.musicList.length !== 0){
			store.usePlay(true)
		}else{
			store.setMusic(musicItem);
			if(store.musicClassify.id !== classifyItem.id || store.musicList.length === 0){
				getMusicListByClassifyIdService(classifyItem.id,1,500).then((res)=>{
					store.setMusicList(res.data);
				});
				store.setMusicClassify({...classifyItem});
			}
		}
		uni.navigateTo({
			url: `../pages/MusicPlayerPage`
		});
	}

	const classifyMusicList = reactive<Array<MusicType>>([]);

	getMusicListByClassifyIdService(classifyItem.id, 1, 3).then((res) => {
		classifyMusicList.push(...res.data);
	});


</script>

<style lang="less">
	@import '../../theme/color.less';
	@import '../../theme/size.less';
	@import '../../theme/style.less';

	.module-block-last {
		margin-bottom: @page-padding;
	}

	.classify-music-list {
		.classify-music-item {
			display: flex;
			margin-top: @page-padding;
			align-items: center;
			gap: @page-padding;

			.song-info {
				display: flex;
				flex-direction: column;
				flex: 1;
				width: 0;

				.song-name {
					font-weight: bold;
					overflow: hidden;
					text-overflow: ellipsis;
					white-space: nowrap;
					max-width: @max-text-width;
				}

				.song-desc {
					color: @disable-text-color;
					overflow: hidden;
					text-overflow: ellipsis;
					white-space: nowrap;
					max-width: @max-text-width;
				}
			}

			.icon-play {
				width: @small-icon-size;
				height: @small-icon-size;
			}
		}
	}
</style>
