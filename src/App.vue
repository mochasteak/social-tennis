<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import { useToast } from 'vue-toastification'
import { VueToggles } from 'vue-toggles'

const toast = useToast()

const players = ref([])
const input_name = ref('')
const isMale = ref(true)
const groupings = ref([])
let round = ref(0)
const inboundGroupIndex = ref(null)
const inboundPlayerIndex = ref(null)
const inboundPlayer = ref(null)
const outboundPlayer = ref(null)
const anchor = ref(null)

// eslint-disable-next-line vue/no-side-effects-in-computed-properties
const players_asc = computed(() => players.value.sort((a, b) => {
  return a.createdAt - b.createdAt
}))


watch(players, (newVal) => {
  localStorage.setItem('players', JSON.stringify(newVal))
}, {
  deep: true
})

watch(groupings, (newVal) => {
  console.log(`watch >> newVal = ${JSON.stringify(newVal)}`)
  // Scroll to the top of the 'groupings' list
  console.log(`watch >> anchor = ${anchor.value}`);
  //const lastGrouping = anchor.value.lastElementChild
  //console.log(`lastGrouping = ${lastGrouping}`);
  //lastGrouping?.scrollIntoView({behavior: "smooth", block: "start"})
}, {deep: true})

const addPlayer = () => {

  console.log('addPlayer invoked');

  // Validate and handle errors
  if (input_name.value.trim() === '' || isMale.value === null) {
    toast.error('Name and gender are both required')
    console.log('addPlayer >> display error toast');
    return
  }


  // Create player object and add to players list
  players.value.push({
    name: input_name.value,
    gender: isMale.value ? 'male' : 'female',
    done: false,
    editable: false,
    createdAt: new Date().getTime()
  })

  // Success toast
  toast.success(`${input_name.value} added to players list`)

  // Clear variables
  input_name.value = null
  isMale.value = 'male'
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

  // Get the player data
  const movedPlayer = evt.dataTransfer.getData('player')
  console.log('movedPlayer = ' + JSON.stringify(movedPlayer));

  // Store the outbound player details
  outboundPlayer.value = groupings.value[groupIndex][playerIndex]

  // Replace the outbound player with the inbound player
  groupings.value[groupIndex].splice(playerIndex, 1, JSON.parse(movedPlayer))

  // Remove the inbound player from their original list and add the outbound player
  groupings.value[inboundGroupIndex.value].splice(inboundPlayerIndex.value, 1, outboundPlayer.value)

  //Reset the variables
  inboundGroupIndex.value = null
  inboundPlayerIndex.value = null
  inboundPlayer.value = null
  outboundPlayer.value = null

}

const removePlayer = (player) => {
  players.value = players.value.filter((t) => t !== player)
  toast.success(`${player.name} removed`)
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

  // Notify user
  toast.success(`Groupings for Round ${round.value} created. (scroll down)`)

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
      <h2 class="title">1. Add Players</h2>

      <form id="new-player-form" @submit.prevent="addPlayer">
        

        <div class="form-inputs">
          <div>
            <input type="text" name="name" id="name" placeholder="Type name here" v-model="input_name" />

          </div>
          <div>
            <VueToggles 
              v-model="isMale" 
              :height="50" 
              :width="120" 
              checkedText="Male" 
              uncheckedText="Female"
              checkedBg="#3A82EE" 
              uncheckedBg="#EA40A4" 
              fontSize="16"
            />
          </div>

        </div>

        <input type="submit" value="Add player" />
      </form>
    </section>

    <section class="player-list">
      <h2 class="title">Players ({{ players_asc.length }})</h2>
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
        <h2 class="title">2. Make groupings</h2>
      </div>
    </section>
    <section>
      <button class="make-groupings-button" @click="makeGroupings">Make groupings</button>
    </section>

    <section class="greeting">
      <h2 class="title" v-if="groupings.length > 0">Groupings</h2>
      <p class="instructions" v-if="groupings.length > 0">Touch and hold a name to drag it to another location to swap
        players</p>


      <div v-for="(grouping, i) in groupings" :key="i" ref="anchor">

        <div v-if="grouping[0].round">
          <h3 class="subtitle">Round {{ grouping[0].round }}</h3>
        </div>

        <div v-else>

          <div class="grouping">

            <div class="player-pair">
              <div v-if="grouping[0]" class="player" :class="grouping[0].gender == 'male' ? 'male' : 'female'"
                draggable="true" @dragstart="startDrag($event, JSON.stringify(grouping[0]), i, 0)"
                @drop="onDrop($event, i, 0)" @dragover.prevent @dragenter.prevent>
                <div class="icon"><i class="fi fi-rr-menu-dots-vertical"></i></div>
                <div class="grouping-name">
                  {{ grouping[0].name }}
                </div>

              </div>

              <div v-if="grouping[1]" class="player" :class="grouping[1].gender == 'male' ? 'male' : 'female'"
                draggable="true" @dragstart="startDrag($event, JSON.stringify(grouping[1]), i, 1)"
                @drop="onDrop($event, i, 1)" @dragover.prevent @dragenter.prevent>
                <div class="icon"><i class="fi fi-rr-menu-dots-vertical"></i></div>
                <div class="grouping-name">
                  {{ grouping[1].name }}
                </div>

              </div>

            </div>

            <div v-if="grouping.length == 4" class="vs">VS</div>

            <div class="player-pair">
              <div v-if="grouping[2]" class="player" :class="grouping[2].gender == 'male' ? 'male' : 'female'"
                draggable="true" @dragstart="startDrag($event, JSON.stringify(grouping[2]), i, 2)"
                @drop="onDrop($event, i, 2)" @dragover.prevent @dragenter.prevent>
                <div class="icon"><i class="fi fi-rr-menu-dots-vertical"></i></div>
                <div class="grouping-name">
                  {{ grouping[2].name }}
                </div>

              </div>

              <div v-if="grouping[3]" class="player" :class="grouping[3].gender == 'male' ? 'male' : 'female'"
                draggable="true" @dragstart="startDrag($event, JSON.stringify(grouping[3]), i, 3)"
                @drop="onDrop($event, i, 3)" @dragover.prevent @dragenter.prevent>
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
