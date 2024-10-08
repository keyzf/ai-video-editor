<template>
	<!-- 图层顶部 -->
	<div class="timeline layer-top" ref="topRef"> </div>
	<!-- 图层 -->
	<div class="timeline layer container" ref="containerRef" :style="{'height': `${modelValue.height}px`}">
		<layer-unit v-for="(item,index) in modelValue.units" :key="item.id" :data="item"
			@on-drag="emits('onDrag', $event)">
			<div class="view" v-html="item.view"></div>
		</layer-unit>
	</div>
	<!-- 图层间隙，插入元素 -->
	<div class="timeline layer-gap" ref="layerGapRef" :class="{'last':last}">
		<div v-if="last">插入新的时间线</div>
	</div>
</template>

<script setup>
	import LayerUnit from './unit.vue'
	import {
		ref,
		watch,
		computed,
		onMounted,
		onBeforeUnmount
	} from 'vue'
	import {
		ElNotification
	} from 'element-plus'
	import {
		useLayersDataStore
	} from '../../../store/layers.js'

	const layersDataStore = useLayersDataStore()
	const emits = defineEmits(['onDrag', 'onDrop'])
	const props = defineProps({
		modelValue: Object,
		dropData: Object,
		last: Boolean
	})
	const topRef = ref()
	const containerRef = ref()
	const layerGapRef = ref()

	function handleContainerMouseenter(event) {
		if (props.dropData && props.dropData.track.dragging) {
			// 确保图层类型相同
			if (props.modelValue.type == props.dropData.type) {
				containerRef.value.classList.add('graggle')
			} else {
				containerRef.value.classList.add('disabled')
			}
		} else {
			containerRef.value.classList.remove('graggle')
			containerRef.value.classList.remove('disabled')
		}
	}

	function handleContainerMouseup(event) {
		// 确保图层类型相同
		if (props.modelValue.type == props.dropData.type) {
			if (containerRef.value.classList.contains('graggle')) {
				emits('onDrop', {
					dropData: props.dropData,
					dropMode: 'appendUnit'
				})
			}
		} else if (props.dropData.track.dragging) {
			ElNotification({
				title: '提示',
				message: '该图层与当前元素类型不匹配。',
				type: 'warning',
			})
		}
		containerRef.value.classList.remove('graggle')
		containerRef.value.classList.remove('disabled')
	}

	function handleContainerMouseleave(event) {
		containerRef.value.classList.remove('graggle')
		containerRef.value.classList.remove('disabled')
	}

	function handleLayerGapMouseenter(event) {
		if (props.dropData && props.dropData.track.dragging) {
			layerGapRef.value.classList.add('graggle')
		} else {
			layerGapRef.value.classList.remove('graggle')
		}
	}

	function handleLayerGapMouseup(event) {
		if (layerGapRef.value.classList.contains('graggle')) {
			emits('onDrop', {
				dropData: props.dropData,
				dropMode: 'newLayer'
			})
		}
		layerGapRef.value.classList.remove('graggle')
	}

	function handleLayerGapMouseleave(event) {
		layerGapRef.value.classList.remove('graggle')
	}

	function handleTopMouseenter(event) {
		if (props.dropData && props.dropData.track.dragging) {
			topRef.value.classList.add('graggle')
		} else {
			topRef.value.classList.remove('graggle')
		}
	}

	function handleTopMouseup(event) {
		if (topRef.value.classList.contains('graggle')) {
			emits('onDrop', {
				dropData: props.dropData,
				dropMode: 'topLayer'
			})
		}
		topRef.value.classList.remove('graggle')
	}

	function handleTopMouseleave(event) {
		topRef.value.classList.remove('graggle')
	}

	onMounted(() => {
		props.modelValue.instance = containerRef.value
		// 拖拽至追加位置
		containerRef.value.addEventListener('mouseenter', handleContainerMouseenter);
		containerRef.value.addEventListener('mouseup', handleContainerMouseup);
		containerRef.value.addEventListener('mouseleave', handleContainerMouseleave);
		// 拖拽至新增位置
		layerGapRef.value.addEventListener('mouseenter', handleLayerGapMouseenter);
		layerGapRef.value.addEventListener('mouseup', handleLayerGapMouseup);
		layerGapRef.value.addEventListener('mouseleave', handleLayerGapMouseleave);
		// 拖拽至顶部位置
		topRef.value.addEventListener('mouseenter', handleTopMouseenter);
		topRef.value.addEventListener('mouseup', handleTopMouseup);
		topRef.value.addEventListener('mouseleave', handleTopMouseleave);
	})

	onBeforeUnmount(() => {
		containerRef.value.removeEventListener('mouseenter', handleContainerMouseenter);
		containerRef.value.removeEventListener('mouseup', handleContainerMouseup);
		containerRef.value.removeEventListener('mouseleave', handleContainerMouseleave);
		layerGapRef.value.removeEventListener('mouseenter', handleLayerGapMouseenter);
		layerGapRef.value.removeEventListener('mouseup', handleLayerGapMouseup);
		layerGapRef.value.removeEventListener('mouseleave', handleLayerGapMouseleave);
		topRef.value.removeEventListener('mouseenter', handleTopMouseenter);
		topRef.value.removeEventListener('mouseup', handleTopMouseup);
		topRef.value.removeEventListener('mouseleave', handleTopMouseleave);
	})
</script>

<style scoped>
	.timeline.layer {
		position: relative;
		background-color: var(--layer-bg);
		box-sizing: border-box;
	}

	.timeline.layer.graggle {
		background-color: var(--layer-graggle-bg);
	}

	.timeline.layer.disabled {
		border-top: 1px solid var(--el-color-danger-light-5);
		border-bottom: 1px solid var(--el-color-danger-light-5);
		background-image: repeating-linear-gradient(40deg, transparent, transparent 20px, var(--el-color-danger-light-8) 10px, var(--el-color-danger-light-8) 40px);
	}

	.timeline.layer-gap {
		height: 6px;
		margin: 3px 0;
	}

	.timeline.layer-gap.graggle {
		background-color: var(--el-color-warning);
		opacity: 0.6;
	}

	.timeline.layer-gap.last {
		height: 40px;
		line-height: 40px;
		margin: 0;
	}

	.timeline.layer-top:first-child {
		height: 6px;
		margin: 3px 0;
	}

	.timeline.layer-top.graggle:first-child {
		background-color: var(--el-color-warning);
		opacity: 0.6;
	}

	.timeline.layer-gap.last div {
		display: none;
		margin-left: 20px;
	}

	.timeline.layer-gap.last.graggle div {
		display: initial;
	}

	.container {
		display: flex;
		gap: 10px;
		align-items: center;
	}

	.layer-unit {
		display: flex;
		align-items: center;
	}

	.layer-unit .view {
		font-size: 14px;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		width: 100%;
		height: 100%;
		min-width: 100px;
	}
</style>

<style>
	/* 资源预览样式 */

	.source-view-image {
		width: 100%;
		height: 100%;
		background-repeat: repeat-x;
		background-size: contain;
		background-position: left;
	}

	.source-view-name {
		position: absolute;
		max-width: 100%;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		font-size: 10px;
		top: 0;
		padding-left: 2px;
	}
 
</style>