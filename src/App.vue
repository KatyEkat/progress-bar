<template>
	<div class="app">

		<div class="buttons">
			<button
				@click="runTo(100, 'success')"
				:class="[
					'btn',
					{
						active: activeButton === 'success',
						success: activeButton === 'success',
					},
				]"
			>
				success
			</button>

			<button
				@click="runTo(70, 'warning')"
				:class="[
					'btn',
					{
						active: activeButton === 'warning',
						warning: activeButton === 'warning',
					},
				]"
			>
				warning
			</button>

			<button
				@click="runTo(65, 'error')"
				:class="[
					'btn',
					{ active: activeButton === 'error', error: activeButton === 'error' },
				]"
			>
				error
			</button>
		</div>

		<div class="progress-wrapper">
			<ProgressCircle :value="progress" :status="status" @done="handleDone" />
		</div>
	</div>
</template>

<script setup>
import { ref, nextTick } from "vue";
import ProgressCircle from "./components/ProgressCircle.vue";

const status = ref("in-progress");
const progress = ref(0);
const target = ref(100);
const finalStatus = ref("success");
const activeButton = ref(null);

async function runTo(newTarget, newStatus) {
	activeButton.value = newStatus;

	target.value = newTarget;
	finalStatus.value = newStatus;

	status.value = "in-progress";
	progress.value = 0;
	await nextTick();

	progress.value = target.value;
}

function handleDone(val) {
	if (val === target.value) {
		status.value = finalStatus.value;
	}
}
</script>

<style scoped>
.app {
	padding: 40px;
	max-width: 600px;
	margin: 0 auto;
	font-family: sans-serif;
	text-align: center;
}

.buttons {
	display: flex;
	gap: 10px;
	justify-content: center;
	margin-bottom: 30px;
}

.btn {
	padding: 8px 16px;
	border-radius: 8px;
	cursor: pointer;
	border: 1px solid #679ed2;
	background: #679ed2;
	color: #fff;
	font-size: 16px;
	transition: background 0.15s, color 0.15s, transform 0.1s;
}

.btn:active {
	transform: scale(0.97);
}

.btn.active {
	border: none;
}

.btn.success.active {
	background: #22c55e;
}

.btn.warning.active {
	background: #eab308;
}

.btn.error.active {
	background: #ef4444;
}

.progress-wrapper {
	display: flex;
	justify-content: center;
}
</style>
