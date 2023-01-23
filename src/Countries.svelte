<script>

export let data
export let all

let countryNames = {
    se: 'Sverige',
    dk: 'Danmark',
    de: 'Tyskland',
    en: 'England'
}
let exportValues = data.map(c => parseInt(c.SidebarViewModel.export))
let importValues = data.map(c => parseInt(c.SidebarViewModel.import))
let max = Math.max(...exportValues, ...importValues)
let sumExport = exportValues.reduce((a, b) => { return a + b }, 0)
let sumImport = importValues.reduce((a, b) => { return a + b }, 0)
const ticks = (max) => { // REFACTOR
    let t = []
    let twh =  max / 1000000
    if (twh > 0.5 && twh < 0.75) t = [0.25, 0.5]
    else if (twh >= 0.75 && twh < 1) t = [0.25, 0.5, 0.75]
    else if (twh >= 1 && twh < 1.5) t = [0.5, 1]
    else if (twh >= 1.5 && twh < 2) t = [0.5, 1, 1.5]
    else if (twh >= 2 && twh < 3) t = [1, 2]
    else if (twh >= 3 && twh < 4) t = [1, 2, 3]
    else if (twh >= 4 && twh < 5) t = [2, 4]
    return t.map(tick => [tick, tick / twh * 100])
}
console.log(ticks(max))

function toTwh(number, decimals = 1) {
    return (number / 1000000).toLocaleString(undefined, { maximumFractionDigits: decimals })
}

</script>

<div class="countries">
    {#each data as country}
    <div class="country">
        <div>{countryNames[country.Country]}</div>
        <div class="country-bars">
            <div class="country-export" data-twh="{toTwh(country.SidebarViewModel.export)}" style="width: {country.SidebarViewModel.export / max * 100}%;"></div>
            <div class="country-import" data-twh="{toTwh(country.SidebarViewModel.import)}" style="width: {country.SidebarViewModel.import / max * 100}%;"></div>
        </div>
    </div>
    {/each}
    <div class="country">
        <div>Andre</div>
        <div class="country-bars">
            <div class="country-export" data-twh="{toTwh(all.summarised.export)}" style="width: {(all.summarised.export - sumExport) / max * 100}%;"></div>
            <div class="country-import" data-twh="{toTwh(all.summarised.import)}" style="width: {(all.summarised.import - sumImport) / max * 100}%;"></div>
        </div>
    </div>
    <div class="grid">
        <div class="tick-wrapper">
            {#each ticks(max) as tick}
            <div style="left: {tick[1]}%;">{tick[0].toLocaleString()} TWh</div>
            {/each}
        </div>
    </div>
</div>

<style>
.countries {
    display: flex;
    flex-direction: column;
    gap: 8px;
    margin-top: 23px;
    position: relative;
}
.grid {
    position: absolute;
    top: -20px;
    left: 85px;
    width: calc(100% - 85px);
    height: 100%;
    z-index: -1;
}
.tick-wrapper {
    position: relative;
    height: 100%;
    width: 100%;
}
.tick-wrapper > div {
    position: absolute;
    height: calc(100% + 20px);
    border-left: 1px solid #999;
    font-size: 0.8em;
    padding-left: 3px;
    color: #404040;
}
.country {
    display: grid;
    grid-template-columns: 85px 1fr;
    align-items: center;
}
.country-bars {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    justify-content: center;
    gap: 3px;
}
.country-bars > div {
    position: relative;
    background: var(--export);
    border-bottom-right-radius: 99px;
    border-top-right-radius: 99px;
    height: 13px;
    width: 0;
}
.country-bars > div:hover {
    opacity: 0.7;
}
.country-bars .country-import {
    background: var(--import);
    transition: .5s ease-in;
}
</style>