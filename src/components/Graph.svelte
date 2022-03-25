<script>
  import { onMount } from "svelte";
  import { axisBottom, axisRight, select } from "d3";
  export let dimensions;
  export let x;
  export let y;

  onMount(() => {
    let svg = select("#graph");
    svg
      .append("g")
      .call(xAxisSettings)
      .style("font-size", "16px")
      .attr("transform", `translate(0, ${y(0)})`);

    svg
      .append("g")
      .call(yAxisSettings)
      .style("font-size", "16px")
      .attr("transform", `translate(${x(0)}, 0)`);

    svg.append("g").attr("class", "senate-group");

    svg.append("g").attr("class", "house-group");

    let popup = svg
      .append("g")
      .attr("class", "pop-up")
      .style("display", "none");

    popup.append("rect").attr("fill", "white");
  });

  let xAxisSettings = axisBottom(x)
    .tickValues([-1, -0.5, 0.5, 1])
    .tickSize(5)
    .tickPadding(5)
    .tickSizeOuter(0);

  let yAxisSettings = axisRight(y)
    .tickValues([-1, -0.5, 0.5, 1])
    .tickSize(5)
    .tickPadding(5)
    .tickSizeOuter(0);
</script>

<div class="graph-container">
  <svg id="graph" viewBox={`0,0,${dimensions.width},${dimensions.height}`} />
</div>

<style>
  .graph-container {
    margin: 1rem -0.75rem;
    display: flex;
  }
  #graph {
    width: 100%;
    max-width: 600px;
    margin: 0 auto;
  }
</style>
