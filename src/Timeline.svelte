<script>
import Tooltip from "./Tooltip.svelte";
import { slide } from 'svelte/transition';
import { getWeek } from "./getWeek";

export let data

function absToBar(value, index, start) { // start = timestamp, span = absolute span from lowest to highest value
  let date = new Date(start + (index * 86400000))
  let weekNumber = getWeek(start + (index * 86400000))
  let bar = {
    a: value,
    c: value > 0 ? "pos" : "neg",
    d: date.toLocaleDateString(undefined, {month: 'short', day: 'numeric'}),
    w: weekNumber
  }
  return bar
}

let timeResolution = data.PhysicalFlowNetExchange.length > 42 ? "week" : "day"

$: points = data.PhysicalFlowNetExchange.map((value, index) => absToBar(value, index, data.StartPointUTC))
$: values = points.map(p => p.a)
$: max = Math.max(...values)
$: span = max + Math.abs(Math.min(...values))
$: zero = max / span * 100

$: if (timeResolution == "week") {
  let w = {} // Temp week object
  points.forEach((p, i) => { // Loop through points
    let weekNumber = (p.w == 52 && i == 0) ? 0 : p.w
    if (!w[weekNumber]) { // If week doesn't exist in temp object, add new point
      let newWeek = p
      newWeek.d = "Uke " + newWeek.w
      w[weekNumber] = newWeek
    }
    else { // Else update week
      w[weekNumber].a += p.a
      w[weekNumber].c = w[weekNumber].a > 0 ? "pos" : "neg"
    }
  })
  points = Object.keys(w).map((key) => w[key])
}

</script>
<div class="exchange-per-day" style="--zero: {zero}%;">
  <h3>Balanse per {timeResolution == "day" ? "dag" : "uke"}</h3>
  <p>En mørk stolpe betyr at det er eksportert mer enn det er importert denne {timeResolution == "day" ? "dagen" : "uka"}.</p>
  <div class="items">
    {#each points as point, i}
    <div class="item" data-date="{point.d}">
    <Tooltip title="{point.d}: {point.a.toLocaleString(undefined, { maximumFractionDigits: 0 })} MWh">
        <div class="bar {point.c}" style="height: {Math.abs(point.a) / span * 100}%;" transition:slide="{{ delay: 50 * i, duration: 1000 }}"></div>
    </Tooltip>
    </div>
    {/each}
  </div>
</div>

<style>
.exchange-per-day {
  margin-top: 30px;
  margin-bottom: 45px;
}
.items {
  display: flex;
  gap: 1%;
  height: 150px;
  margin-top: 15px;
  position: relative;
}
.item {
  flex: 1;
  height: 100%;
  max-width: 20px;
  position: relative;
}
.item:first-child::before {
  position: absolute;
  bottom: -20px;
  width: 100px;
  font-size: .8em;
  color: #404040;
  content: attr(data-date);
  line-height: 1;
}
.item:last-child::before {
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
</style>