<script>
  import { afterUpdate, onMount } from "svelte";
  import { drag, select, selectAll } from "d3";
  import SlimSelect from "slim-select";

  export let data;
  export let dataIdx;
  export let dimensions;
  export let selected;
  export let accuracy;
  export let selectRandom;
  export let x;
  export let y;
  export let setColor;
  let selectInput;
  $: percentAccuracy = accuracy * 100;

  onMount(() => {
    selectInput = new SlimSelect({
      select: "#slim-select",
      placeholder: "Select a congress person",
    });
  });

  afterUpdate(() => {
    selectInput.set(selected);
  });

  const selectSenate = () => {
    dataIdx = 0;
    selectAll(".senate").classed("d-none", false);
    selectAll(".house").classed("d-none", true);
    selected = "";
    select(".guess").remove();
  };

  const selectHouse = () => {
    dataIdx = 1;
    selectAll(".senate").classed("d-none", true);
    selectAll(".house").classed("d-none", false);
    selected = "";
    select(".guess").remove();
  };

  const handleUpdate = () => {
    if (selected) {
      let selector = dataIdx ? ".house-group" : ".senate-group";
      let selectedData = data[dataIdx].find((d) => d.bioguide_id == selected);
      let group = select(selector);
      selectAll(".guess").remove();
      selectAll(".member").style("opacity", 1);
      group
        .append("circle")
        .attr("r", "12")
        .attr("cx", x(0))
        .attr("cy", y(0))
        .attr("fill", () => setColor(selectedData))
        .attr("stroke", "black")
        .attr("stroke-width", 3)
        .attr("class", "guess")
        .call(
          drag()
            .on("start", (event, d) => {
              select(".guess").raise();
            })
            .on("drag", updateGuess)
        );
    }
  };

  const updateGuess = (event, d) => {
    let xcord = event.x;
    let ycord = event.y;

    if (xcord > dimensions.width - dimensions.marginX) {
      xcord = dimensions.width - dimensions.marginX;
    } else if (xcord < dimensions.marginX) {
      xcord = dimensions.marginX;
    }

    if (ycord > dimensions.height - dimensions.marginY) {
      ycord = dimensions.height - dimensions.marginY;
    } else if (ycord < dimensions.marginY) {
      ycord = dimensions.marginY;
    }

    select(".guess").attr("cx", xcord).attr("cy", ycord);
  };
</script>

<div class="dashboard">
  <div class="radio-buttons">
    <button
      on:click={selectSenate}
      class={"radio-button" + (dataIdx ? "" : " active")}>Senate</button
    ><button
      on:click={selectHouse}
      class={"radio-button" + (dataIdx ? " active" : "")}>House</button
    >
  </div>
  <div class="select-container">
    <label for="slim-select"
      >Select a {dataIdx ? "House " : "Senate "}Member</label
    >
    <!-- svelte-ignore component-name-lowercase -->
    <select id="slim-select" bind:value={selected} on:change={handleUpdate}>
      <option data-placeholder="true" />
      {#each data[dataIdx] as rep}
        <option value={rep.bioguide_id}>
          {rep.name +
            " " +
            rep.state_abbrev +
            "-" +
            (rep.district_code ? rep.district_code + "-" : "") +
            rep.party[0]}
        </option>
      {/each}
    </select>
  </div>
  <div class="accuracy-container">
    <button class="random-button" on:click={selectRandom}>Random</button>
    <span class="accuracy">
      Accuracy: {percentAccuracy.toFixed(0) + "%"}
    </span>
  </div>
</div>

<style>
  .dashboard {
    margin-bottom: 1rem;
  }
  .radio-buttons {
    margin-bottom: 0.5rem;
  }
  .radio-button {
    background-color: lightgray;
    border: solid 2px black;
  }
  .radio-button:first-of-type {
    border-radius: 5px 0 0 5px;
    border-right: none;
  }

  .radio-button.active {
    background-color: black;
    color: white;
  }

  .radio-button:last-of-type {
    border-radius: 0 5px 5px 0;
  }

  .random-button {
    margin-right: 1rem;
  }

  .select-container {
    margin-bottom: 0.5rem;
  }

  .accuracy-container {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
  }

  .accuracy {
    font-weight: bold;
    font-size: 1.25rem;
  }
</style>
