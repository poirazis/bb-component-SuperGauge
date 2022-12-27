<script>
  import Speedometer from "svelte-speedometer"
  import SuperGauge from "./lib/SuperGauge.svelte";
  import { getContext } from "svelte"

  export let gaugeType
  export let trackSize = "medium"
  export let showCenter

  export let min
  export let max
  export let value

  export let arcSize
  export let showNeedle
  export let showValue
  export let fillColor
  export let endColor

  // Properties to set Major and Minor Ticks
  export let showTicks
  export let showTickLabels
  export let majorTicks 
  export let minorTicks

  const { styleable } = getContext("sdk")
  const component = getContext("component")
</script>

<div use:styleable={$component.styles}>

      <SuperGauge 
        {fillColor}
        {endColor}
        {trackSize} 
        {showCenter}
        {min}
        {max}
        {value} 
        {showNeedle} 
        {showValue}
        {showTicks}
        {showTickLabels}
        {majorTicks}
        {minorTicks}
        gaugeType={arcSize}
      />

</div>