<script>
  import Chart from "svelte-frappe-charts";

  function normalcdf(X) {
    //HASTINGS.  MAX ERROR = .000001
    const T = 1 / (1 + 0.2316419 * Math.abs(X));
    const D = 0.3989423 * Math.exp((-X * X) / 2);
    let Prob =
      D *
      T *
      (0.3193815 +
        T * (-0.3565638 + T * (1.781478 + T * (-1.821256 + T * 1.330274))));
    if (X > 0) {
      Prob = 1 - Prob;
    }
    return Prob;
  }

  let closingPrices = [
    350, 350, 181.5, 181.5, 229, 229.8, 262.97, 233.48, 248.88, 265.71, 256.02,
    258, 292, 303.12, 282.0, 222.35, 229.44,
  ];

  let todayClose;

  let mean =
    closingPrices.reduce((total, curr) => total + curr) / closingPrices.length;
  let sd = Math.sqrt(
    closingPrices.reduce((total, curr) => total + Math.pow(curr - mean, 2)) /
      closingPrices.length -
      1
  );

  $: score = (todayClose - mean) / sd;

  $: probabilityOfMarginCall = normalcdf(score);

  let labels = [];
  let probabilities = [];
  let cumulative = [];

  console.log(mean);
  console.log(sd);
  for (let i = 150; i <= 400; i += 10) {
    labels.push(`$${i}`);
    const chance = normalcdf((i - mean) / sd);
    cumulative.push(chance.toFixed(3));
  }

  console.log(cumulative);

  let data = {
    title: "Cumulative Chance of Margin Call",
    labels,
    datasets: [
      {
        name: "Chance of Margin Call",
        values: cumulative,
      },
    ],
  };
</script>

<main>
  <label
    >Sample Price: <input
      bind:value={todayClose}
      type="number"
      placeholder="313.14"
    /></label
  >

  {#if todayClose && todayClose > 0}
    <p>
      Probability of Margin Call at ${todayClose.toFixed(2)}:
      <b><i>{(probabilityOfMarginCall * 100).toFixed(2)}%</i></b>
    </p>
  {/if}

  <h3>Cumulative Chance of Margin Call</h3>
  <Chart {data} type="line" />

  <hr />

  Mean: {mean.toFixed(2)}<br />
  SD: {sd.toFixed(2)}<br />
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
