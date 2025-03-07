<script lang="ts">
  import { onMount } from "svelte";

  function setVersions() {
    let v: Set<string> = new Set();
    for (let i = 1; i < data.length; i++) {
      let match = data[i][1].match(/[^a-z0-9](v\d)[^a-z0-9]/i);
      if (match) {
        v.add(match[1]);
      }
    }
    versions = v;

    if (versions.size > 1) {
      setTimeout(genPlot, 1000);
    }
  }

  function versionPlotLayout() {
    return {
      title: false,
      autosize: true,
      margin: { r: 35, l: 70, t: 10, b: 20, pad: 0 },
      hovermode: "closest",
      plot_bgcolor: "transparent",
      paper_bgcolor: "transparent",
      height: 180,
      yaxis: {
        title: { text: "Requests" },
        gridcolor: "gray",
        showgrid: false,
        fixedrange: true,
      },
      xaxis: {
        visible: false,
      },
      dragmode: false,
    };
  }

  let colors = [
    "#3FCF8E", // Green
    "#5784BA", // Blue
    "#EBEB81", // Yellow
    "#218B82", // Sea green
    "#FFD6A5", // Orange
    "#F9968B", // Salmon
    "#B1A2CA", // Purple
    "#E46161", // Red
  ];

  function pieChart() {
    let versionCount = {};
    for (let i = 1; i < data.length; i++) {
      let match = data[i][1].match(/[^a-z0-9](v\d)[^a-z0-9]/i);
      if (match) {
        let version = match[1];
        if (!(version in versionCount)) {
          versionCount[version] = 0;
        }
        versionCount[version]++;
      }
    }

    let versions = [];
    let count = [];
    for (let version in versionCount) {
      versions.push(version);
      count.push(versionCount[version]);
    }

    return [
      {
        values: count,
        labels: versions,
        type: "pie",
        marker: {
          colors: colors,
        },
      },
    ];
  }

  function versionPlotData() {
    return {
      data: pieChart(),
      layout: versionPlotLayout(),
      config: {
        responsive: true,
        showSendToCloud: false,
        displayModeBar: false,
      },
    };
  }

  function genPlot() {
    let plotData = versionPlotData();
    //@ts-ignore
    new Plotly.newPlot(
      plotDiv,
      plotData.data,
      plotData.layout,
      plotData.config
    );
  }

  let versions: Set<string>;
  let plotDiv: HTMLDivElement;
  let mounted = false;
  onMount(() => {
    setTimeout(() => {
      mounted = true;
    }, 500);
  });

  $: data && mounted && setVersions();

  export let data: RequestsData;
</script>

{#if versions != undefined && versions.size > 1}
  <div class="card">
    <div class="card-title">Version</div>
    <div id="plotly">
      <div id="plotDiv" bind:this={plotDiv}>
        <!-- Plotly chart will be drawn inside this DIV -->
      </div>
    </div>
  </div>
{/if}

<style scoped>
  .card {
    margin: 2em 0 2em 0;
    padding-bottom: 1em;
    flex: 1;
  }
  #plotDiv {
    margin-right: 20px;
  }
  @media screen and (max-width: 1030px) {
    .card {
      width: auto;
      flex: 1;
      margin: 0 0 2em 0;
    }
  }
</style>
