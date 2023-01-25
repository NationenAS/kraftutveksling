<script>
  
import { onMount } from "svelte";
import Timeline from "./Timeline.svelte";
import Countries from "./Countries.svelte";

$: data = {
  points: [],
  summarised: {
    max: 0,
    min: 0,
    export: 0,
    import: 0,
    netto: 0,
    exportPercent: 0,
    importPercent: 0
  },
  timespan: {
    from: "0",
    to: "0"
  },
  css: {
    zero: 0
  }
}
let response = [],
    focused = ""

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
    .then(r => r.json())
    .then(d => {
      response = d
      populate("", response)
    })
})

function absToBar(value, index, start, span) { // start = timestamp, span = absolute span from lowest to highest value
  let date = new Date(start + (index * 86400000))
  let bar = {
    a: value,
    h: Math.abs(value) / span * 100,
    c: value > 0 ? "pos" : "neg",
    d: date.toLocaleDateString(undefined, {month: 'short', day: 'numeric'})
  }
  return bar
}

function populate(c, d) { // Country, Data
  d = response[response.findIndex((x) => x.Country === c)]
  let span = d.SidebarViewModel.max + Math.abs(d.SidebarViewModel.min)
  let zero = d.SidebarViewModel.max / span * 100
  let bars = d.PhysicalFlowNetExchange.map((p, i) => absToBar(p, i, d.StartPointUTC, span))
  data.points = bars
  data.summarised = d.SidebarViewModel
  data.timespan.from = d.StartPointUTC
  data.timespan.to = d.EndPointUTC
  data.css.zero = zero
}

function focusCountry(event) {
  focused = event.detail.country
  populate(event.detail.country, response)
}

</script>


<div class="exchange-app">
  <h2>Live: Kraftutveksling med {countryNames[focused]} hittil i år</h2>
  {#if response.length > 0}
  <p>Grafene viser hvor mye strøm som har blitt sendt og mottatt gjennom utenlandskablene så langt i 2023.</p>
  <div class=exchange>
    <div>
      <h3>Eksport</h3>
      <div class="twh">{(data.summarised.export / 1000000).toLocaleString(undefined, { maximumFractionDigits: 2 })} TWh</div>
    </div>
    <div class=percentage>
      <div>
        <div class=export style="width: {data.summarised.exportPercent}%;">{(data.summarised.exportPercent).toLocaleString(undefined, { maximumFractionDigits: 1 })}%</div>
        <div class=import style="width: {data.summarised.importPercent}%;">{(data.summarised.importPercent).toLocaleString(undefined, { maximumFractionDigits: 1 })}%</div>
      </div>
    </div>
    <div>
      <h3>Import</h3>
      <div>{(data.summarised.import / 1000000).toLocaleString(undefined, { maximumFractionDigits: 2 })} TWh</div>
    </div>
  </div>
  <Timeline data={data} />
  <Countries data={response} on:setCountry={focusCountry} {countryNames} {focused} />
  <div class="meta">
    1 terawattime (TWh) = 1 000 000 megawattimer (MWh) = 1 000 000 000 kilowattimer (kWh). Oppdatert {(new Date(data.timespan.to)).toLocaleDateString(undefined, {year: 'numeric', month: 'long', day: 'numeric'})}. Utvikling: Jarand Ullestad/Nationen. Kilde: Statnett.
  </div>
  {:else}
  <p>Laster inn data...</p>
  {/if}
</div>

<style>
.exchange-app {
  font-family: adelle_sansregular, "Adelle Sans";
  font-size: 16px;
  line-height: 1.3;
  margin-block: 40px;
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