<script setup lang="ts">
import { onMounted, ref } from 'vue';

// Defines the component props.
interface Props
{
	// The text to display and animate e.g. 'James Minor'.
	text: string,
	// Should this span stay on screen once it's pop-in animation completes?
	stay?: boolean,
	// Initial delay (in ms) appended to every letter's pop-in animation.
	initialDelay?: number,
	// The time (in ms) that the span should be visible. Note that this only works if the stay prop is set to true.
	onScreenTime?: number,
	// Should the span when it is visible, immediately, or not at all?
	animate?: 'visible' | 'immediate' | 'false',
}

// Defines the default values for the optional props.
const props = withDefaults(defineProps<Props>(),
	{
		stay: true,
		initialDelay: 500,
		onScreenTime: 1250,
		animate: 'visible',
	});

// Defines the events available to the component.
const emits = defineEmits<{
	// Event fired when the pop-in animation completes.
	(e: 'pop-in-complete'): void,
	// Event fired when the pop-out animation completes.
	(e: 'pop-out-complete'): void,
	// Event fired when all animations are done. For spans that stay, at the end of pop-in, otherwise end of pop-out.
	(e: 'animation-complete'): void,
}>();

// How many animations has the row completed?
const animationCount = ref<number>(0);

// Ref connected to the container element for the span.
const containerRef = ref();

// Array to hold the data for each letter within the span.
const letterData = ref<{
	// The specified letter of the text string, e.g. 'J'.
	letter: string,
	// The animation delay in milliseconds.
	delay: number,
	// Does this letter's span have the pop-in CSS animation class?
	popInClass: boolean,
	// Does this letter's span have the pop-out CSS animation class?
	popOutClass: boolean,
}[]>([]);

// Generates the letter delays for each letter in the text prop.
function generateLetterAnimationDelays()
{
	const letters = props.text.split('');
	const delaySeparation: number = 50;

	// Generating the delay pool set.
	const delaySet: Set<number> = new Set();
	while (delaySet.size < letters.length)
	{
		const delayMultiplier = Math.floor(Math.random() * letters.length) + 1;
		delaySet.add(delaySeparation * delayMultiplier);
	}

	const delayPool: number[] = Array.from(delaySet);

	// Generating the data array.
	for (let i = 0; i < letters.length; i++)
	{
		letterData.value.push({
			letter: letters[i],
			delay: delayPool[i],
			popInClass: false,
			popOutClass: false,
		});
	}
}

// Starts the span animation.
function startAnimation()
{
	letterData.value.forEach((entry) =>
	{
		// Pop-in timeout.
		setTimeout(() =>
		{
			entry.popInClass = true;
		}, entry.delay + props.initialDelay);

		// Pop-out timeout.
		if (!props.stay)
		{
			setTimeout(() =>
			{
				entry.popOutClass = true;
			}, entry.delay + props.initialDelay + props.onScreenTime);
		}
	});
}

// Handling DOM mounting.
onMounted(() =>
{
	generateLetterAnimationDelays();

	switch (props.animate)
	{
		case 'visible':
			const observer = new IntersectionObserver((entries) =>
			{
				entries.forEach((entry) =>
				{
					if (animationCount.value == 0 && entry.isIntersecting)
					{
						startAnimation();
						observer.disconnect();
					}
				});
			}, { threshold: 1.0 });
			observer.observe(containerRef.value);
			break;
		case 'immediate':
			startAnimation();
			break;
		case 'false':
			break;
	}
});

// Handles the firing of any events for conveying the current animation state.
function handleLetterAnimationEnd()
{
	animationCount.value++;

	// How many total animations the row should complete, in order for animation 'completion' to be determined.
	const animationIterationTargets = {
		// Target for the pop-in animations.
		popIn: props.text.length,
		// Target for the pop-out animations.
		popOut: props.text.length * 2,
		// Target for the entire animation sequence to be completed.
		complete: props.stay ? props.text.length : props.text.length * 2,
	};

	// Checking and firing any animation events.
	if (animationCount.value >= animationIterationTargets.popIn)
	{
		emits('pop-in-complete');
	}
	if (animationCount.value >= animationIterationTargets.popOut)
	{
		emits('pop-out-complete');
	}
	if (animationCount.value >= animationIterationTargets.complete)
	{
		emits('animation-complete');
	}
}
</script>

<template>
	<span class="animated-span" ref="containerRef">
		<span
			v-for="(data, index) in letterData"
			:key="index"

			:class="{ 'pop-in': data.popInClass, 'pop-out': data.popOutClass, }"

			@animationend="handleLetterAnimationEnd"
		>
			{{ data.letter }}
		</span>
	</span>
</template>

<style scoped>
.animated-span > span {
	opacity: 0;
}

.pop-in {
	animation: anim-pop-in 30ms forwards;
}

.pop-out {
	animation: anim-pop-out 30ms forwards;
}

@keyframes anim-pop-in {
	from {
		opacity: 0;
	}
	to {
		opacity: 1;
	}
}

@keyframes anim-pop-out {
	from {
		opacity: 1;
	}
	to {
		opacity:    0;
		visibility: hidden;
	}
}

@media (prefers-reduced-motion) {
	.animated-span > span {
		opacity:   0;
		animation: none;
	}
}
</style>