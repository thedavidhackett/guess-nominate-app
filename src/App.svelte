<script>
  import { onMount, afterUpdate } from "svelte";
  import { csv, scaleLinear, select, selectAll } from "d3";

  import Dashboard from "./components/Dashboard.svelte";
  import Graph from "./components/Graph.svelte";

  let data = [[], []];
  let dataIdx = 0;
  let selected = "";
  let accuracy = 0.0;
  let guessCount = 0;

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

  const setX = (d) => x(d.nominate_dim1);
  const setY = (d) => y(d.nominate_dim2);
  const setColor = (d) =>
    d.party == "Republican" ? "red" : d.party == "Democrat" ? "blue" : "yellow";

  onMount(async () => {
    data[0] = await csv("./data/senate.csv");
    data[1] = await csv("./data/house.csv");

    selectRandom();
  });

  afterUpdate(() => console.log(selected));

  const selectRandom = () => {
    selected =
      data[dataIdx][Math.floor(Math.random() * data[dataIdx].length)]
        .bioguide_id;
  };

  const showAnswer = () => {
    let guess = select(".guess");
    guess.on(".drag", null);
    let selector = dataIdx ? ".house-group" : ".senate-group";
    let selectedIdx = data[dataIdx].findIndex((d) => d.bioguide_id == selected);
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
    let scoreX =
      1 -
      diffX /
        (dimensions.width - dimensions.marginLeft - dimensions.marginRight);

    let scoreY =
      1 -
      diffY /
        (dimensions.height - dimensions.marginBottom - dimensions.marginTop);
    accuracy =
      accuracy * ((guessCount - 1) / guessCount) +
      ((scoreX + scoreY) / 2) * (1 / guessCount);

    selected = "";
  };
</script>

<div class="container">
  <h1>Can Your Guess How Liberal or Conservative Your Congress Person Is?</h1>
  <Dashboard
    {data}
    {dimensions}
    {dataIdx}
    bind:selected
    {accuracy}
    {selectRandom}
    {x}
    {y}
    {setColor}
  />
  <Graph {dimensions} {x} {y} />

  <div>
    <button class="show-answer" on:click={showAnswer}>Show Answer</button>
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

  button.show-answer {
    border-radius: 5px;
    border: solid 2px black;
    width: 100%;
  }
</style>
