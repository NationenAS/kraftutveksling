<script>

import Tooltip from "./Tooltip.svelte";

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

</script>

<div class="countries">
    {#each data as country}
    <div class="country">
        <div>{countryNames[country.Country]}</div>
        <div class="country-bars">
            <div class="country-export" data-twh="{(country.SidebarViewModel.export / 1000000).toLocaleString(undefined, { maximumFractionDigits: 1 })}" style="width: {country.SidebarViewModel.export / max * 100}%;">
            </div>
            <div class="country-import" data-twh="{(country.SidebarViewModel.import / 1000000).toLocaleString(undefined, { maximumFractionDigits: 1 })}" style="width: {country.SidebarViewModel.import / max * 100}%;"></div>
        </div>
    </div>
    {/each}
    <div class="country">
        <div>Andre</div>
        <div class="country-bars">
            <div class="country-export" data-twh="{((all.summarised.export - sumExport) / 1000000).toLocaleString(undefined, { maximumFractionDigits: 1 })}" style="width: {(all.summarised.export - sumExport) / max * 100}%;"></div>
            <div class="country-import" data-twh="{((all.summarised.import - sumImport) / 1000000).toLocaleString(undefined, { maximumFractionDigits: 1 })}" style="width: {(all.summarised.import - sumImport) / max * 100}%;"></div>
        </div>
    </div>
</div>

<style>
.countries {
    display: flex;
    flex-direction: column;
    gap: 8px;
    margin-top: 8px;
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