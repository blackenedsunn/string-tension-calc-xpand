<script setup lang="ts">
import {onMounted, ref, toRaw, watch} from 'vue';
import {Chart} from 'chart.js/auto';
import store from '@/model/store';
import * as c from '@/model/consts';

const canvas = ref<HTMLCanvasElement | null>(null);
let chart: Chart<'line', number[], string> | null = null;

onMounted(() => {
	chart = new Chart(canvas.value!, {
		type: 'line',
		data: {
			labels: [],
			datasets: [],
		},
		options: {
			scales: {
				y: {
					min: 0,
				},
			},
		},
	});
});

watch([store.guitars, store.unit], ([gtrs, unit], [prevGtrs, prevUnit]) => {
	const maxNumStrs = Math.max(...gtrs.map(gtr => gtr.strings.length));
	chart!.data.labels = c.STRING_NAMES.slice(0, maxNumStrs).reverse();

	chart!.data.datasets = gtrs.map((gtr, gtrIdx) => ({
		label: 'Guitar #' + (gtrIdx + 1),
		data: [
			...Array(maxNumStrs - gtr.strings.length).fill(NaN),
			...[...toRaw(gtr.strings)].reverse().map(str => str.tension),
		],
		tension: .1,
		pointRadius: 8,
		pointHoverRadius: 6,
	}));
	chart!.update('none');
}, {
	deep: true,
});
</script>

<template>
	<div :class="m.chart" v-show="store.guitars.value.length > 0">
		<canvas ref="canvas" />
	</div>
</template>

<style module="m" lang="scss">
	.chart {
		max-width: 100vw;
		max-height: 240px;
	}
</style>