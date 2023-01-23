<script>
import Tooltip from "./Tooltip.svelte";
import { slide } from 'svelte/transition';
export let data
</script>
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

<style>
.exchange-per-day {
  margin-top: 30px;
  margin-bottom: 45px;
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
.day:last-child::before {
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