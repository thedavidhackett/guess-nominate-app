<script>
  import { onMount } from "svelte";
  import {
    axisBottom,
    axisRight,
    csv,
    drag,
    scaleLinear,
    select,
    selectAll,
  } from "d3";
  import SlimSelect from "slim-select";

  let senate = [];
  let house = [];
  let senateSelected = true;
  let selected = "";

  let dimensions = {
    width: 600,
    height: 600,
    marginTop: 10,
    marginBottom: 10,
    marginLeft: 10,
    marginRight: 10,
  };

  let y = scaleLinear()
    .domain([-1, 1])
    .range([dimensions.height - dimensions.marginTop, dimensions.marginBottom]);

  let x = scaleLinear()
    .domain([-1, 1]) //d3 extent
    .range([dimensions.marginLeft, dimensions.width - dimensions.marginRight]);

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

  const setX = (d) => x(d.nominate_dim1);
  const setY = (d) => y(d.nominate_dim2);
  const setColor = (d) =>
    d.party == "Republican" ? "red" : d.party == "Democrat" ? "blue" : "yellow";

  onMount(async () => {
    senate = await csv("./data/senate.csv");
    house = await csv("./data/house.csv");
    new SlimSelect({
      select: "#slim-select",
    });

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

    svg.append("g").attr("class", "senate");
    // .selectAll(".senator")
    // .data(senate)
    // .join("circle")
    // .attr("r", 12)
    // .attr("cx", setX)
    // .attr("cy", setY)
    // .attr("class", "senator")
    // .attr("fill", setColor);

    svg.append("g").attr("class", "house");
    // .selectAll(".rep")
    // .data(house)
    // .join("circle")
    // .attr("r", 12)
    // .attr("cx", setX)
    // .attr("cy", setY)
    // .attr("class", "rep d-none")
    // .attr("fill", setColor);
  });

  const selectSenate = () => {
    senateSelected = true;
    selectAll(".senator").classed("d-none", false);
    selectAll(".rep").classed("d-none", true);
  };

  const selectHouse = () => {
    senateSelected = false;
    selectAll(".senator").classed("d-none", true);
    selectAll(".rep").classed("d-none", false);
  };

  const handleUpdate = () => {
    let selector = senateSelected ? ".senate" : ".house";
    let data = senateSelected ? senate : house;
    let selectedData = data.find((d) => d.bioguide_id == selected);
    let group = select(selector);
    selectAll(".guess").remove();
    group
      .append("circle")
      .attr("r", "12")
      .attr("cx", x(0))
      .attr("cy", y(0))
      .attr("fill", () => setColor(selectedData))
      .attr("class", "guess")
      .call(
        drag()
          .on("start", (event, d) => {
            select(".guess").raise().attr("stroke", "black");
          })
          .on("drag", updateGuess)
          .on("end", (event, d) => select(".guess").attr("stroke", null))
      );
  };

  const updateGuess = (event, d) => {
    let xcord = event.x;
    let ycord = event.y;

    if (xcord > dimensions.width - dimensions.marginRight) {
      xcord = dimensions.width - dimensions.marginRight;
    } else if (xcord < dimensions.marginLeft) {
      xcord = dimensions.marginLeft;
    }

    if (ycord > dimensions.height - dimensions.marginBottom) {
      ycord = dimensions.height - dimensions.marginBottom;
    } else if (ycord < dimensions.marginTop) {
      ycord = dimensions.marginTop;
    }

    select(".guess").attr("cx", xcord).attr("cy", ycord);
  };
</script>

<div class="container">
  <h1>Can Your Guess How Liberal or Conservative Your Congress Person Is?</h1>
  <div class="dashboard">
    <div class="buttons">
      <button on:click={selectSenate} class={senateSelected ? "active" : ""}
        >Senate</button
      ><button on:click={selectHouse} class={senateSelected ? "" : "active"}
        >House</button
      >
    </div>
    <!-- svelte-ignore component-name-lowercase -->
    <select id="slim-select" bind:value={selected} on:change={handleUpdate}>
      {#if senateSelected}
        {#each senate as senator}
          <option value={senator.bioguide_id}>
            {senator.name + " " + senator.state_abbrev + "-" + senator.party[0]}
          </option>
        {/each}
      {/if}
      {#if !senateSelected}
        {#each house as rep}
          <option value={rep.bioguide_id}>
            {rep.name +
              " " +
              rep.state_abbrev +
              "-" +
              rep.district_code +
              "-" +
              rep.party[0]}
          </option>
        {/each}
      {/if}
    </select>
  </div>
  <div class="graph-container">
    <svg id="graph" viewBox={`0,0,${dimensions.width},${dimensions.height}`} />
  </div>
</div>

<style>
  .container {
    margin: 0 auto;
    padding: 1rem;
  }

  @media only screen and (min-width: 600px) {
    .container {
      max-width: 750px;
    }
  }

  .buttons {
    margin-bottom: 0.5rem;
  }
  button {
    background-color: lightgray;
    padding: 0.5rem 1.5rem;
    border: solid 2px black;
    text-transform: uppercase;
    font-weight: bold;
  }
  button:first-of-type {
    border-radius: 5px 0 0 5px;
    border-right: none;
  }

  button.active {
    background-color: black;
    color: white;
  }

  button:last-of-type {
    border-radius: 0 5px 5px 0;
  }

  .dashboard {
    margin-bottom: 1rem;
  }
  .graph-container {
    margin: 0 -0.75rem;
    display: flex;
  }
  #graph {
    width: 100%;
    max-width: 600px;
    margin: 0 auto;
  }
</style>
