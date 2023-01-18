<script>
	export let title = ''
	let isHovered = false,
        x, 
        y
	
	function mouseOver(event) {
		isHovered = true
		x = event.pageX
		y = event.pageY
	}
	function mouseMove(event) {
		x = event.pageX
		y = event.pageY
	}
	function mouseLeave() {
		isHovered = false
	}
</script>

<!-- svelte-ignore a11y-mouse-events-have-key-events -->
<div
	on:mouseover={mouseOver}
    on:mouseleave={mouseLeave}
	on:mousemove={mouseMove}>
	<slot />
</div>

{#if isHovered}
<div style="top: {y + 20}px; left: {x}px;" class="tooltip">{title}</div>
{/if}

<style>
	.tooltip {
		background: #404040;
        font-size: .9em;
        color: white;
		border-radius: 99px;
		padding: 6px 8px;
		position: fixed;
        z-index: 10000;
        transform: translateX(-50%);
	}
</style>