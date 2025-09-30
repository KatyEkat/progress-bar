<template>
	<div class="progress-container" :class="[status, { dashboard }]">
		<svg
			class="progress-svg"
			:width="size"
			:height="size"
			:viewBox="`0 0 ${size} ${size}`"
		>
			<circle
				class="progress-bg"
				:cx="center"
				:cy="center"
				:r="radius"
				fill="none"
				:stroke-width="strokeWidth"
			/>

			<circle
				class="progress-bar"
				:cx="center"
				:cy="center"
				:r="radius"
				fill="none"
				:stroke="currentColor"
				:stroke-dasharray="circumference"
				:stroke-dashoffset="dashOffset"
				stroke-linecap="round"
				:stroke-width="strokeWidth"
			/>
		</svg>

		<div class="progress-text">
			<span v-if="status === 'in-progress'">{{ displayValue }}%</span>
			<span v-else-if="status === 'success'">✔️</span>
			<span v-else-if="status === 'warning'">❕</span>
			<span v-else-if="status === 'error'">✖️</span>
		</div>
	</div>
</template>

<script setup>
import { ref, computed, watch } from "vue";

const props = defineProps({
	value: { type: Number, default: 0 },
	size: { type: Number, default: 150 },
	strokeWidth: { type: Number, default: 10 },
	status: { type: String, default: "in-progress" },
	dashboard: { type: Boolean, default: false },
});

const emit = defineEmits(["done"]);

const animatedValue = ref(0);

const radius = computed(() => (props.size - props.strokeWidth) / 2);
const center = computed(() => props.size / 2);
const circumference = computed(() => 2 * Math.PI * radius.value);
const dashOffset = computed(
	() => circumference.value - (animatedValue.value / 100) * circumference.value
);

let rafId = null;
function stopAnimation() {
	if (rafId != null) {
		cancelAnimationFrame(rafId);
		rafId = null;
	}
}

function easeInOut(t) {
	return t < 0.5 ? 2 * t * t : 1 - Math.pow(-2 * t + 2, 2) / 2;
}

function animateValue(from, to, duration = 700) {
	stopAnimation();
	if (from === to) {
		animatedValue.value = to;
		emit("done", to);
		return;
	}
	const start = performance.now();

	const tick = (time) => {
		const p = Math.min((time - start) / duration, 1);
		const e = easeInOut(p);
		animatedValue.value = from + (to - from) * e;
		if (p < 1) {
			rafId = requestAnimationFrame(tick);
		} else {
			rafId = null;
			animatedValue.value = to;
			emit("done", to);
		}
	};

	rafId = requestAnimationFrame(tick);
}

watch(
	() => props.value,
	(newVal) => {
		if (newVal === 0) {
			stopAnimation();
			animatedValue.value = 0;
			return;
		}
		animateValue(animatedValue.value, newVal);
	},
	{ immediate: true }
);

const currentColor = computed(() => {
	if (props.status !== "in-progress") {
		return {
			success: "#22c55e",
			warning: "#eab308",
			error: "#ef4444",
		}[props.status];
	}
	const r = Math.round(255 - (animatedValue.value * 255) / 100);
	const g = Math.round((animatedValue.value * 255) / 100);
	return `rgb(${r},${g},0)`;
});

const displayValue = computed(() => Math.round(animatedValue.value));
</script>

<style scoped>
.progress-container {
	position: relative;
	display: inline-flex;
	justify-content: center;
	align-items: center;
}

.progress-svg {
	transform: rotate(-90deg);
}

.progress-bg {
	stroke: #f2f3f7;
	stroke-width: 10;
}

.progress-bar {
	transition: stroke 0.5s;
	stroke-width: 10;
	transform-origin: center;
}

.progress-text {
	position: absolute;
	font-size: 1.2rem;
	font-weight: 600;
	color: #4b5563;
	display: flex;
	align-items: center;
	justify-content: center;
}

.success .progress-bar {
	stroke: #22c55e;
}

.warning .progress-bar {
	stroke: #eab308;
}

.error .progress-bar {
	stroke: #ef4444;
}

.dashboard .progress-svg {
	transform: rotate(180deg);
}
.dashboard .progress-container {
	clip-path: inset(50% 0 0 0);
}
</style>
