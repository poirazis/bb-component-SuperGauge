<script>
	import { tweened, spring } from 'svelte/motion';
	import { backInOut } from 'svelte/easing';
	import { arc as d3arc } from 'd3-shape';
	import { scaleLinear } from 'd3-scale';
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';

	export let arcSize = 120
	export let gaugeMinSize
	export let title 
	export let trackSize

	export let min = 0
	export let max = 1000
	export let value = 350

	export let showValue = true

	export let trackColor
	export let trackFillColor
	export let trackGradientColor = undefined
	export let tickLabelColor
	export let valueColor
	export let valueSize
	export let animationType = "spring"

	export let showTicks = false
  	export let showTickLabels = false
  	export let majorTicks = 10

	// Use a unique ID to target the elements Fill Gradient Definition
	let _id = Math.random()
	let valueArc

	// Array of configuration settigns for each gauge variation
	let gaugeTypeMap = [];
	gaugeTypeMap[90] =  { startAngle: -90, endAngle:0, innerRadius: 226, outerRadius:236, innerArcRadius: 88, cornerRadius:10, canvas: { width:256, height: 256 }, pivot: {x: 236, y: 236}, valuePos: {x: 155, y: 155} }
	gaugeTypeMap[120] = { startAngle: -60, endAngle:60, innerRadius: 118, outerRadius:128, innerArcRadius: 40, cornerRadius:10, canvas: { width:256, height: 148 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 110} }
	gaugeTypeMap[180] = { startAngle: -90, endAngle:90, innerRadius: 118, outerRadius:128, innerArcRadius: 40, cornerRadius:10, canvas: { width:256, height: 148 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 118} }
	gaugeTypeMap[240] = { startAngle: -120, endAngle:120, innerRadius: 118, outerRadius:128, innerArcRadius: 40, cornerRadius:10, canvas: { width:256, height: 192 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 128} }
	gaugeTypeMap[270] = { startAngle: -180, endAngle:90, innerRadius: 118, outerRadius:128, innerArcRadius: 40, cornerRadius:10, canvas: { width:256, height: 276 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 128} }
	gaugeTypeMap[360] = { startAngle: 0, endAngle:360, innerRadius: 118, outerRadius:128, innerArcRadius: 40, cornerRadius:10, canvas: { width:256, height: 276 }, pivot: {x: 128, y: 128}, valuePos: {x: 128, y: 128} }
	
	let gaugeConfig
	$: gaugeConfig = gaugeTypeMap[arcSize];

	let trackSizeMap =[]
	trackSizeMap["tiny"] = 4
	trackSizeMap["small"] = 12
	trackSizeMap["medium"] = 18
	trackSizeMap["large"] = 24
	trackSizeMap["huge"] = 48
	$: _width = trackSizeMap[trackSize]
	
	$: startAngle = gaugeConfig.startAngle;
	$: endAngle = gaugeConfig.endAngle;
	$: innerRadius = gaugeConfig.outerRadius - _width
	$: outerRadius = gaugeConfig.outerRadius;
	$: cornerRadius = gaugeConfig.cornerRadius;
	$: pivot = gaugeConfig.pivot;
	$: canvas = gaugeConfig.canvas;
	$: valuePos = gaugeConfig.valuePos;

	// An array with the angle and label of each tick
	let _majorTicks = []
	let _value = writable(0);

	$: _value.set(value);
	$: if (animationType === "spring") {
			_value = spring(50, { stiffness: .1 })
		} else if (animationType === "tweened") {
			_value = tweened(50, { easing: backInOut, duration: 750 })
		}

	$: scale = scaleLinear()
		.domain([min, max])
		.range([startAngle, endAngle]);
	
	$: generateTicks( majorTicks , gaugeConfig )
	
	$: valueAngle = scale($_value);

	$: trackFillColor = trackFillColor ? trackFillColor : "var(--primaryColor)";	
		
	$: _textBaseline = (arcSize == 180) || (arcSize == 120) ? "bottom" : "middle"

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

	function generateTicks ( majorTicks ) {
		let _majorStep = ( max - min ) / majorTicks;
		let _pos = Number(min) || 0;

		_majorTicks = [];
		while ( _pos <= max ) {
			_majorTicks.push({ angle:scale(Number(_pos)) , label: Math.round(_pos) });
			_pos += _majorStep;
		}

		if ( arcSize == 360 ) _majorTicks.shift();
	}

	// Each SV gradient element has a unique ID so you can show more than
	// one gauge. We need to set the fill of the valueArc element to this instance
	// gradient.
	function setSelfGradient () {
		let _path = "url(\"#" + _id + "\")"
		valueArc.style.fill = _path;
	}

	onMount ( () => setSelfGradient () )
</script>

<div class="svg-box" 
	style:min-width = {gaugeMinSize}
	style:--trackColor={ trackColor || "var(--spectrum-global-color-gray-400)" }
	style:--tickLabelColor={ tickLabelColor || "var(--spectrum-global-color-gray-400)" }
	style:--valueColor={ valueColor || "var(--spectrum-global-color-gray-700)" }
	>
	<svg class="svg-box-content" viewbox="-2, -2, { canvas.width + 4 }, { canvas.height + 4} " version="1.1" xmlns="http://www.w3.org/2000/svg">
		<path d={trackArc()} class="track" transform="translate({pivot.x}, {pivot.y})" />
		<path d={arc()} class="valueArc" transform="translate({pivot.x}, {pivot.y})" bind:this={valueArc}/>

		{#if showTicks && _majorTicks.length > 0}
			{#each _majorTicks as tick}
				<line class="tick" x1={pivot.x} y1={outerRadius - innerRadius} x2={pivot.x} y2={ outerRadius - innerRadius + 12 } transform="rotate({Number(tick.angle)} {pivot.x} {pivot.y})"/>
				{#if showTickLabels}
					<text class="tick-label" x={pivot.x} y={outerRadius - innerRadius + 22} transform="rotate({tick.angle} {pivot.x} {pivot.y})"> {tick.label} </text>
				{/if}
			{/each}
		{/if}

		{#if showValue}
			<text class="spectrum-Heading spectrum-Heading--size{valueSize}" dominant-baseline={ _textBaseline } transform="translate({valuePos.x} {valuePos.y})">
				{Math.round($_value)}
			</text>
		{/if} 

		<text class="title"  transform="translate({canvas.width / 2 } {canvas.height})">
			{title.toUpperCase()}
		</text>

		<defs>
		<linearGradient id={_id} x1="0%" y1="0%" x2="100%" y2="0%">
			<stop offset="0%"   stop-color={trackFillColor} />
			<stop offset="100%" stop-color={trackGradientColor || trackFillColor} />
		</linearGradient>
	</defs>
	</svg>

</div>


<style>
	.svg-box {
		width: 100%;
		min-width: 128px;
		position: relative;
	}

	.svg-box-content {
		position: relative;
		top: 0;
		left: 0;
	}

	.track {
		stroke: var(--trackColor);
		stroke-width: 1px;
		fill: none;
	}
	.tick {
		stroke: var(--trackColor);
		stroke-width: 1px;
		fill: none;
	}
	.tick-label {
		fill: var(--tickLabelColor);
		font-weight: 400;
		font-size: 0.65rem;
		text-anchor: middle;
	}

	.spectrum-Heading {
		text-anchor: middle;
		fill: var(--valueColor);
	}
	.title {
		text-anchor: middle;
		font-size: 0.70rem;
		fill: var(--spectrum-global-color-gray-600);
	}
</style>