<template>
	<div class="timeline-layers" ref="layersRef" @contextmenu="onContextMenu">
		<layer-item v-for="(item,index) in layers" :key="item.id" v-model="layers[index]" :last="layers.length==index+1"
			:drop-data="dragData" @on-drag="onDrag($event,index)" @on-drop="onDrop($event,index)"></layer-item>
	</div>
	<adsorption-line-hint-vue></adsorption-line-hint-vue>
	<select-unit-vue></select-unit-vue>
</template>

<script setup>
	import adsorptionLineHintVue from './components/adsorption-line-hint.vue'
	import selectUnitVue from './components/select-unit.vue'
	import ContextMenu from '@imengyu/vue3-context-menu'
	import LayerItem from './item.vue'
	import {
		ref,
		onMounted,
		watch,
		nextTick
	} from 'vue'
	import {
		v4 as uuidv4
	} from 'uuid'
	import Layer from '../../../bean/Layer.js'
	import {
		useTrackStore
	} from '../../../store/track.js'
	import {
		useViewportStore
	} from '../../../store/viewport.js'
	import {
		useLayersDataStore,
		useSelectUnitStore
	} from '../../../store/layers.js'


	const layersDataStore = useLayersDataStore()
	const selectUnitStore = useSelectUnitStore()
	const trackStore = useTrackStore()
	const viewportStore = useViewportStore()
	const emits = defineEmits(['onDrag'])
	const props = defineProps({
		modelValue: {
			type: Array
		}
	})
	const layers = ref(props.modelValue)
	const layersRef = ref()
	const dragData = ref(null)

	const onContextMenu = (e) => {
		e.preventDefault();
		const unit = layersDataStore.getUnitUnderMouse(e)
		if (unit) {
			const dark = document.querySelector('html').classList.contains('dark')
			ContextMenu.showContextMenu({
				theme: 'mac' + (dark ? ' dark' : ''),
				x: e.x,
				y: e.y,
				items: [{
					label: "对齐音频",
					hidden: unit.type != 'image',
					svgIcon: '#fa-crosshairs',
					svgProps: {
						fill: dark ? '#aaa' : '#666',
					},
					onClick: () => {
						selectUnitStore.selection((selected) => {
							unit.track.x = selected.track.x;
							unit.track.w = selected.track.w;
							layersDataStore.sortLayers();
						})
						selectUnitStore.cautionUnit('audio')
					}
				}, {
					label: "显示",
					hidden: !unit.visible || unit.display,
					svgIcon: '#fa-eye',
					svgProps: {
						fill: dark ? '#aaa' : '#666',
					},
					onClick: () => unit.display = true
				}, {
					label: "隐藏",
					hidden: !unit.visible || !unit.display,
					svgIcon: '#fa-eye-slash',
					svgProps: {
						fill: dark ? '#aaa' : '#666',
					},
					onClick: () => unit.display = false
				}, {
					label: "声音",
					hidden: !unit.audible || !unit.muted,
					svgIcon: '#fa-volume-low',
					svgProps: {
						fill: dark ? '#aaa' : '#666',
					},
					onClick: () => unit.muted = false
				}, {
					label: "静音",
					hidden: !unit.audible || unit.muted,
					svgIcon: '#fa-volume-xmark',
					svgProps: {
						fill: dark ? '#aaa' : '#666',
					},
					onClick: () => unit.muted = true
				}, {
					divided: 'up',
					label: "删除",
					svgIcon: '#fa-trash-can',
					svgProps: {
						fill: dark ? '#aaa' : '#666',
					},
					onClick: () => {
						const layer = layersDataStore.getLayerUnderMouse(e)
						layer.remove(unit.id)
						layersDataStore.clearEmptyLayer()
					}
				}]
			});
		}
	}

	const onDrag = (event, index) => {
		dragData.value = event
		// 告知上级有元素移动，更新时间轴长度
		emits('onDrag', event)
		// 拖拽结束后(不选择拖拽中进行节省性能,所以拖拽中可以重合,用户体验良好)进行一个x坐标的排序，并且如果有重合调整坐标，保证友好的顺序以及不重合。
		if (!event.track.dragging) {
			// 走线程，onDrag 比 onDrop 触发早，防止优先排序后在更换时间线
			setTimeout(() => layersDataStore.sortLayers())
		}
		viewportStore.playing = false
	}
	const onDrop = (event, newIndex) => {
		// 找出当前位置
		let layerIndex, unitIndex;
		for (let i = 0; i < layers.value.length; i++) {
			const layer = layers.value[i];
			for (let j = 0; j < layer.length; j++) {
				if (layer.units[j].id == event.dropData.id) {
					layerIndex = i;
					unitIndex = j;
				}
			}
		}
		// 删除当前元素位置但不销毁，删除后加入到新的位置
		layers.value[layerIndex].units.splice(unitIndex, 1)
		// 元素放置到新图层
		if (event.dropMode == 'newLayer')
			layersDataStore.insertLayer(newIndex + 1, Layer.list(event.dropData))
		// 元素追加到现有图层
		if (event.dropMode == 'appendUnit')
			layersDataStore.appendUnit(newIndex, event.dropData)
		// 元素添加顶部新图层
		if (event.dropMode == 'topLayer')
			layersDataStore.insertLayer(newIndex, Layer.list(event.dropData))
		// 拖拽结束后(不选择拖拽中进行节省性能,所以拖拽中可以重合,用户体验良好)进行一个x坐标的排序，并且如果有重合调整坐标，保证友好的顺序以及不重合。
		layersDataStore.sortLayers()
	}
</script>
<style scoped>
	.timeline-layers {
		flex: 1 1 0%;
		overflow-y: auto;
	}

	.timeline-layers::-webkit-scrollbar {
		width: 0px;
	}
</style>