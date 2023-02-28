<script>
  
import { onMount } from "svelte";
import Timeline from "./Timeline.svelte";
import Countries from "./Countries.svelte";

let response = [],
    focused = ""

$: data = {
  SidebarViewModel: {},
  EndPointUTC: 0
}

const countryNames = {
    se: 'Sverige',
    dk: 'Danmark',
    de: 'Tyskland',
    en: 'England',
    nl: 'Nederland',
    fi: 'Finland',
    "": 'utlandet'
}

// Get data
onMount(async () => {
  fetch("https://statnett-utveksling.vercel.app/api/full")
    .then(r => r.json()).then(d => {
      response = d
      populate("")
    })
})

function populate(countryId) {
  data = {
    SidebarViewModel: {},
    EndPointUTC: 0
  }
  data = response[response.findIndex((x) => x.Country === countryId)]
  data = data
}

function focusCountry(event) {
  focused = event.detail.country
  populate(event.detail.country)
}

</script>
<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="true">
	<link href="https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;700&display=swap" rel="stylesheet">
</svelte:head>


<div class="exchange-app">
  <h2>Live: Kraftutveksling med {countryNames[focused]} hittil i år</h2>
  {#if response.length > 0}
  <p>Grafene viser hvor mye strøm som har blitt sendt og mottatt gjennom utenlandskablene så langt i 2023.</p>
  <div class=exchange>
    <div>
      <h3>Eksport</h3>
      <div class="twh">{(data.SidebarViewModel.export / 1000000).toLocaleString(undefined, { maximumFractionDigits: 2 })} TWh</div>
    </div>
    <div class=percentage>
      <div>
        <div class=export style="width: {data.SidebarViewModel.exportPercent}%;">{(data.SidebarViewModel.exportPercent).toLocaleString(undefined, { maximumFractionDigits: 1 })}%</div>
        <div class=import style="width: {data.SidebarViewModel.importPercent}%;">{(data.SidebarViewModel.importPercent).toLocaleString(undefined, { maximumFractionDigits: 1 })}%</div>
      </div>
    </div>
    <div>
      <h3>Import</h3>
      <div>{(data.SidebarViewModel.import / 1000000).toLocaleString(undefined, { maximumFractionDigits: 2 })} TWh</div>
    </div>
  </div>
  <Timeline {data} />
  <Countries data={response} on:setCountry={focusCountry} {countryNames} {focused} />
  <div class="meta">
    1 terawattime (TWh) = 1.000.000 megawattimer (MWh). 1 MWh = 1000 kilowattimer (kWh). Oppdatert {(new Date(data.EndPointUTC)).toLocaleDateString(undefined, {year: 'numeric', month: 'long', day: 'numeric'})}. Utvikling: Jarand Ullestad/Nationen. Kilde: Statnett.
  </div>
  {:else}
  <p>Laster inn data...</p>
  {/if}
</div>

<style>
.exchange-app {
  font-family: "Open Sans", sans-serif;
  font-size: 16px;
  line-height: 1.3;
  border-top: 3px solid #4d6711;
  --import: #d9d4b6;
  --export: #404040;
}
h2 {
  font-size: 1.4em;
  margin-bottom: .5em;
  font-family: tiempos_regular, "Tiempos Headline";
}
.exchange {
  display: flex;
  gap: 17px;
  margin-top: 30px;
}
.percentage {
  flex: 1;
  display: flex;
  align-items: center;
}
.percentage > div {
  display: flex;
  width: 100%;
}
.import, .export {
  padding: 10px 15px;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  transition: .5s ease-out;
  line-height: 1;
}
.export {
  background: var(--export);
  color: white;
  border-top-left-radius: 99px;
  border-bottom-left-radius: 99px;
}
.import {
  justify-content: flex-end;
  background: var(--import);
  border-top-right-radius: 99px;
  border-bottom-right-radius: 99px;
}
:global(.exchange-app h3) {
  margin: 0 0 .1em;
  font-size: 1.1em;
}
:global(.exchange-app p) {
  margin: 0;
  line-height: 1.3;
}
.meta {
  margin-top: 30px;
  font-size: .9em;
  color: #999;
}
</style>