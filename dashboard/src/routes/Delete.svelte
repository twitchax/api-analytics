<script lang="ts">
  let state: "delete" | "loading" | "deleted" | "error" = "delete";
  let apiKey = "";
  async function genAPIKey() {
    setState("loading");
    const response = await fetch(
      `https://www.apianalytics-server.com/api/delete/${apiKey}`
    );

    if (response.status === 200) {
      setState("deleted");
    } else {
      setState("error");
    }
  }

  function setState(value: typeof state) {
    state = value;
  }
</script>

<div class="generate">
  <div class="content">
    <h2>Delete account</h2>
    <input type="text" bind:value={apiKey} placeholder="Enter API key" />
    <button
      id="formBtn"
      on:click={genAPIKey}
      class:no-display={state != "delete"}>Delete</button
    >
    <button id="formBtn" class:no-display={state != "loading"}>
      <div class="spinner">
        <div class="loader" />
      </div>
    </button>
    <button
      id="formBtn"
      class="copied-btn"
      class:no-display={state != "deleted"}>Deleted</button
    >
    <button id="formBtn" class:no-display={state != "error"}>Error</button>
  </div>
  <div class="details">
    <div class="keep-secure">Keep your API key safe and secure.</div>
    <div class="highlight logo">API Analytics</div>
    <img class="footer-logo" src="img/logo.png" alt="" />
  </div>
</div>

<style scoped>
  .spinner {
    height: auto;
  }
  .loader {
    border: 3px solid #343434;
    border-top: 3px solid var(--highlight);
    height: 10px;
    width: 10px;
  }
</style>
