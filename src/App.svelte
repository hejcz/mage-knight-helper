<script>
	let name = 'world';
	
	const STATE_TACTIC = 0
	const STATE_ROUND = 1
	const STATE_UPDATE_DUMMY = 2
	
	let current_state = STATE_TACTIC
	
	const tactics_names = {
		day: [
			"Ranny ptaszek",
			"Analiza",
			"Kradzie| many",
			"Planowanie",
			"Wielki start",
			"Odpowiedni moment"
		],
		night: [
			"Od zmierzchu",
			"DBuga noc",
			"Poszukiwanie many",
			"Medytacja o póBnocy",
			"Przygotowanie",
			"Moc gromadzenia"
		]
	}
	
	// [r,g,b,w]
	let state = {
		dummy: {
			cards: [4,4,4,4],
			crystals: [1,0,2,0],
			discard: [0,0,0,0],
			last_turn: false
		},
		tactics: {
			day: [false, false, false, false, false, false],
			night: [false, false, false, false, false, false]
		},
		player_tactic: null,
		dummy_tactic: null,
		reputation: 0,
		fame: 0,
		day: true
	}
	
	$: left_cards = state.dummy.cards.reduce((acc, x, index) => acc + x - state.dummy.discard[index], 0)
	$: cards_to_draw_from = state.dummy.cards.reduce((acc, x, index) => acc.concat(Array(x - state.dummy.discard[index]).fill(index)), [])
	$: last_turn = cards_to_draw_from.length === 0
	
	let crystal_color = undefined
	let card_color = undefined
	let tactic = undefined
	
	function index_to_color(index) {
		switch (index) {
			case 0:
				return 'RED'
			case 1:
				return 'GREEN'
			case 2:
				return 'BLUE'
			case 3:
				return 'WHITE'
		}
	}
	
	function index_to_css_color(index) {
		switch (index) {
			case 0:
				return 'red'
			case 1:
				return 'green'
			case 2:
				return 'blue'
			case 3:
				return 'lightgrey'
		}
	}
	
	function update_dummy() {
		if (crystal_color == null || card_color == null) {
			return;
		}
		state.dummy.crystals[crystal_color]++;
		state.dummy.cards[card_color]++;
		state.dummy = state.dummy
		crystal_color = undefined
		card_color = undefined
		
		current_state = STATE_TACTIC;
	}
	
	function select_tactic() {
		if (tactic == null) {
			return;
		}
		//// tactics
		// player
		const tactics = state.day ? state.tactics.day : state.tactics.night
		tactics[tactic] = true
		state.player_tactic = tactic
		// dummy
		const shufflable_tactics = [...tactics].map((x, index) => x ? null : index)
		shuffle(shufflable_tactics)
		tactics[shufflable_tactics[0]] = true
		state.dummy_tactic = shufflable_tactics[0]
		tactic = undefined
		
		current_state = STATE_ROUND;
	}
	
	function dummy_proceed() {
		if (state.dummy.last_turn) {
			state.dummy.discard = [0,0,0,0],
			state.dummy.last_turn = false
			state.day = !state.day
			current_state = STATE_UPDATE_DUMMY;
			return
		}
		
		if (cards_to_draw_from.length === 0) {
			state.dummy.last_turn = true
			return
		}
		
		const shuffled = [...cards_to_draw_from]
		shuffle(shuffled)
		
		let toRemove = 0
		if (cards_to_draw_from.length < 3) {
			toRemove = cards_to_draw_from.length
		} else {
			toRemove = Math.min(3 + state.dummy.crystals[shuffled[2]], cards_to_draw_from.length)
		}
		for (let i = 0; i < toRemove; ++i) {
			const removed = shuffled[i] // index
			console.log(index_to_color(removed))
			state.dummy.discard[removed]++;
		}
		state.dummy = state.dummy
	}
	
	function shuffle(array) {
		let currentIndex = array.length,  randomIndex;

		// While there remain elements to shuffle.
		while (currentIndex != 0) {

			// Pick a remaining element.
			randomIndex = Math.floor(Math.random() * currentIndex);
			currentIndex--;

			// And swap it with the current element.
			[array[currentIndex], array[randomIndex]] = [
				array[randomIndex], array[currentIndex]];
		}

		return array;
	}
</script>

<style>
@import url('https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css');
</style>

<h1>Dummy player</h1>
<p>
	Cards: 
	{#each state.dummy.cards as cards, index}
		{#each Array(cards) as _, i}
				<i class="fa fa-file" aria-hidden="true" style:margin-right=5px style:color={index_to_css_color(index)}></i>
		{/each}
	{/each}
</p>
<p>
	Crystals:
	{#each state.dummy.crystals as crystals, index}
		{#each Array(crystals) as _, i}
				<i class="fa fa-diamond" aria-hidden="true" style:margin-right=5px style:color={index_to_css_color(index)}></i>
		{/each}
	{/each}
</p>
{#if state.player_tactic != null}
<p>
	Player's tactic: {(state.day ? tactics_names.day : tactics_names.night)[state.player_tactic]}
</p>
<p>
	Dummy's tactic: {(state.day ? tactics_names.day : tactics_names.night)[state.dummy_tactic]}
</p>
{/if}

{#if current_state === STATE_TACTIC}
<p>
		Chosen tactic (from remaining tactic cards):
		<select bind:value={tactic}>
		{#each state.day ? state.tactics.day : state.tactics.night as taken, index}
			{#if !taken}
			<option value={index}>
				{(state.day ? tactics_names.day : tactics_names.night)[index]}
			</option>
			{/if}
		{/each}
	</select>
</p>
<button on:click={select_tactic}>
	Select tactic
</button>
{/if}

{#if current_state === STATE_ROUND}
<p>
{#if state.dummy.last_turn}
	Last turn of the round begins
{:else if last_turn}
	End of the round will be announced next turn
{:else}
	Cards left: {left_cards}
{/if}
</p>
<button on:click={dummy_proceed}>
	Proceed
</button>
{/if}

{#if current_state === STATE_UPDATE_DUMMY}
<p>
		Update dummy player
	</p>

	<p>
		Crystal (from remaining spell):
		<select bind:value={crystal_color}>
		{#each [0, 1, 2, 3] as color_index}
		<option value={color_index}>
			{index_to_color(color_index)}
		</option>
		{/each}
	</select>
	</p>

	<p>
		Card (from remaining advanced card):
		<select bind:value={card_color}>
		{#each [0, 1, 2, 3] as color_index}
		<option value={color_index}>
			{index_to_color(color_index)}
		</option>
		{/each}
	</select>
	</p>

	

	<button on:click={update_dummy}>
		Update
	</button>
{/if}
