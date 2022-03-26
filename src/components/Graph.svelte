<script>
  import { onMount } from "svelte";
  import { axisBottom, axisRight, select } from "d3";
  export let dimensions;
  export let x;
  export let y;

  onMount(() => {
    let graph = select("#graph");

    graph
      .attr("width", dimensions.innerWidth)
      .attr("height", dimensions.innerHeight)
      .attr(
        "transform",
        `translate(${dimensions.marginX},${dimensions.marginY})`
      );

    graph.append("g").call(xAxisSettings).style("font-size", "12px");
    //.attr("transform", `translate(0, ${y(0)})`);

    graph.append("g").call(yAxisSettings);
    //.attr("transform", `translate(${x(0)}, 0)`);

    graph
      .append("text")
      .attr("x", x(0))
      .attr("y", -2)
      .attr("text-anchor", "middle")
      .style("font-weight", "bold")
      .style("font-size", "20px")
      .text("Socially Conservative");

    graph
      .append("text")
      .attr("x", x(0))
      .attr("y", dimensions.innerHeight + 20)
      .attr("text-anchor", "middle")
      .style("font-weight", "bold")
      .style("font-size", "20px")
      .text("Socially Liberal");

    graph
      .append("text")
      .attr("x", 0)
      .attr("y", y(0))
      .attr("text-anchor", "middle")
      .attr("transform", `translate(-${x(0) + 2},${y(0)}) rotate(-90)`)
      .style("font-weight", "bold")
      .style("font-size", "20px")
      .text("Fiscally Liberal");

    graph
      .append("text")
      .attr("x", dimensions.innerWidth)
      .attr("y", y(0))
      .attr("text-anchor", "middle")
      .attr("transform", `translate(${x(2) + 2},-${y(0)}) rotate(90)`)
      .style("font-weight", "bold")
      .style("font-size", "20px")
      .text("Fiscally Conservative");

    graph.append("g").attr("class", "senate-group");

    graph.append("g").attr("class", "house-group");
  });

  let xAxisSettings = axisBottom(x)
    .ticks(2)
    .tickFormat("")
    .tickSize(dimensions.innerHeight);

  let yAxisSettings = axisRight(y)
    .ticks(2)
    .tickFormat("")
    .tickSize(dimensions.innerWidth);
</script>

<div class="graph-container">
  <svg id="graph-area" viewBox={`0,0,${dimensions.width},${dimensions.height}`}>
    <g id="graph" />
  </svg>
</div>

<style>
  .graph-container {
    display: flex;
    margin-bottom: 0.5rem;
  }
  #graph-area {
    width: 100%;
  }
</style>
