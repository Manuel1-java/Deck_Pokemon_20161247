<template>
  <div class="container py-5 text-center">
    <h1 class="mb-4 text-white">Generador de Mazo TCG</h1>
    
    <div class="controls mb-5" style="height: 60px;">
      <button v-if="phase === 'IDLE' || phase === 'REVEALED'" @click="handleAction" class="btn btn-warning btn-lg fw-bold shadow">
        {{ phase === 'IDLE' ? 'Generar Mazo de 40 Cartas' : 'Generar Nuevo Mazo' }}
      </button>
      <button v-else-if="phase === 'STACKED'" @click="handleAction" class="btn btn-info text-white btn-lg fw-bold shadow">
        Expandir Mazo
      </button>
      <button v-else-if="phase === 'EXPANDED'" @click="handleAction" class="btn btn-success btn-lg fw-bold shadow">
        Revelar Cartas
      </button>
      <button v-else class="btn btn-secondary btn-lg fw-bold shadow" disabled>
        {{ phase === 'FETCHING' ? 'Consultando PokéAPI...' : 'Animando...' }}
      </button>
    </div>

    <TransitionGroup name="card-anim" tag="div" :class="['table-zone', phase]">
      <div v-for="(card, index) in visibleCards" :key="card.id + '-' + index" 
           class="card-wrapper" 
           :style="{'--stack-index': index}">
        
        <div class="flip-container" :class="{ 'is-revealed': phase === 'REVEALED' }">
          <div class="flipper">
            <div class="front">
              <PokemonCard :pokemon="card" />
            </div>
            <div class="back"></div>
          </div>
        </div>

      </div>
    </TransitionGroup>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
import PokemonCard from './components/PokemonCard.vue';

const deck = ref([]);
const visibleCards = ref([]);
const phase = ref('IDLE');

const typeRelations = {
  normal: { weak: 'fighting', resist: 'ghost' }, fire: { weak: 'water', resist: '' },
  water: { weak: 'electric', resist: '' }, grass: { weak: 'fire', resist: 'water' },
  electric: { weak: 'fighting', resist: 'steel' }, ice: { weak: 'fire', resist: '' },
  fighting: { weak: 'psychic', resist: '' }, poison: { weak: 'psychic', resist: '' },
  ground: { weak: 'grass', resist: 'electric' }, flying: { weak: 'electric', resist: 'fighting' },
  psychic: { weak: 'dark', resist: 'fighting' }, bug: { weak: 'fire', resist: 'grass' },
  rock: { weak: 'water', resist: '' }, ghost: { weak: 'dark', resist: 'normal' },
  dragon: { weak: 'fairy', resist: '' }, dark: { weak: 'fighting', resist: 'psychic' },
  steel: { weak: 'fire', resist: 'grass' }, fairy: { weak: 'steel', resist: 'dark' }
};

const delay = ms => new Promise(res => setTimeout(res, ms));

const fetchDeckData = async () => {
  const randomIds = Array.from({ length: 40 }, () => Math.floor(Math.random() * 1025) + 1);
  const deckData = await Promise.all(randomIds.map(async (id) => {
    const [pokeRes, speciesRes] = await Promise.all([
      axios.get(`https://pokeapi.co/api/v2/pokemon/${id}`),
      axios.get(`https://pokeapi.co/api/v2/pokemon-species/${id}`)
    ]);
    const p = pokeRes.data;
    const species = speciesRes.data;
    
    const selectedMoves = p.moves.slice(0, 4);
    const movesData = await Promise.all(selectedMoves.map(async (m) => {
      try {
        const moveDetails = await axios.get(m.move.url);
        return { name: m.move.name.replace('-', ' '), power: moveDetails.data.power || '', cost: Math.floor(Math.random() * 3) + 1 };
      } catch (e) { return { name: m.move.name.replace('-', ' '), power: '', cost: 1 }; }
    }));

    const primaryType = p.types[0].type.name;
    const flavorTextEntry = species.flavor_text_entries.find(f => f.language.name === 'es') || species.flavor_text_entries.find(f => f.language.name === 'en');
    
    return {
      id: p.id, name: p.name, stage: species.evolves_from_species ? 'Fase 1' : 'Básico',
      image: p.sprites.other['official-artwork'].front_default || p.sprites.front_default,
      types: p.types.map(t => t.type.name), primaryType, hp: p.stats.find(s => s.stat.name === 'hp').base_stat,
      weight: p.weight / 10, height: p.height / 10, moves: movesData,
      weakness: typeRelations[primaryType]?.weak || '', resistance: typeRelations[primaryType]?.resist || '',
      retreat: Math.min(Math.max(Math.ceil(p.weight / 500), 1), 4),
      flavor: flavorTextEntry ? flavorTextEntry.flavor_text.replace(/\f/g, ' ') : 'Un Pokémon misterioso.',
      rarity: '★', illustrator: 'Ken Sugimori'
    };
  }));
  return deckData;
};

const handleAction = async () => {
  if (phase.value === 'IDLE' || phase.value === 'REVEALED') {
    if (phase.value === 'REVEALED') {
      phase.value = 'EXPANDED';
      await delay(600);
      phase.value = 'STACKED';
      await delay(600);
      phase.value = 'REVERSING';
      while (visibleCards.value.length > 0) {
        visibleCards.value.pop();
        await delay(30);
      }
      deck.value = [];
    }

    phase.value = 'FETCHING';
    deck.value = await fetchDeckData();
    
    phase.value = 'STACKING';
    for (let i = 0; i < deck.value.length; i++) {
      visibleCards.value.push(deck.value[i]);
      await delay(35);
    }
    phase.value = 'STACKED';

  } else if (phase.value === 'STACKED') {
    phase.value = 'EXPANDED';
  } else if (phase.value === 'EXPANDED') {
    phase.value = 'REVEALED';
  }
};
</script>

<style>
body { background-color: #2c3e50; overflow-x: hidden; }

.table-zone {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 15px;
  min-height: 60vh;
  perspective: 1200px;
}

.table-zone.STACKING, .table-zone.STACKED, .table-zone.REVERSING {
  gap: 0;
  align-items: center;
}
.table-zone.STACKING .card-wrapper,
.table-zone.STACKED .card-wrapper,
.table-zone.REVERSING .card-wrapper {
  margin-left: -278px;
  margin-top: calc(var(--stack-index) * -0.5px);
  pointer-events: none;
}
.table-zone.STACKING .card-wrapper:first-child,
.table-zone.STACKED .card-wrapper:first-child,
.table-zone.REVERSING .card-wrapper:first-child {
  margin-left: 0;
}

.card-wrapper {
  width: 280px;
  height: 392px;
  transition: margin 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}

.card-anim-enter-active, .card-anim-leave-active {
  transition: all 0.2s ease;
}
.card-anim-enter-from, .card-anim-leave-to {
  opacity: 0;
  transform: translateY(-50px) scale(0.9);
}

.flip-container {
  width: 100%;
  height: 100%;
}
.flipper {
  width: 100%;
  height: 100%;
  position: relative;
  transition: transform 0.6s cubic-bezier(0.4, 0.2, 0.2, 1);
  transform-style: preserve-3d;
}
.is-revealed .flipper {
  transform: rotateY(180deg);
}
.front, .back {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  backface-visibility: hidden;
  border-radius: 12px;
  box-shadow: -3px 4px 10px rgba(0,0,0,0.4);
}
.front {
  transform: rotateY(180deg);
}
.back {
  transform: rotateY(0deg);
  background-image: url('/pokemon_back.jpg');
  background-size: cover;
  background-position: center;
  border: 10px solid #f2da5e;
}
</style>