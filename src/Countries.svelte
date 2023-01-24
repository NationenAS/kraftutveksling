<script>

import { createEventDispatcher } from 'svelte'
const dispatch = createEventDispatcher()

export let data
export let countryNames
export let focused

$: countries = data.filter(c => c.Country !== "")
$: exportValues = countries.map(c => parseInt(c.SidebarViewModel.export))
$: importValues = countries.map(c => parseInt(c.SidebarViewModel.import))
$: max = Math.max(...exportValues, ...importValues)
$: sumExport = exportValues.reduce((a, b) => { return a + b }, 0)
$: sumImport = importValues.reduce((a, b) => { return a + b }, 0)

const ticks = (max) => { // REFACTOR
    let t = []
    let twh =  max / 1000000
    if (twh > 0.5 && twh < 0.85) t = [0.25, 0.5]
    else if (twh >= 0.85 && twh < 1.1) t = [0.25, 0.5, 0.75]
    else if (twh >= 1.1 && twh < 1.6) t = [0.5, 1]
    else if (twh >= 1.6 && twh < 2.2) t = [0.5, 1, 1.5]
    else if (twh >= 2.2 && twh < 3.3) t = [1, 2]
    else if (twh >= 3.3 && twh < 4.4) t = [1, 2, 3]
    else if (twh >= 4.4 && twh < 5.5) t = [2, 4]
    return t.map(tick => [tick, tick / twh * 100])
}

function toTwh(number, decimals = 1) {
    return (number / 1000000).toLocaleString(undefined, { maximumFractionDigits: decimals })
}

function setCountry(c) {
    dispatch('setCountry', {
        country: c
    })
}

</script>

<div class="exchange-per-country" class:focused="{focused !== ""}">
    <h3>Utveksling per land</h3>
    <p>Trykk på et land for å se mer. {#if focused !== ""}<span on:click={() => {setCountry("")}} on:keypress={() => {setCountry("")}}>Tilbakestill &circlearrowright;</span>{/if}</p>
    <div class="countries">
        <div class="grid">
            <div class="tick-wrapper">
                {#each ticks(max) as tick, i}
                <div style="left: {tick[1]}%;">{tick[0].toLocaleString()}{ i == 0 ? " TWh" : ""}</div>
                {/each}
            </div>
        </div>
        {#each data as country}
        {#if country.Country === ""}
        <div class="country">
            <div>Andre</div>
            <div class="country-bars">
                <div class="country-export" data-twh="{toTwh(country.SidebarViewModel.export)}" style="width: {(country.SidebarViewModel.export - sumExport) / max * 100}%;"></div>
                <div class="country-import" data-twh="{toTwh(country.SidebarViewModel.import)}" style="width: {(country.SidebarViewModel.import - sumImport) / max * 100}%;"></div>
            </div>
        </div>
        {:else}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <div class="country" on:click={() => {setCountry(country.Country)}} class:focus="{focused == country.Country}">
            <div>{countryNames[country.Country]}</div>
            <div class="country-bars">
                <div class="country-export" data-twh="{toTwh(country.SidebarViewModel.export)}" style="width: {country.SidebarViewModel.export / max * 100}%;"></div>
                <div class="country-import" data-twh="{toTwh(country.SidebarViewModel.import)}" style="width: {country.SidebarViewModel.import / max * 100}%;"></div>
            </div>
        </div>
        {/if}
        {/each}
    </div>
</div>

<style>
.exchange-per-country {
  margin-top: 40px;
}
span {
    padding: 2px 4px;
    text-decoration: underline;
    cursor: pointer;
}
.countries {
    display: flex;
    flex-direction: column;
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
    padding-block: 4px;
}
.country:not(:last-child) {
    cursor: pointer;
}
.country:not(:last-child):hover {
    background: #eee;
}
.country.focus {
    background: rgb(222, 222, 222);
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