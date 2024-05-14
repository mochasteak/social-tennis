<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const players = ref([])
const input_content = ref('')
const input_category = ref(null)
const groupings = ref([])

// eslint-disable-next-line vue/no-side-effects-in-computed-properties
const players_asc = computed(() => players.value.sort((a,b) =>{
	return a.createdAt - b.createdAt
}))


watch(players, (newVal) => {
	localStorage.setItem('players', JSON.stringify(newVal))
}, {
	deep: true
})

const addTodo = () => {
	if (input_content.value.trim() === '' || input_category.value === null) {
		return
	}

	players.value.push({
		content: input_content.value,
		category: input_category.value,
		done: false,
		editable: false,
		createdAt: new Date().getTime()
	})
}

const removeTodo = (todo) => {
	players.value = players.value.filter((t) => t !== todo)
}

const deleteGroupings = () => {
  groupings.value = []
}

const makeGroupings = () => {
  const totalFoursomes = Math.floor(players.value.length / 4)
  const playersRemainder = players.value.length % 4
  console.log('totalFoursomes = ' + totalFoursomes)
  console.log(`playersRemainder = ${playersRemainder}`)

  // Make a copy of players
  const tempPlayers = players.value

  // Shuffle copy of players
  for (let i = tempPlayers.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [tempPlayers[i], tempPlayers[j]] = [tempPlayers[j], tempPlayers[i]];
  }

  // Divide into groups of 4
  for (let i = 0; i < tempPlayers.length; i += 4) {
    groupings.value.push(tempPlayers.slice(i, i + 4));
  }

}

onMounted(() => {
	players.value = JSON.parse(localStorage.getItem('players')) || []
})
</script>

<template>
	<main class="app">
		
		<section class="greeting">
			<h1 class="headline">
				Social Tennis Generator
			</h1>
		</section>

		<section class="create-todo">
			<h2 class="title">Add Player</h2>

			<form id="new-todo-form" @submit.prevent="addTodo">
				<h4>What's their name?</h4>
				<input 
					type="text" 
					name="content" 
					id="content" 
					placeholder="e.g. John S."
					v-model="input_content" />
				
				<h4>Gender?</h4>
				<div class="options">

					<label>
						<input 
							type="radio" 
							name="gender" 
							id="category1" 
							value="male"
							v-model="input_category" />
						<span class="bubble male"></span>
						<div>Male</div>
					</label>

					<label>
						<input 
							type="radio" 
							name="gender" 
							id="category2" 
							value="female"
							v-model="input_category" />
						<span class="bubble female"></span>
						<div>Female</div>
					</label>

				</div>

				<input type="submit" value="Add player" />
			</form>
		</section>

		<section class="todo-list">
			<h2 class="title">Players</h2>
			<div class="list" id="todo-list">

				<div v-for="todo in players_asc" :key="todo.content" :class="`todo-item ${todo.done && 'done'}`">

					<div class="todo-content" :class="todo.category">
						{{todo.content}}
					</div>

					<div class="actions">
						<button class="delete" @click="removeTodo(todo)">Delete</button>
					</div>
				</div>

			</div>
		</section>

    <section class="greeting">
      <div>
        <h2 class="title">Make groupings</h2>
      </div>
    </section>
    <section>
      <button class="make-groupings-button" @click="makeGroupings">Make groupings</button>
    </section>

    <section class="greeting"  v-if="groupings.length > 0">
      <h2 class="title">Groupings</h2>
      <div v-for="(grouping, i) in groupings" :key="i">
        <div class="grouping">
          <div class="player-pair">
            <div v-if="grouping[0]" class="player" :class="grouping[0].category == 'male' ? 'male' : 'female'">{{grouping[0].content}}</div>
           <div v-if="grouping[1]" class="player" :class="grouping[1].category == 'male' ? 'male' : 'female'">{{grouping[1].content}}</div>
          </div>

        <div v-if="grouping.length == 4" class="vs">VS</div>

        <div>
          <div v-if="grouping[2]" class="player" :class="grouping[2].category == 'male' ? 'male' : 'female'">{{grouping[2].content}}</div>
        <div v-if="grouping[3]" class="player" :class="grouping[3].category == 'male' ? 'male' : 'female'">{{grouping[3].content}}</div>
        </div>

        <!-- <div v-for="(player, i) in grouping" :key="i">
          <div class="player" :class="player.category == 'male' ? 'male' : 'female' ">{{ player.content }}</div>
        </div> -->
      </div>
      </div>

      <button v-if="groupings.length > 0" class="delete-groupings-button" @click="deleteGroupings">Delete groupings</button>
    </section>


    <p>players = {{ players }}</p>
    <br>
    <p>groupings = {{ groupings }}</p>
    

	</main>
</template>
