<script>
  
import { onMount } from "svelte";
import Tooltip from "./Tooltip.svelte";
import { slide } from 'svelte/transition';

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

// Get data
onMount(async () => {
  fetch("https://statnett-utveksling.vercel.app/api")
    .then(r => r.json())
    .then(d => {
      build(d.data)
    })
    .catch(e => { console.log(e) })
})

function build(d) {
  let span = d.SidebarViewModel.max + Math.abs(d.SidebarViewModel.min)
  let zero = d.SidebarViewModel.max / span * 100
  function absToBar(value, index, start) {
    let date = new Date(start + (index * 86400000))
    let bar = {
      a: value,
      h: Math.abs(value) / span * 100,
      c: value > 0 ? "pos" : "neg",
      d: date.toLocaleDateString(undefined, {month: 'short', day: 'numeric'})
    }
    return bar
  }
  let bars = d.PhysicalFlowNetExchange.map((p, i) => absToBar(p, i, d.StartPointUTC))
  data = {
    points: bars,
    summarised: d.SidebarViewModel,
    timespan: {
      from: d.StartPointUTC,
      to: d.EndPointUTC
    },
    css: {
      zero: zero
    }
  }
}

</script>

<div class="exchange-app">
  <h2>Live: Kraftutveksling med utlandet hittil i år</h2>
  <p>Grafene viser hvor mye strøm som har blitt sendt og mottat gjennom utenlandskablene så langt i 2023.</p>
  <div class=exchange>
    <div>
      <h3>Eksport</h3>
      <div class="twh">{(data.summarised.export / 1000000).toLocaleString(undefined, { maximumFractionDigits: 2 })} TWh</div>
    </div>
    <div class=percentage>
      <div>
        <div class=export style="width: {data.summarised.exportPercent}%;">{(data.summarised.exportPercent).toFixed(0)}%</div>
        <div class=import style="width: {data.summarised.importPercent}%;">{(data.summarised.importPercent).toFixed(0)}%</div>
      </div>
    </div>
    <div>
      <h3>Import</h3>
      <div>{(data.summarised.import / 1000000).toLocaleString(undefined, { maximumFractionDigits: 2 })} TWh</div>
    </div>
  </div>
  <div class="exchange-per-day" style="--zero: {data.css.zero}%;">
    <h3>Balanse per dag</h3>
    <p>En mørk stolpe betyr at det er eksportert mer enn det er importert denne dagen.</p>
    <div class="days">
      {#each data.points as point, i}
      <div class="day" data-date="{point.d}">
        <Tooltip title="{point.d}: {(point.a).toLocaleString(undefined, { maximumFractionDigits: 0 })} MWh">
          <div class="bar {point.c}" style="height: {point.h}%;" transition:slide="{{ delay: 50 * i, duration: 1000 }}"></div>
        </Tooltip>
      </div>
      {/each}
    </div>
  </div>
  <div class="meta">
    1 terawattime (TWh) = 1 000 000 megawattimer (MWh) = 1 000 000 000 kilowattimer (kWh). Oppdatert {(new Date(data.timespan.to)).toLocaleDateString(undefined, {year: 'numeric', month: 'long', day: 'numeric'})}. Utvikling: Jarand Ullestad/Nationen. Kilde: Statnett.
  </div>
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
  gap: 20px;
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
h3 {
  margin: 0 0 .1em;
  font-size: 1.1em;
}
p {
  margin: 0;
  line-height: 1.3;
}
.exchange-per-day {
  margin-top: 30px;
  margin-bottom: 30px;
}
.days {
  display: flex;
  gap: 1%;
  height: 150px;
  margin-top: 15px;
  position: relative;
}
.day {
  flex: 1;
  height: 100%;
  max-width: 20px;
  position: relative;
}
.day:first-child::before {
  position: absolute;
  bottom: -20px;
  width: 100px;
  font-size: .8em;
  color: #404040;
  content: attr(data-date);
  line-height: 1;
}
.day:last-of-type::before {
  position: absolute;
  bottom: -20px;
  right: 0;
  text-align: right;
  width: 100px;
  font-size: .8em;
  color: #404040;
  content: attr(data-date);
  line-height: 1;
}
.bar {
  position: absolute;
  top: var(--zero);
  width: 100%;
}
.bar.pos {
  background: var(--export);
  transform: translateY(-100%);
  border-top-left-radius: 99px;
  border-top-right-radius: 99px;
}
.bar.neg {
  background: var(--import);
  border-bottom-left-radius: 99px;
  border-bottom-right-radius: 99px;
}
.bar:hover {
  filter:opacity(0.7)
}
.meta {
  margin-top: 40px;
  font-size: .9em;
  color: #999;
}
/* .import::before {
  position: absolute;
  top: -3px;
  left: 0;
  width: 2px;
  background: white;
  height: calc(100% + 6px);
  content: "";
} */
</style>