<script>
  import { afterUpdate, onMount } from "svelte";
  import { drag, select, selectAll } from "d3";
  import SlimSelect from "slim-select";

  export let data;
  export let dataIdx;
  export let dimensions;
  export let selected;
  export let overallAccuracy;
  export let guessAccuracy;
  export let selectRandom;
  export let x;
  export let y;
  export let setColor;
  let green = "#77d970";

  let selectInput;
  $: percentAccuracy = overallAccuracy * 100;
  $: guessPercentAccuracy = guessAccuracy * 100;

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
        .attr("stroke", "yellowgreen")
        .attr("stroke-width", 3)
        .attr("class", "guess")
        .call(
          drag()
            .on("start", (event, d) => {
              let guess = select(".guess");
              guess.raise();
              selectAll("animate").attr("values", "0");
            })
            .on("drag", updateGuess)
            .on("end", () => {
              select(".guess animate").attr("values", "1;0;1");
            })
        )
        .append("animate")
        .attr("attributeName", "stroke-opacity")
        .attr("values", "1;0;1")
        .attr("dur", "2s")
        .attr("repeatCount", "indefinite");
    }
  };

  const updateGuess = (event, d) => {
    let xcord = event.x;
    let ycord = event.y;

    if (xcord > dimensions.innerWidth) {
      xcord = dimensions.innerWidth;
    } else if (xcord < 0) {
      xcord = 0;
    }

    if (ycord > dimensions.innerHeight) {
      ycord = dimensions.innerHeight;
    } else if (ycord < 0) {
      ycord = 0;
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
    <div class="accuracy">
      <div>Overall Accuracy: {percentAccuracy.toFixed(0) + "%"}</div>
      <div>Last Guess: {guessPercentAccuracy.toFixed(0) + "%"}</div>
    </div>
  </div>
</div>

<style>
  .dashboard {
    margin-bottom: 1rem;
  }
  .radio-buttons {
    margin-bottom: 0.5rem;
  }

  .radio-button:first-of-type {
    border-radius: 5px 0 0 5px;
    border-right: none;
  }

  .radio-button:last-of-type {
    border-radius: 0 5px 5px 0;
  }

  .radio-button:not(.active) {
    background-color: var(--gray);
    color: black;
  }

  .random-button {
    margin-right: 1rem;
    height: fit-content;
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
    text-align: right;
  }
</style>
