<script lang="ts">
	interface Women {
		[name: string]: number
	}
	interface Woman {
		name: string
		age: number
	}
	let women: Women = null

	const feys = [
		"Maya Fey", "Pearl Fey", "Ami Fey", "Iris Hawthorne", "Iris", "Dahlia Hawthorne", "Morgan Fey", "Misty Fey", "Bikini"
	]
	const generatePairing = (year: number): [Woman, Woman] => {
		const candidates = Object.entries(women).filter(([_, birthYear]) => {
			return (year - birthYear) >= 16 
		})
		let candidateIndex = Math.floor(Math.random() * candidates.length)
		const [ name, birthYear ] = candidates.splice(candidateIndex, 1)[0]
		const age = year - birthYear

		const upperBound = 2 * (age - 7)
		const lowerBound = age / 2 - 7

		const lastName = name.split(" ").slice(-1)[0]
		const filteredCandidates = candidates.filter(
			([candidateName, candidateBirthYear]) => {
				let candidateLastName = name.split(" ").slice(-1)[0]
				let candidateAge = year - candidateBirthYear
				const cUpperBound = 2 * (candidateAge - 7)
				const cLowerBound = candidateAge / 2 - 7
				return (
					(candidateLastName != lastName) &&
					!(feys.includes(name) && feys.includes(candidateName)) &&
					age >= cLowerBound && age <= cUpperBound && candidateAge >= lowerBound && candidateAge <= upperBound &&
					(candidateAge < 18 && age < 18)||(candidateAge >= 18 && age >= 18)
				)
			}
		)
		candidateIndex = Math.floor(Math.random() * filteredCandidates.length)
		const paired = filteredCandidates[candidateIndex]
		return [{ name, age }, { name: paired[0], age: year - paired[1]}]
	}

	let ship: [Woman, Woman]

	let year = 2016

	$: {
		if(year < 2016) {
			year = 2016
		}
		if(year > 2035) {
			year = 2035
		}
	}

	const importWomen = async () => {
		women = await (await fetch("women.json")).json()
		ship = generatePairing(year)
	}

	let showList = false
	let sortKey
	let sortAsc = true
	$: sortMult = sortAsc ? 1 : -1
	$: showText = !showList ? "Show list of women" : "Collapse list of women"

</script>

<main>
	<h1>THE FEMSLASH GENERATOR IS BORN ANEW</h1>
	{#await importWomen()}
		<h2>Loading women...</h2>
	{:then}
		<h3>YOUR RANDOM SHIP IS</h3>
		<h2>{ship[0].name} (age {ship[0].age}) and {ship[1].name} (age {ship[1].age})</h2>
		<label for="refYear">Reference year</label><input type="number" min=2016 max=2035 bind:value={year} id="refYear" name="refYear"><br/>
		<button on:click={() => ship = generatePairing(year)}>Generate anew</button>
		<button on:click={() => showList = !showList}>{showText}</button>
		<div class="dropdown {showList ? "uncollapsed" : "collapse"}">
			<label for="sortBy">Sort by:</label>
			<select id="sortBy" name="sortBy" bind:value={sortKey}>
				<option value="name">Name</option>
				<option value="age">Age</option>
			</select>
			<label for="sortAsc">Ascending</label>
			<input type="checkbox" id="sortAsc" name="sortAsc" bind:checked={sortAsc}>
			<ul>
			{#each Object.entries(women).sort(
				([n1, a1], [n2, a2]) => {
					if(sortKey == "age") return sortMult * (a1 - a2)
					return sortMult * n1.localeCompare(n2)
				}
			) as [name, birthYear]}
				<li>{name} (born {birthYear})</li>
			{/each}
			</ul>
		</div>
	{:catch}
		<h2>Women are DEAD</h2>
	{/await}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	.dropdown {
		width: 85%;
		height: 50%;
		max-height: 50vh;
		overflow-y: scroll;
		margin: auto;
		text-align: center;
		transition: opacity 250ms;
	}
	.dropdown label {
		display: inline;
	}
	ul {
		list-style-type: none;
	}
	.collapse {
		display: none;
	}
	.uncollapsed {
		display: inherit;
	}

	h1 {
		color: #ff3e00;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>