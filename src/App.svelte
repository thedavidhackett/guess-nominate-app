<script>
  import { onMount } from "svelte";
  import { csv, scaleLinear, select, selectAll } from "d3";

  import Accordion from "./components/Accordion.svelte";
  import Dashboard from "./components/Dashboard.svelte";
  import Graph from "./components/Graph.svelte";

  let data = [[], []];
  let dataIdx = 0;
  let selected = "";
  let overallAccuracy = 0.0;
  let guessAccuracy = 0.0;
  let guessCount = 0;

  let dimensions = {
    width: 500,
    height: 500,
    marginX: 24,
    marginY: 24,
  };

  dimensions.innerWidth = dimensions.width - 2 * dimensions.marginX;
  dimensions.innerHeight = dimensions.height - 2 * dimensions.marginY;

  let democratBlue = "#2300EB";
  let republicanRed = "#EB3317";
  let independentYellow = "#EBCD02";

  let y = scaleLinear().domain([-1, 1]).range([dimensions.innerHeight, 0]);

  let x = scaleLinear()
    .domain([-1, 1]) //d3 extent
    .range([0, dimensions.innerWidth]);

  const setX = (d) => x(d.nominate_dim1);
  const setY = (d) => y(d.nominate_dim2);
  const setColor = (d) =>
    d.party == "Republican"
      ? republicanRed
      : d.party == "Democrat"
      ? democratBlue
      : independentYellow;

  onMount(async () => {
    data[0] = await csv("./data/senate.csv");
    data[1] = await csv("./data/house.csv");

    selectRandom();
  });

  const selectRandom = () => {
    selected =
      data[dataIdx][Math.floor(Math.random() * data[dataIdx].length)]
        .bioguide_id;
  };

  const showAnswer = () => {
    if (selected) {
      let guess = select(".guess");
      guess.on(".drag", null);
      let selector = dataIdx ? ".house-group" : ".senate-group";
      let selectedIdx = data[dataIdx].findIndex(
        (d) => d.bioguide_id == selected
      );
      let selectedData = data[dataIdx][selectedIdx];
      console.log(selected);
      console.log(selectedData);
      let group = select(selector);
      selectAll(".member").style("opacity", 0.5);
      let trueValue = group
        .selectAll("#" + selectedData.bioguide_id)
        .data([selectedData])
        .enter()
        .append("circle")
        .attr("r", 12)
        .attr("cx", setX)
        .attr("cy", setY)
        .attr("class", (d) => d.chamber.toLowerCase() + " member")
        .attr("fill", setColor)
        .on("mouseover", async (event, d) => {
          let label = group
            .append("g")
            .attr("id", `label-${selectedData.bioguide_id}`)
            .attr("transform", `translate(${setX(d)},${setY(d)})`);

          label
            .append("rect")
            .attr("width", 120)
            .attr("height", 60)
            .attr("x", 0)
            .attr("y", 0)
            .attr("fill", "white");

          let nameArray = d.name.split(",").reverse().join(" ").split(" ");
          let count = 1;
          let i = 0;

          while (i < nameArray.length) {
            let frag = "";
            while (frag.length < 12) {
              frag += nameArray[i] + " ";
              i++;
            }
            label
              .append("text")
              .text(frag)
              .attr("x", 5)
              .attr("y", 15 * count);
            i++;
            count++;
          }

          label
            .append("text")
            .text(d.party)
            .attr("x", 5)
            .attr("y", 15 * count);
          count++;
          label
            .append("text")
            .text(d.state_name)
            .attr("x", 5)
            .attr("y", 15 * count);
        })
        .on("mouseout", () => {
          select(`#label-${selectedData.bioguide_id}`).remove();
        });

      data[dataIdx] = [
        ...data[dataIdx].slice(0, selectedIdx),
        ...data[dataIdx].slice(selectedIdx + 1),
      ];

      guessCount++;
      let trueX = trueValue.attr("cx");
      let trueY = trueValue.attr("cy");
      let diffX = Math.abs(trueX - guess.attr("cx"));
      let diffY = Math.abs(trueY - guess.attr("cy"));
      let scoreX = 1 - diffX / (dimensions.innerWidth - Math.abs(trueX));

      let scoreY = 1 - diffY / (dimensions.innerHeight - Math.abs(trueY));

      guessAccuracy = (scoreX + scoreY) / 2;
      overallAccuracy =
        overallAccuracy * ((guessCount - 1) / guessCount) +
        ((scoreX + scoreY) / 2) * (1 / guessCount);

      selected = "";
    }
  };
</script>

<div class="container">
  <h1>Guess the Legislator's Ideology</h1>
  <div class="accordions">
    <Accordion title={"How do you measure ideology?"}>
      <p>
        <strong>DW NOMINATE Scores</strong> are used to determine how liberal or
        conservative a member of congress is. You can learn more about these
        scores at
        <a href="https://voteview.com" rel="norefferer" target="_blank"
          >voteview.com</a
        >. The scores range from 1 to -1 with 1 being the most conservative, 0
        being moderate, and -1 being the most liberal. The scores measure the
        legislators in two dimensions. Their voting patterns on economic issues
        and their voting patterns on other (generally social) issues.
      </p></Accordion
    >
    <Accordion title={"How to play"} open={true}>
      <p>
        Select a senator or represenative you'd like to guess (or choose
        randomly). Move the dot to where you think their ideology lays on the
        two dimensions. Closer to the left side means more economically liberal,
        closer to the top means more fiscally conservative.
      </p>
    </Accordion>
  </div>
  <Dashboard
    {data}
    {dimensions}
    bind:dataIdx
    bind:selected
    {overallAccuracy}
    {guessAccuracy}
    {selectRandom}
    {x}
    {y}
    {setColor}
  />
  <Graph {dimensions} {x} {y} />

  <div class="show-answer-container">
    <button
      class={`show-answer${selected ? "" : " disabled"}`}
      on:click={showAnswer}>Show Answer</button
    >
  </div>
</div>

<style>
  .accordions {
    margin-bottom: 1rem;
  }
  .container {
    margin: 0 auto;
    padding: 1rem;
  }

  @media only screen and (min-width: 532px) {
    .container {
      max-width: 532px;
    }
  }

  .show-answer-container {
    padding: 0 1rem;
  }

  button.show-answer {
    width: 100%;
  }
</style>
