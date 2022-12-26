<script>
  import Speedometer from "svelte-speedometer"
  import SuperGauge from "./lib/SuperGauge.svelte";
  import { getContext } from "svelte"

  export let min
  export let max
  export let value
  export let gaugeType
  export let arcSize
  export let showNeedle
  export let showValue
  export let fillColor

  // Properties to set Major and Minor Ticks
  // export let showTicks
  // export let showTickLabels
  export let majorTicks 
  // export let minorTicks

  const { styleable } = getContext("sdk")
  const component = getContext("component")
</script>

<div use:styleable={$component.styles}>

      <SuperGauge 
        color={fillColor} 
        {min}
        {max}
        {value} 
        {showNeedle} 
        {showValue}
        {majorTicks}
      />

</div>
