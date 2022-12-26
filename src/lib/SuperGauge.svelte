<script>
	import { tweened, spring } from 'svelte/motion';
	import { backInOut } from 'svelte/easing';
	import { arc as d3arc } from 'd3-shape';
	import { scaleLinear } from 'd3-scale';
    import { text } from 'svelte/internal';

	export let min = 0
	export let max = 1000
	export let value = 350

	export let showNeedle = false
	export let showValue = true
	export let color = "blue"
	export let endColor = "lime"

	export let showTicks = false
  	export let showTickLabels = false
  	export let majorTicks = 20
  	export let minorTicks = 0
	 
	let startAngle = -90;
	let endAngle = 0;
	let innerRadius = 242;
	let outerRadius = 252;
	let cornerRadius = 10;

	// An array with the angle and label of each tick
	let _majorTicks = []

    // $: _value = spring(50, { stiffness: .1 });
	$: _value.set(value);
    $: _value = tweened(50, { easing: backInOut, duration: 750 });
	$: scale = scaleLinear()
		.domain([min, max])
		.range([startAngle, endAngle]);
	

	function generateTicks ( majorTicks, minorTicks ) {
		let _majorStep = ( max - min ) / majorTicks;
		let _minorStep = _majorStep / minorTicks;
		let _pos = Number(min) || 0;
		let _tick = {};

		_majorTicks = [];
		while ( _pos <= max ) {
			_majorTicks.push({ angle:scale(Number(_pos)) , label: Math.round(_pos) });
			_pos += _majorStep;
		}
		console.log(_majorTicks)
	}

	$: generateTicks( majorTicks , minorTicks )
	
	$: valueAngle = scale($_value);

	$: arc = d3arc()
		.innerRadius(innerRadius)
		.outerRadius(outerRadius)
		.startAngle(startAngle * Math.PI / 180)
		.endAngle(valueAngle * Math.PI / 180)
		.cornerRadius(cornerRadius);
	
	$: trackArc = d3arc()
		.innerRadius(innerRadius)
		.outerRadius(outerRadius)
		.startAngle(startAngle * Math.PI / 180)
		.endAngle(endAngle * Math.PI / 180)
		.cornerRadius(cornerRadius);

	$: innerArc = d3arc()
		.innerRadius(0)
		.outerRadius(102)
		.startAngle(startAngle * Math.PI / 180)
		.endAngle(endAngle * Math.PI / 180)
		.cornerRadius(8);
	
	let trackArcEl
	$: boundingBox = trackArc && trackArcEl ? trackArcEl.getBBox() : {};

</script>

<div class="svg-box">

<svg class="svg-box-content" viewbox="0, 0, 266, 266" >
	<path d={trackArc()} transform="translate(254, 254)" class="track" bind:this={trackArcEl} />
	<path d={arc()} transform="translate(254, 254)" />
	<path d={innerArc()} class="innerArc" transform="translate(254, 254)" />

	{#if _majorTicks.length > 0}
		{#each _majorTicks as tick}
			<line class="tick" x1=254 y1=12 x2=254 y2=22 transform="rotate({Number(tick.angle)} 254 254)"/>
			<text class="tick-label" x=254 y=32 transform="rotate({tick.angle} 254 254)"> {tick.label} </text>
		{/each}
	{/if}

	{#if showValue}
		<text transform="translate(210,238)">
			{Math.round($_value)}
		</text>
	{/if} 

	{#if showNeedle}
		<polygon class="needle" points="254,20 256,150 252,150" transform="rotate({valueAngle} 254 254)"/>
	{/if}



	<defs>
    <linearGradient id="fillGradient" x1="0%" y1="0%" x2="0%" y2="100%">
      <stop offset="0%"   stop-color="{color}" />
      <stop offset="100%" stop-color="{endColor}" />
    </linearGradient>
  </defs>
</svg>
</div>


<style>
	.svg-box {
		width: 100%;
		min-width: 256px;
		position: relative;
	}

	.needle {
		stroke: var(--spectrum-global-color-gray-700);
		stroke-width: 1px;
		fill: var(--spectrum-global-color-gray-700);;
	}

	.innerArc {
		stroke: var(--spectrum-global-color-gray-600);
		stroke-width: 1px;
		fill: rgba(0,0,0, 0.15);
	}

	.svg-box-content {
		position: relative;
		top: 0;
		left: 0;
	}

	path {
		fill: url(#fillGradient);
/* 		fill: red; */
/* 		fill: conic-gradient(gold 40%, #f06 0); */
		
	}
	
	.track {
		stroke: var(--spectrum-global-color-gray-500);
		stroke-width: 1px;
		fill: none;
	}
	
	text {
		fill: var(--spectrum-global-color-gray-900);
		font-size: 1.8rem;
		text-anchor: middle;
	}
	
	.tick {
		stroke: var(--spectrum-global-color-gray-500);
		stroke-width: 1px;
		fill: none;
	}

	.tick-label {
		fill: var(--spectrum-global-color-gray-700);
		font-weight: 400;
		font-size: 0.65rem;
		text-anchor: middle;
	}

	.tick-label-min {
		fill: var(--spectrum-global-color-gray-700);
		font-weight: 400;
		font-size: 1rem;
		text-anchor: start;
	}
	.tick-label-max {
		fill: var(--spectrum-global-color-gray-700);
		font-weight: 400;
		font-size: 1rem;
		text-anchor: end;
	}
</style>