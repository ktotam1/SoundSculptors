<script>
	// https://github.com/topojson/us-atlas
	// https://github.com/d3/d3-geo
	// https://observablehq.com/@mbostock/u-s-state-map
	// TODO: https://observablehq.com/@d3/u-s-map
	// TODO: https://observablehq.com/@jeantimex/us-state-county-map
	
	import { onMount } from 'svelte';
	import * as topojson from 'topojson-client';
	import { geoPath, geoConicEqualArea, geoMercator } from 'd3-geo';
	import { draw } from 'svelte/transition';
    import {markets, iso2CodesByCountryName} from './markets'
	// https://github.com/topojson/us-atlas#us-atlas-topojson
	const projection = geoMercator()//geoConicEqualArea().scale(1000).translate([487.5, 305])
	
	const path = geoPath().projection(projection);
	let countries = []
	let mesh;
	export let selected;
	let max = 1
	let min = 0
	export let colors = {};
	$: max = Math.max(...Object.values(colors))
	$: min = Math.min(...Object.values(colors))

    let latitude = 0;
    let longitude = 0;
    
	const width = 1000//window.innerWidth - 10;
	const height = 550//window.innerHeight - 20;
	//$: console.log({ selected })
		if (navigator.geolocation) {
	navigator.geolocation.getCurrentPosition(function(position) {
		latitude = position.coords.latitude;
		longitude = position.coords.longitude;
	});
	} else {
	console.log("Geolocation is not supported by this browser.");
	}
    let points = []
	$: points  = [
		{ lat: latitude, long: longitude },
	].map(p => projection([p.long, p.lat]))
	
	onMount(async () => {
		const us = await fetch('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json')
			.then(d => d.json()).catch(err => console.log(err))
		console.log({ us })
		
		
		const bad = ["Antarctica", "Fr. Polynesia", ]
		countries = topojson.feature(us, us.objects.countries).features.filter(x => !bad.includes(x.properties.name));

		mesh = topojson.mesh(us, us.objects.states, (a, b) => a !== b);
		
		console.log({ countries, mesh })
	})
	function getColor(name) {
    	return colors[iso2CodesByCountryName[name.toLowerCase()]] || 'white';
 	 }
</script>

<svg viewBox="-25 -100 {width} {height}">
	<!-- State shapes -->
	<g fill="white" stroke="black">
		{#each countries as feature, i}
            {#if iso2CodesByCountryName[feature.properties.name.toLowerCase()] in colors}
				<path d={path(feature)} on:click={() => selected = feature} style={`fill:${colors[iso2CodesByCountryName[feature.properties.name.toLowerCase()]]};`}  />
				<!-- <path
				d={path(feature)}
				on:click={() => selected = feature}
				class="state {selected === feature ? 'selected' : ''}"
				style="fill: {selected === feature ? 'green' : getColor(feature.properties.name)};"
				/> -->
				in:draw={{ delay: i * 50, duration: 1000 }} 
            {:else if markets.includes(iso2CodesByCountryName[feature.properties.name.toLowerCase()])}
				<path d={path(feature)} on:click={() => selected = feature} class="state"  />

			{:else}
            <path d={path(feature)}/>
            {/if}
		{/each}
	</g>
		
	<!-- Interior lines -->
<!-- 	<path d={path(mesh)} fill="none" stroke="black" /> -->
		
	{#if selected}
		<path d={path(selected)} fill="rgb(0,100,0)" stroke="black" stroke-width={2} />
	{/if}
		
	<!-- {#each countries as feature, i}
	  <path d={path(feature)} on:click={() => selected = feature} class="state" stroke="rgb(0 0 0 / 10%)" fill="none" />
	{/each}
	 
	{#each points as [cx, cy]}
		<circle {cx} {cy} r={10} fill="black" />
		<circle {cx} {cy} r={8} fill="white" />
		<circle {cx} {cy} r={5} fill="black" />
	{/each}  -->
</svg>

<!-- <div class="selectedName">{"Countries with most similar music tastes to: "+ selected?.properties.name ?? ''}</div> -->
	
<style>
	.state{
		fill: lightgreen;
		cursor: pointer;
	}
	.state:hover {
		fill: hsl(0 0% 50% / 20%);
	}
	
	.selectedName {
		text-align: center;
		margin-top: 8px;
		font-size: 1.5rem;
	}
	.tooltip {
		border: 1px solid #ddd;
		box-shadow: 1px 1px 1px #ddd;
		background: white;
		border-radius: 4px;
		padding: 4px;
		position: absolute;
	}
</style>