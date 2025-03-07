<script lang="ts">
  import { onMount } from "svelte";
  import periodToDays from "../../../lib/period";

  function defaultLayout() {
    let periodAgo = new Date();
    let days = periodToDays(period);
    if (days != null) {
      periodAgo.setDate(periodAgo.getDate() - days);
    } else {
      periodAgo = null;
    }
    let tomorrow = new Date();
    tomorrow.setDate(tomorrow.getDate());
    return {
      title: false,
      autosize: true,
      margin: { r: 35, l: 70, t: 20, b: 20, pad: 10 },
      hovermode: "closest",
      plot_bgcolor: "transparent",
      paper_bgcolor: "transparent",
      height: 170,
      yaxis: {
        title: { text: "Response time (ms)" },
        gridcolor: "gray",
        showgrid: false,
        fixedrange: true,
      },
      xaxis: {
        title: { text: "Date" },
        showgrid: false,
        fixedrange: true,
        range: [periodAgo, tomorrow],
        visible: false,
      },
      dragmode: false,
    };
  }

  function initResponseTimes(): {
    [date: string]: { total: number; count: number };
  } {
    let responseTimes = {};
    let days = periodToDays(period);
    if (days) {
      if (days <= 7) {
        // Freq count for every minute
        for (let i = 0; i < 60 * 24 * days; i++) {
          let date = new Date();
          date.setSeconds(0, 0);
          // Round down to multiple of 5
          date.setMinutes(Math.floor(date.getMinutes() / 5) * 5 - i * 5);
          let dateStr = date.toISOString();
          responseTimes[dateStr] = { total: 0, count: 0 };
        }
      } else {
        // Freq count for every day
        for (let i = 0; i < days; i++) {
          let date = new Date();
          date.setHours(0, 0, 0, 0);
          date.setDate(date.getDate() - i);
          let dateStr = date.toISOString();
          responseTimes[dateStr] = { total: 0, count: 0 };
        }
      }
    }
    return responseTimes;
  }

  function bars() {
    let responseTimes = initResponseTimes();

    let days = periodToDays(period);
    for (let i = 1; i < data.length; i++) {
      let date = new Date(data[i][7]);
      if (days) {
        if (days <= 7) {
          // Round down to multiple of 5
          date.setMinutes(Math.floor(date.getMinutes() / 5) * 5, 0, 0);
        } else {
          date.setHours(0, 0, 0, 0);
        }
      }
      let dateStr = date.toISOString();
      if (!(dateStr in responseTimes)) {
        responseTimes[dateStr] = { total: 0, count: 0 };
      }
      responseTimes[dateStr].total += data[i][4];
      responseTimes[dateStr].count++;
    }

    // Combine date and avg response time into (x, y) tuples for sorting
    let responseTimeArr = [];
    for (let date in responseTimes) {
      let point = [new Date(date), 0];
      if (responseTimes[date].count > 0) {
        point[1] = responseTimes[date].total / responseTimes[date].count;
      }
      responseTimeArr.push(point);
    }
    // Sort by date
    responseTimeArr.sort((a, b) => {
      return a[0] - b[0];
    });
    // Split into two lists
    let dates = [];
    let rt = [];
    let min_rt = Number.POSITIVE_INFINITY;
    for (let i = 0; i < responseTimeArr.length; i++) {
      dates.push(responseTimeArr[i][0]);
      rt.push(responseTimeArr[i][1]);
      if (responseTimeArr[i][1] < min_rt) {
        min_rt = responseTimeArr[i][1];
      }
    }

    return [
      {
        x: dates,
        y: rt,
        type: "lines",
        marker: { color: "#707070" },
        hovertemplate: `<b>%{y:.1f}ms avg</b><br>%{x|%d %b %Y}</b><extra></extra>`,
        showlegend: false,
        line: { shape: "spline", smoothing: 1 },
      },
      // 'Hidden' horizontal line at min y to provide fill colour up to response time line
      {
        x: dates,
        y: Array(rt.length).fill(Math.max(min_rt - 5, 0)),
        type: "lines",
        marker: { color: "transparent" },
        fill: "tonexty",
        fillcolor: "#70707030",
        hovertemplate: `<b>%{y:.1f}ms avg</b><br>%{x|%d %b %Y}</b><extra></extra>`,
        showlegend: false,
      },
    ];
  }

  function buildPlotData() {
    return {
      data: bars(),
      layout: defaultLayout(),
      config: {
        responsive: true,
        showSendToCloud: false,
        displayModeBar: false,
      },
    };
  }

  function genPlot() {
    let plotData = buildPlotData();
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

  export let data: RequestsData, period: string;
</script>

<div id="plotly">
  <div id="plotDiv" bind:this={plotDiv}>
    <!-- Plotly chart will be drawn inside this DIV -->
  </div>
</div>
