<script lang="ts">
  import { onMount } from "svelte";

  function getBrowser(userAgent: string): string {
    if (userAgent == null) {
      return "Unknown";
    }
    if (userAgent.match(/curl\//)) {
      return "Curl";
    } else if (userAgent.match(/PostmanRuntime\//)) {
      return "Postman";
    } else if (userAgent.match(/insomnia\//)) {
      return "Insomnia";
    } else if (userAgent.match(/python-requests\//)) {
      return "Python requests";
    } else if (userAgent.match(/node-fetch\//)) {
      return "Nodejs fetch";
    } else if (userAgent.match(/go-http-client\//)) {
      return "Go http";
    } else if (userAgent.match(/Java\//)) {
      return "Java";
    } else if (userAgent.match(/Seamonkey\//)) {
      return "Seamonkey";
    } else if (userAgent.match(/Firefox\//)) {
      return "Firefox";
    } else if (userAgent.match(/Chrome\//)) {
      return "Chrome";
    } else if (userAgent.match(/Chromium\//)) {
      return "Chromium";
    } else if (userAgent.match(/Safari\//)) {
      return "Safari";
    } else if (userAgent.match(/Edg\//)) {
      return "Edge";
    } else if (userAgent.match(/OPR\//) || userAgent.match(/Opera\//)) {
      return "Opera";
    } else if (userAgent.match(/; MSIE /) || userAgent.match(/Trident\//)) {
      return "Internet Explorer";
    } else {
      return "Other";
    }
  }

  function browserPlotLayout() {
    let monthAgo = new Date();
    monthAgo.setDate(monthAgo.getDate() - 30);
    let tomorrow = new Date();
    tomorrow.setDate(tomorrow.getDate() + 1);
    return {
      title: false,
      autosize: true,
      margin: { r: 35, l: 70, t: 20, b: 20, pad: 0 },
      hovermode: "closest",
      plot_bgcolor: "transparent",
      paper_bgcolor: "transparent",
      height: 180,
      width: 411,
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
    let browserCount = {};
    for (let i = 1; i < data.length; i++) {
      let browser = getBrowser(data[i][2]);
      if (!(browser in browserCount)) {
        browserCount[browser] = 0;
      }
      browserCount[browser]++;
    }

    let browsers = [];
    let count = [];
    for (let browser in browserCount) {
      browsers.push(browser);
      count.push(browserCount[browser]);
    }
    return [
      {
        values: count,
        labels: browsers,
        type: "pie",
        marker: {
          colors: colors,
        },
      },
    ];
  }

  function browserPlotData() {
    return {
      data: pieChart(),
      layout: browserPlotLayout(),
      config: {
        responsive: true,
        showSendToCloud: false,
        displayModeBar: false,
      },
    };
  }

  function genPlot() {
    let plotData = browserPlotData();
    //@ts-ignore
    new Plotly.newPlot(
      plotDiv,
      plotData.data,
      plotData.layout,
      plotData.config
    );
  }

  let plotDiv: HTMLDivElement;
  let mounted = false;
  onMount(() => {
    mounted = true;
  });

  $: data && mounted && genPlot();
  export let data: RequestsData;
</script>

<div id="plotly">
  <div id="plotDiv" bind:this={plotDiv}>
    <!-- Plotly chart will be drawn inside this DIV -->
  </div>
</div>

<style>
  #plotDiv {
    margin-right: 20px;
  }
</style>
