<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const players = ref([])
const input_name = ref('')
const input_gender = ref(null)
const groupings = ref([])
let round = ref(0)
const inboundGroupIndex = ref(null)
const inboundPlayerIndex = ref(null)
const inboundPlayer = ref(null)
const outboundPlayer = ref(null)

// eslint-disable-next-line vue/no-side-effects-in-computed-properties
const players_asc = computed(() => players.value.sort((a, b) => {
  return a.createdAt - b.createdAt
}))


watch(players, (newVal) => {
  localStorage.setItem('players', JSON.stringify(newVal))
}, {
  deep: true
})

const addPlayer = () => {
  if (input_name.value.trim() === '' || input_gender.value === null) {
    return
  }

  players.value.push({
    name: input_name.value,
    gender: input_gender.value,
    done: false,
    editable: false,
    createdAt: new Date().getTime()
  })

  // Clear variables
  input_name.value = null
  input_gender.value = null
}

const startDrag = (evt, player, groupIndex, playerIndex) => {
  console.log('startDrag')
  console.log('player = ' + JSON.stringify(player));
  console.log('groupIndex = ' + groupIndex + 'playerIndex = ' + playerIndex);
  evt.dataTransfer.dropEffect = 'move'
  evt.dataTransfer.effectAllowed = 'move'
  evt.dataTransfer.setData('player', player)
  inboundGroupIndex.value = groupIndex
  inboundPlayerIndex.value = playerIndex
  inboundPlayer.value = player
}

const onDrop = (evt, groupIndex, playerIndex) => {
  console.log('onDrop >> groupIndex = ' + groupIndex)

  // Get the player data and add to new grouing
  const movedPlayer = evt.dataTransfer.getData('player')
  console.log('movedPlayer = ' + JSON.stringify(movedPlayer));

  // Store the outbound player details
  outboundPlayer.value = groupings.value[groupIndex][playerIndex]

  // Add the new player to group, replacing the existing player
  groupings.value[groupIndex].splice(playerIndex, 1, JSON.parse(movedPlayer))

  // Remove the moved player from their original list and add the swapped player
  groupings.value[inboundGroupIndex.value].splice(inboundPlayerIndex.value, 1, outboundPlayer.value)

  //Reset the variables
  inboundGroupIndex.value = null
  inboundPlayerIndex.value = null
  inboundPlayer.value = null
  outboundPlayer.value = null

}

const removePlayer = (player) => {
  players.value = players.value.filter((t) => t !== player)
}

const deleteGroupings = () => {
  groupings.value = []
  round.value = 0
}

const makeGroupings = () => {
  const totalFoursomes = Math.floor(players.value.length / 4)
  const playersRemainder = players.value.length % 4
  console.log('totalFoursomes = ' + totalFoursomes)
  console.log(`playersRemainder = ${playersRemainder}`)

  // Increment the round
  round.value += 1
  console.log('now in round: ' + round.value);

  // Make a copy of players
  const tempPlayers = players.value

  // Shuffle copy of players
  for (let i = tempPlayers.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [tempPlayers[i], tempPlayers[j]] = [tempPlayers[j], tempPlayers[i]];
  }

  // Add a separator
  groupings.value.push([{ round: round.value }])

  // Divide into groups of 4 and add to 'groupings'
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

    <section class="create-player">
      <h2 class="title">Add Player</h2>

      <form id="new-player-form" @submit.prevent="addPlayer">
        <h4>What's their name?</h4>
        <input type="text" name="name" id="name" placeholder="e.g. John S." v-model="input_name" />

        <h4>Gender?</h4>
        <div class="options">

          <label>
            <input type="radio" name="gender" id="gender1" value="male" v-model="input_gender" />
            <span class="bubble male"></span>
            <div>Male</div>
          </label>

          <label>
            <input type="radio" name="gender" id="gender2" value="female" v-model="input_gender" />
            <span class="bubble female"></span>
            <div>Female</div>
          </label>

        </div>

        <input type="submit" value="Add player" />
      </form>
    </section>

    <section class="player-list">
      <h2 class="title">Players  ({{ players_asc.length }})</h2>
      <div class="list" id="player-list">

        <div v-if="players_asc.length < 1">
          <div class="player-item">
            <div class="player-name">No players yet</div>
          </div>
          
        </div>

        <div v-for="player in players_asc" :key="player.createdAt" :class="`player-item ${player.done && 'done'}`">

          <div class="player-name" :class="player.gender">
            {{ player.name }}
          </div>

          <div class="actions">
            <button class="delete" @click="removePlayer(player)"><i class="fi fi-rs-trash"></i></button>
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

    <section class="greeting" v-if="groupings.length > 0">
      <h2 class="title">Groupings</h2>
      <p class="instructions" v-if="groupings.length > 0">Touch and hold a name to drag it to another location to swap players</p>


      <div v-for="(grouping, i) in groupings" :key="i">

        <div v-if="grouping[0].round">
          <h3 class="subtitle">Round {{ grouping[0].round }}</h3>
        </div>

        <div v-else>

          <div class="grouping">

            <div class="player-pair">
              <div 
                v-if="grouping[0]" 
                class="player" 
                :class="grouping[0].gender == 'male' ? 'male' : 'female'"
                draggable="true"
                @dragstart="startDrag($event, JSON.stringify(grouping[0]), i, 0 )"
                @drop="onDrop($event, i, 0)"
                @dragover.prevent
                @dragenter.prevent
                >
                <div class="icon"><i class="fi fi-rr-menu-dots-vertical"></i></div>
                <div class="grouping-name">
                  {{ grouping[0].name }}
                </div>
                
              </div>

              <div 
                v-if="grouping[1]" 
                class="player" 
                :class="grouping[1].gender == 'male' ? 'male' : 'female'"
                draggable="true"
                @dragstart="startDrag($event, JSON.stringify(grouping[1]), i, 1)"
                @drop="onDrop($event, i, 1)"
                @dragover.prevent
                @dragenter.prevent
                >
                <div class="icon"><i class="fi fi-rr-menu-dots-vertical"></i></div>
                <div class="grouping-name">
                  {{ grouping[1].name }}
                </div>
                
              </div>

            </div>
            
            <div v-if="grouping.length == 4" class="vs">VS</div>

            <div class="player-pair">
              <div 
                v-if="grouping[2]" 
                class="player" 
                :class="grouping[2].gender == 'male' ? 'male' : 'female'"
                draggable="true"
                @dragstart="startDrag($event, JSON.stringify(grouping[2]), i, 2)"
                @drop="onDrop($event, i, 2)"
                @dragover.prevent
                @dragenter.prevent
                >
                <div class="icon"><i class="fi fi-rr-menu-dots-vertical"></i></div>
                <div class="grouping-name">
                  {{ grouping[2].name }}
                </div>
                
              </div>

              <div 
                v-if="grouping[3]" 
                class="player" 
                :class="grouping[3].gender == 'male' ? 'male' : 'female'"
                draggable="true"
                @dragstart="startDrag($event, JSON.stringify(grouping[3]), i, 3)"
                @drop="onDrop($event, i, 3)"
                @dragover.prevent
                @dragenter.prevent
                >
                <div class="icon"><i class="fi fi-rr-menu-dots-vertical"></i></div>
                <div class="grouping-name">
                  {{ grouping[3].name }}
                </div>
              </div>

            </div>
          </div>
        </div>


      </div>

      <button v-if="groupings.length > 0" class="delete-groupings-button" @click="deleteGroupings">Delete
        groupings</button>
    </section>

  </main>
</template>
