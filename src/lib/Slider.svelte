<script>
	import { onMount } from "svelte";
  import { createEventDispatcher } from 'svelte';
  
  const dispatch = createEventDispatcher();
	
	export let min = 0;
	export let max = 100;
	export let step = 1;
	export let value = 50;
	export let width = null;
  export let name = null;
  export let color = "teal";
	
	let range, thumb, track;

	const updateSlider = (value) => {
		if (thumb) {
			let percent = ((value - min) / (max - min)) * 100;
			thumb.style.left = `${percent}%`;
			thumb.style.transform = `translate(-${percent}%, -50%)`;
			track.style.width = `${percent}%`;
		}
	}

  const event = (type, e) => dispatch(type, {e});

	onMount(() => {
		range.oninput = (e) => updateSlider(e.target.value);
		updateSlider(value) // Init value
	})
	
	$: updateSlider(value);
</script>

<div class="wrap" style:width={typeof width == 'number' ? `${width}px` : width} style:--color={color}>
  <input
    type="range" class="range"
    {name} {min} {max} {step}
    bind:value bind:this={range}
    on:input={e => event('input', e)}
    on:select={e => event('select', e)}>
  <div class="track">
    <div class="track-inner" bind:this={track}/>
  </div>
  <div class="thumb" bind:this={thumb}/>
</div>

<style>
	.wrap {
    width: 100%;
		position: relative;
	}
	.range {
    width: 100%;
		cursor: pointer;
		opacity: 0;
	}
	.range::-ms-tooltip {
		display: none;
	}
	.track {
		width: 100%;
		height: 4px;
		background-color: #ccc;
		position: absolute;
		top: calc(50% - 4px);
		transform: translateY(-50%);
		pointer-events: none;
	}
  .wrap:hover .track {
    background-color: #aaa;
  }
	.track-inner {
		width: 0;
		height: 100%;
		background: var(--color, black);
	}
	.thumb {
		width: 16px;
		height: 16px;
		background: var(--color, black);
		border-radius: 50%;
		position: absolute;
		top: calc(50% - 4px);
		left: 0;
		transform: translate(0%, -50%);
		pointer-events: none;
	}
</style>