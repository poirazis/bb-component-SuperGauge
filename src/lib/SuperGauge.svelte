<script>
	import { tweened, spring } from 'svelte/motion';
	import { backInOut } from 'svelte/easing';
	import { arc as d3arc } from 'd3-shape';
	import { scaleLinear } from 'd3-scale';
    import { onMount } from 'svelte';

	export let gaugeType = 120
	export let trackSize
	export let showCenter = false

	export let min = 0
	export let max = 1000
	export let value = 350

	export let showNeedle = false
	export let showValue = true
	export let fillColor
	export let endColor

	export let showTicks = false
  	export let showTickLabels = false
  	export let majorTicks = 10
  	export let minorTicks = 0

	// Use a unique ID to target the elements Fill Gradient Definition
	let _id = Math.random()
	let valueArc

	// Array of configuration settigns for each gauge variation
	let gaugeTypeMap = [];
	gaugeTypeMap[90] =  { startAngle: -90, endAngle:0, innerRadius: 246, outerRadius:256, innerArcRadius: 88, cornerRadius:10, canvas: { width:256, height: 256 }, pivot: {x: 256, y: 256}, valuePos: {x: 256, y: 256} }
	gaugeTypeMap[120] = { startAngle: -60, endAngle:60, innerRadius: 118, outerRadius:128, innerArcRadius: 40, cornerRadius:10, canvas: { width:256, height: 128 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 110} }
	gaugeTypeMap[180] = { startAngle: -90, endAngle:90, innerRadius: 118, outerRadius:128, innerArcRadius: 40, cornerRadius:10, canvas: { width:256, height: 128 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 118} }
	gaugeTypeMap[240] = { startAngle: -120, endAngle:120, innerRadius: 118, outerRadius:128, innerArcRadius: 40, cornerRadius:10, canvas: { width:256, height: 192 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 118} }
	gaugeTypeMap[270] = { startAngle: -180, endAngle:90, innerRadius: 118, outerRadius:128, innerArcRadius: 40, cornerRadius:10, canvas: { width:256, height: 256 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 118} }
	gaugeTypeMap[360] = { startAngle: 0, endAngle:360, innerRadius: 118, outerRadius:128, innerArcRadius: 40, cornerRadius:10, canvas: { width:256, height: 256 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 134} }
	
	let gaugeConfig
	$: gaugeConfig = gaugeTypeMap[gaugeType];

	let trackSizeMap =[]
	trackSizeMap["tiny"] = 4
	trackSizeMap["small"] = 12
	trackSizeMap["medium"] = 18
	trackSizeMap["large"] = 24
	$: _width = trackSizeMap[trackSize]
	
	$: startAngle = gaugeConfig.startAngle;
	$: endAngle = gaugeConfig.endAngle;
	$: console.log(Number(trackSizeMap[trackSize]))
	$: console.log(trackSize)
	$: innerRadius = gaugeConfig.outerRadius - _width
	$: outerRadius = gaugeConfig.outerRadius;
	$: cornerRadius = gaugeConfig.cornerRadius;
	$: pivot = gaugeConfig.pivot;
	$: canvas = gaugeConfig.canvas;
	$: innerArcRadius = gaugeConfig.innerArcRadius;
	$: valuePos = gaugeConfig.valuePos;

	// An array with the angle and label of each tick
	let _majorTicks = []

    // $: _value = spring(50, { stiffness: .1 });
	$: _value.set(value);
    $: _value = tweened(50, { easing: backInOut, duration: 750 });
	$: scale = scaleLinear()
		.domain([min, max])
		.range([startAngle, endAngle]);
	
	$: generateTicks( majorTicks , minorTicks , gaugeConfig )
	
	$: valueAngle = scale($_value);

	$: fillColor = fillColor ? fillColor : "lime";
	$: endColor = endColor ? endColor : fillColor;

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
		.outerRadius(innerArcRadius)
		.startAngle(startAngle * Math.PI / 180)
		.endAngle(endAngle * Math.PI / 180)
		.cornerRadius(2);

	function generateTicks ( majorTicks, minorTicks ) {
		let _majorStep = ( max - min ) / majorTicks;
		let _minorStep = _majorStep / minorTicks;
		let _pos = Number(min) || 0;

		_majorTicks = [];
		while ( _pos <= max ) {
			_majorTicks.push({ angle:scale(Number(_pos)) , label: Math.round(_pos) });
			_pos += _majorStep;
		}
	}

	function setSelfGradient () {
		console.log(valueArc)
		let _path = "url(\"#" + _id + "\")"
		console.log(_path)
		valueArc.style.fill = _path;
	}

	onMount ( () => setSelfGradient () )
</script>

<div class="svg-box" style="--fillColor: {fillColor};">

	<svg class="svg-box-content" viewbox="-2, -2, { canvas.width + 4 }, { canvas.height + 4} " version="1.1" xmlns="http://www.w3.org/2000/svg">
		<path d={trackArc()} class="track" transform="translate({pivot.x}, {pivot.y})" />
		<path d={arc()} class="valueArc" transform="translate({pivot.x}, {pivot.y})" bind:this={valueArc}/>

		{#if showCenter}
			<path d={innerArc()} class="innerArc" transform="translate({pivot.x}, {pivot.y})" />
		{/if}

		{#if showTicks && _majorTicks.length > 0}
			{#each _majorTicks as tick}
				<line class="tick" x1={pivot.x} y1={outerRadius - innerRadius} x2={pivot.x} y2={ outerRadius - innerRadius + 12 } transform="rotate({Number(tick.angle)} {pivot.x} {pivot.y})"/>
				{#if showTickLabels}
					<text class="tick-label" x={pivot.x} y={outerRadius - innerRadius + 20} transform="rotate({tick.angle} {pivot.x} {pivot.y})"> {tick.label} </text>
				{/if}
			{/each}
		{/if}

		{#if showValue}
			<text class="value" transform="translate({valuePos.x} {valuePos.y})">
				{Math.round($_value)}
			</text>
		{/if} 

		{#if showNeedle}
			<polygon class="needle" 
				points="{pivot.x}, {_width + 4} {pivot.x + 2},{pivot.y - innerArcRadius - 2} {pivot.x - 2},{pivot.y - innerArcRadius - 2}" transform="rotate({valueAngle} {pivot.x}, {pivot.y})" />
		{/if}

		<defs>
		<linearGradient id={_id} x1="0%" y1="0%" x2="100%" y2="0%">
			<stop offset="0%"   stop-color={fillColor} />
			<stop offset="100%" stop-color={endColor} />
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

	.svg-box-content {
		position: relative;
		top: 0;
		left: 0;
	}
	.needle {
		fill: var(--spectrum-global-color-gray-400);
	}

	.track {
		stroke: var(--spectrum-global-color-gray-400);
		stroke-width: 1px;
		fill: none;
	}

	.innerArc {
		stroke: var(--spectrum-global-color-gray-400);
		stroke-width: 2px;
		fill: var(--spectrum-global-color-gray-400)
	}
	.tick {
		stroke: var(--spectrum-global-color-gray-400);
		stroke-width: 1px;
		fill: none;
	}
	.tick-label {
		fill: var(--spectrum-global-color-gray-600);
		font-weight: 400;
		font-size: 0.65rem;
		text-anchor: middle;
	}

	.value {
		font-weight: 500;
		font-size: 0.9rem;
		text-anchor: middle;
		fill: var(--spectrum-global-color-gray-900);
	}
</style>