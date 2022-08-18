<script>
	import { scaleLinear } from "d3-scale";
  import { arc } from "d3-shape";

  export let percent = 100;

  const domain = [0, 40, 80, 100];
  const range = ["red", "red", "teal", "teal"];
  
  const colorScale = scaleLinear().domain(domain).range(range);

  function drawArc(percent) {
    return arc()({
      innerRadius: 13,
      outerRadius: 16,
      startAngle: 0,
      endAngle: (Math.PI * 2) * (percent / 100)
    });
  }
</script>

<div class="house">
  <svg viewBox="-4 -4 32 32" fill={colorScale(percent != null ? percent : 0)}>
    <path d="M 19 9.3 V 4 h -3 v 2.6 L 12 3 L 2 12 h 3 v 8 h 14 v -8 h 3 z"/>
    <path transform="translate(12,12)" d={drawArc(100)} fill="#ddd"/>
    <path transform="translate(12,12)" d={drawArc(percent)}/>
  </svg>
  <div class="house-text">{@html percent != null ? `${Math.round(percent)}%` : '&#9211;'}</div>
</div>

<style>
  .house {
    width: 110px;
    position: relative;
  }
  svg {
    width: 110px;
    overflow: visible;
  }
  .house-text {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);
    color: white;
  }
</style>