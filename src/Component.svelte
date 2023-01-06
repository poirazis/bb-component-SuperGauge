<script>
  import SuperGauge from "./lib/SuperGauge.svelte";
  import { getContext } from "svelte"

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  export let title
  export let arcSize
  export let gaugeMinSize
  export let trackSize = "medium"

  export let min
  export let max
  export let value = 200
  export let precision

  export let showValue

  export let trackColor
  export let trackFillColor
  export let trackGradientColor
  export let tickLabelColor
  export let valueColor
  export let valueSize
  export let animationType

  // Properties to set Major and Minor Ticks
  export let showTicks
  export let showTickLabels
  export let majorTicks 
  export let minorTicks

  $: _value = Number(value) || 0;

  $: minorTicks = (minorTicks > 0) ? minorTicks : 1;
  $: majorTicks = (majorTicks > 0) ? majorTicks : 1;
</script>

<div use:styleable={$component.styles}>
      <SuperGauge 
        {title}
        {gaugeMinSize}
        {trackSize} 
        {min}
        {max}
        value={_value}
        {precision}
        {showValue}
        {showTicks}
        {showTickLabels}
        {majorTicks}
        {valueSize}
        {arcSize}
        {trackColor}
        {trackFillColor}
        {trackGradientColor}
        {tickLabelColor}
        {valueColor}
        {animationType}
      />     
</div>