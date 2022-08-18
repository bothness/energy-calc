<script>
	import { onMount } from "svelte";
	import House from "./lib/House.svelte";
	import Slider from "./lib/Slider.svelte";

	const cost = 800; // Supplier cost of meeting energy 
	let budget = [1000,2000,3000,4000,5000]; // Energy budget for each income quintile

	let params = {
		supply: 80, // supply as % of demand
		price: 3000, // price of energy per customer
		ration: 100, // limit placed on customers, as % of demand
		received: [null, null, null, null, null], // % of demand received by each quintile
		gross: null, // Gross income made by supplier
		cost: null, // Total cost to supplier
		profit: null, // Supplier income after costs
		surplus: null // Energy left over after the amount that customers can afford
	};

	function calc() {
		let available = 5 * 100 * (params.supply / 100); // Total energy available for all 5 quintiles, relative to 5 * 100% normal demand
		let used = 0;
		let received = [];
		budget.forEach(b => {
			let rec;
			if (params.price * (params.ration / 100) > b) {
				rec = 100 * (b / params.price);
			} else {
				rec = params.ration;
			}
			received.push(rec);
			used += rec;
		});
		if (used > available) {
			params.received = [null, null, null, null, null];
			params.gross = null;
			params.cost = null;
			params.profit = null; 
			params.surplus = null;
		} else {
			params.received = received;
			params.gross = (used / 100) * params.price;
			params.cost = (used / 100) * cost;
			params.profit = params.gross - params.cost;
			params.surplus = (available - used) / 5;
		}
		console.log(received)
	}

	onMount(calc);
</script>

<main>
	<h1>Energy market calculator</h1>

	<p>This tool is designed to illustrate a highly simplified energy market, and what happens when supply isn't capable of meeting demand. It is based on a single energy supplier*, supplying five households, each with a different income level, with annual budgets for energy ranging from £1,000 to £5,000.</p>

	<p>When supply is below 100% of the normal consumption level, you have a choice to balance supply and demand either by: (1) increasing the prices until some households are forced to use less; (2) rationing the supply to all households; or (3) through a combination of both measures.</p>

	<p><small>*The "energy supplier" represents the extraction, generation and transmission of all types of energy supplied to homes.</small></p>

	<section>
		<h2>Select your market conditions <span>Values are relative to normal annual consumption (demand) and price levels</span></h2>

		<!-- svelte-ignore a11y-label-has-associated-control -->
		<label>
			Supply: <strong>{params.supply}%</strong> of normal consumption
			<Slider
				name="supply"
				bind:value={params.supply}
				min={0} max={100} step={1}
				on:input={calc}/>
		</label>
		<!-- svelte-ignore a11y-label-has-associated-control -->
		<label>
			Energy price: <strong>£{params.price.toLocaleString()}</strong><br/>
			<small>(Note: Cost to the supplier to meet normal consumption is assumed to be fixed at £800 per household)</small>
			<Slider
				name="price"
				bind:value={params.price}
				min={800} max={5000} step={10}
				on:input={calc}/>
		</label>
		<!-- svelte-ignore a11y-label-has-associated-control -->
		<label>
			Ration: <strong>{params.ration}%</strong> of normal consumption
			<Slider
				name="ration"
				bind:value={params.ration}
				min={10} max={100} step={1}
				on:input={calc}/>
		</label>
		<!-- <label>
			Household energy budgets, poorest to richest
		</label>
		<div class="table-container">
			<table>
				<tr style:white-space="nowrap">
					{#each budget as val, i}
					<td>£<input type="number" bind:value={val} min={i == 0 ? 0 : budget[i - 1]} max={i == budget.length - 1 ? 10000 : budget[i + 1]} on:change={calc}/></td>
					{/each}
				</tr>
			</table>
		</div> -->
	</section>

	<section>
		{#if Array.isArray(params.received) && params.gross != null && params.cost != null && params.profit != null && params.surplus != null}
		<h2>Household energy received as % of normal consumption</h2>
		{:else}
		<h2>Sorry. Supply won't balance demand under these conditions. Expect disruption!</h2>
		{/if}
		<div class="table-container">
			<table>
				<tr>
					{#each params.received as val, i}
					<td style:text-align="{i == 0 ? 'left' : 'right'}">
						{@html i == 0 ? '&larr; Poorest' : i == params.received.length - 1 ? 'Richest &rarr;' : ''}
					</td>
					{/each}
				</tr>
				<tr>
					{#each params.received as val}
					<td><House percent={val}/></td>
					{/each}
				</tr>
				<tr>
					{#each budget as val, i}
					<td>
						<table class="data">
							<tr>
								<td>Budget:</td>
								<td>£{val.toLocaleString()}</td>
							</tr>
							<tr>
								<td>Spend:</td>
								<td><strong>£{Math.round(params.received[i] * params.price * 0.01).toLocaleString()}</strong></td>
							</tr>
						</table>
					</td>
					{/each}
				</tr>
			</table>
		</div>
	</section>

	<section>
		<h2>Energy supplier income, costs and profits</h2>
		<table class="data">
			<tr>
				<td>Gross income:</td>
				<td>{@html params.gross != null ? `<strong>£${Math.round(params.gross).toLocaleString()}</strong>` : 'N/A'}</td>
			</tr>
			<tr>
				<td>Costs:</td>
				<td>{@html params.cost != null ? `<strong>£${Math.round(params.cost).toLocaleString()}</strong>` : 'N/A'}</td>
			</tr>
			<tr>
				<td>Profit:</td>
				<td>{@html params.profit != null ? `<strong>£${Math.round(params.profit).toLocaleString()}</strong> (<strong>${Math.round((params.profit / params.gross) * 100)}%</strong> of gross income)` : 'N/A'}</td>
			</tr>
			<tr>
				<td>Surplus:</td>
				<td>{@html params.surplus != null ? `<strong>${Math.round(params.surplus)}%</strong> of available supply` : 'N/A'}</td>
			</tr>
		</table>
	</section>

	<p>This calculator was coded by <a href="https://twitter.com/bothness" target="_blank">Ahmad Barclay</a>. You can find the <a href="https://github.com/bothness/energy-calc/" target="_blank">source code here</a>.</p>
</main>

<style>
	:global(*) {
		box-sizing: border-box;
	}
	main {
		width: 680px;
		max-width: calc(100% - 24px);
		margin: 0 auto;
	}
	section {
		background-color: #eee;
		padding: 8px 12px;
		margin-bottom: 18px;
		border-radius: 6px;
	}
	section h2 {
		font-size: 1.2em;
		margin: 0 0 8px;
	}
	section h2 > span {
		display: block;
		font-size: 1rem;
		font-weight: normal;
		color: #666;
		margin-bottom: 20px;
	}
	small {
		color: #666;
		size: 0.9rem;
	}
	label {
		display: block;
	}
	label input {
		display: block;
	}
	input[type=number] {
		display: inline;
		width: 100px;
	}
	input[type=range] {
		width: 100%;
	}
	.table-container {
		overflow-x: auto;
	}
	.table-container table {
		border-collapse: collapse;
		width: 100%;
	}
	.table-container table td + td {
		border-left: 12px solid rgba(255,255,255,0);
	}
	table.data {
		margin-bottom: 12px;
	}
	table.data td + td {
		border-left: 4px solid rgba(255,255,255,0);
	}
</style>