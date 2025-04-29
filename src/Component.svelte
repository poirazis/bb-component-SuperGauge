<script>
  import SuperGauge from "./lib/SuperGauge.svelte";
  import { getContext } from "svelte";

  const { styleable } = getContext("sdk");
  const component = getContext("component");

  export let arcSize;
  export let gaugeMinSize;
  export let trackSize = "medium";

  export let min = 0;
  export let max = 100;
  export let value = 35;
  export let valueSuffix = "";
  export let precision;

  export let showValue;

  export let trackColor;
  export let trackFillColor = "var(--spectrum-global-color-blue-400)";
  export let trackBackdropColor = "var(--spectrum-global-color-gray-50)";
  export let trackGradientColor;
  export let tickLabelColor;
  export let valueColor;
  export let valueSize;
  export let animationType;

  // Properties to set Major and Minor Ticks
  export let showTicks;
  export let showTickLabels;
  export let majorTicks;
  export let minorTicks;

  $: _value = Number(value) || 0;

  $: minorTicks = minorTicks > 0 ? minorTicks : 1;
  $: majorTicks = majorTicks > 0 ? majorTicks : 1;
  $: $component.styles = {
    ...$component.styles,
    normal: {
      width: "400px",
      ...$component.styles.normal,
    },
  };
</script>

<div use:styleable={$component.styles}>
  <SuperGauge
    {gaugeMinSize}
    {trackSize}
    {min}
    {max}
    value={_value}
    {valueSuffix}
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
    {trackBackdropColor}
  />
</div>
