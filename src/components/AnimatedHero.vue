<script setup lang="ts">
import AnimatedSpan from './AnimatedSpan.vue';
import { ref } from 'vue';
import * as feather from 'feather-icons';

const props = defineProps<{
	text: string,
	rows: number,
}>();

const completedRowAnimations = ref<number>(0);

function decoratorRowsVisible(): boolean
{
	return completedRowAnimations.value < props.rows * 2;
}
</script>

<template>
	<div class="hero" ref="observerTarget">
		<template v-if="decoratorRowsVisible()">
			<div
				v-for="index in props.rows"
				:aria-hidden="true"
				:style="{
					position: 'absolute',
					transform: `translateY(-${index * 100}%)`
				}"
			>
				<AnimatedSpan
					:text="props.text" :key="index" :stay="false" :initial-delay="1500" animate="immediate"
					@animation-complete="completedRowAnimations++"
				/>
			</div>
		</template>
		<div :class="{ 'hero__text': true, 'hero__text--animate': !decoratorRowsVisible() }">
			<AnimatedSpan :text="props.text" :initial-delay="1500" animate="immediate" />
		</div>
		<template v-if="decoratorRowsVisible()">
			<div
				v-for="index in props.rows"
				:aria-hidden="true"
				:style="{
					position: 'absolute',
					transform: `translateY(${index * 100}%)`
				}"
			>
				<AnimatedSpan
					:text="props.text" :key="index" :stay="false" :initial-delay="1500" animate="immediate"
					@animation-complete="completedRowAnimations++"
				/>
			</div>
		</template>
		<span :class="{'chevron': true, 'chevron--show': !decoratorRowsVisible() }" v-html="feather.icons['chevron-down'].toSvg({ height: 36, width: 36 })"/>
	</div>
</template>

<style scoped>
.hero {
	position:         relative;

	width:            100%;
	height:           100svh;

	display:          flex;
	flex-direction:   column;
	align-items:      center;
	justify-content:  center;

	overflow:         hidden;

	background-color: #111111;
}

.hero * {
	text-transform: uppercase;
	text-align:     center;

	font-size:      clamp(2.5rem, 5vw + 1rem, 5rem);
	font-weight:    800;

	user-select:    none;

	color:          white;
}

.hero__text--animate {
	animation: anim-hero-text 250ms 250ms forwards;
}

.chevron {
	position: absolute;

	opacity: 0;

	bottom: 2%;

	animation: anim-chevron-bob 600ms;
	animation-timing-function: ease-out;
	animation-direction: alternate;
	animation-iteration-count: infinite;

	transition: opacity 300ms;
}

.chevron--show {
	opacity: 1;
}

@keyframes anim-hero-text {
	from {
		padding: inherit;
		border:  none;
	}
	to {
		padding:      1rem 2.5rem;
		border-style: solid;
		border-color: white;
		border-width: 0.125rem;
	}
}

@keyframes anim-chevron-bob {
	from {
		bottom: 2%;
	}
	to {
		bottom: 1%
	}
}

@media (prefers-reduced-motion) {
	.chevron {
		opacity: 1;
		animation: none;
	}
}
</style>