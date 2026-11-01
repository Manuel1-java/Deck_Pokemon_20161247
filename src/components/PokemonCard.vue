<template>
  <div :class="['tcg-card', pokemon.primaryType]">
    <div class="card-inner">
      
      <div class="d-flex justify-content-between align-items-center mb-1">
        <div>
          <span class="stage-badge">{{ pokemon.stage }}</span>
          <h5 class="text-capitalize fw-bold mb-0 ms-1 d-inline">{{ pokemon.name }}</h5>
        </div>
        <div class="d-flex align-items-center">
          <span class="hp-text text-danger fw-bold me-1">HP {{ pokemon.hp }}</span>
          <span :class="['type-icon', pokemon.primaryType]"></span>
        </div>
      </div>

      <div class="image-frame bg-light d-flex justify-content-center align-items-center shadow-sm">
        <img :src="pokemon.image" class="img-fluid pokemon-img" :alt="pokemon.name">
      </div>

      <div class="stats-bar text-center my-1 fst-italic">
        NO. {{ pokemon.id }} &nbsp; HT: {{ pokemon.height }}m &nbsp; WT: {{ pokemon.weight }}kg
      </div>

      <div class="moves-container flex-grow-1">
        <div v-for="move in pokemon.moves" :key="move.name" class="move-row border-bottom border-dark border-opacity-10">
          <div class="energy-cost">
            <span v-for="n in move.cost" :key="n" :class="['energy-dot', pokemon.primaryType]"></span>
          </div>
          <span class="move-name text-capitalize fw-semibold">{{ move.name }}</span>
          <span class="move-damage fw-bold">{{ move.power }}</span>
        </div>
      </div>

      <div class="flavor-text my-2 text-muted">
        {{ pokemon.flavor }}
      </div>

      <div class="combat-metrics d-flex justify-content-between text-center border-top border-dark pt-1">
        <div class="metric-box">
          <small class="d-block fw-bold">Debilidad</small>
          <span v-if="pokemon.weakness" :class="['type-icon-small', pokemon.weakness]"></span> x2
        </div>
        <div class="metric-box">
          <small class="d-block fw-bold">Resistencia</small>
          <span v-if="pokemon.resistance" :class="['type-icon-small', pokemon.resistance]"></span> 
          <span v-if="pokemon.resistance">-30</span>
        </div>
        <div class="metric-box">
          <small class="d-block fw-bold">Retirada</small>
          <div class="d-flex justify-content-center">
            <span v-for="n in pokemon.retreat" :key="n" class="energy-dot normal me-1"></span>
          </div>
        </div>
      </div>

      <div class="card-footer-info d-flex justify-content-between mt-1">
        <span>Illus. {{ pokemon.illustrator }}</span>
        <span>{{ pokemon.id }}/1025 {{ pokemon.rarity }}</span>
      </div>

    </div>
  </div>
</template>

<script setup>
defineProps({
  pokemon: { type: Object, required: true }
});
</script>

<style scoped>
.tcg-card {
  width: 100%;
  aspect-ratio: 2.5 / 3.5; 
  border-radius: 12px;
  padding: 10px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.3);
  display: flex;
  flex-direction: column;
}

.card-inner {
  background: rgba(255, 255, 255, 0.85);
  border: 4px solid #f2da5e;
  border-radius: 4px;
  padding: 8px;
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.image-frame { 
  border: 3px solid #b3a125; 
  border-radius: 2px; 
  height: 36%;
  overflow: hidden; 
  box-shadow: inset 0 0 10px rgba(0,0,0,0.2); 
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

.pokemon-img { 
  width: 100%;
  height: 90%;
  object-fit: contain;
  filter: drop-shadow(2px 4px 6px rgba(0,0,0,0.4)); 
  transform: scale(1.15);
  transition: transform 0.3s ease;
}

.image-frame:hover .pokemon-img {
  transform: scale(1.25);
}

.normal { background: linear-gradient(135deg, #A8A77A, #C6C6A7); }
.fire { background: linear-gradient(135deg, #EE8130, #F5AC78); }
.water { background: linear-gradient(135deg, #6390F0, #9DB7F5); }
.electric { background: linear-gradient(135deg, #F7D02C, #FAE078); }
.grass { background: linear-gradient(135deg, #7AC74C, #A7DB8D); }
.ice { background: linear-gradient(135deg, #96D9D6, #BCE6E6); }
.fighting { background: linear-gradient(135deg, #C22E28, #D67873); }
.poison { background: linear-gradient(135deg, #A33EA1, #C183C1); }
.ground { background: linear-gradient(135deg, #E2BF65, #EBD69D); }
.flying { background: linear-gradient(135deg, #A98FF3, #C6B7F5); }
.psychic { background: linear-gradient(135deg, #F95587, #FA92B2); }
.bug { background: linear-gradient(135deg, #A6B91A, #C6D16E); }
.rock { background: linear-gradient(135deg, #B6A136, #D1C17D); }
.ghost { background: linear-gradient(135deg, #735797, #A292BC); }
.dragon { background: linear-gradient(135deg, #6F35FC, #A27DFA); }
.dark { background: linear-gradient(135deg, #705746, #A29288); }
.steel { background: linear-gradient(135deg, #B7B7CE, #D1D1E0); }
.fairy { background: linear-gradient(135deg, #D685AD, #F4BDC9); }

.stage-badge { background: #333; color: white; padding: 2px 6px; font-size: 0.65rem; border-radius: 4px; font-weight: bold; }
.stats-bar { font-size: 0.65rem; background: linear-gradient(to right, transparent, rgba(200,180,50,0.4), transparent); margin-top: 4px; }

.moves-container { 
  padding: 2px 0; 
  display: flex;
  flex-direction: column;
  justify-content: space-evenly; 
  flex-grow: 1;
}
.move-row { display: flex; align-items: center; padding: 1px 0; }
.energy-cost { width: 45px; display: flex; gap: 2px; flex-shrink: 0; }
.energy-dot { width: 10px; height: 10px; border-radius: 50%; border: 1px solid rgba(0,0,0,0.5); box-shadow: inset -1px -1px 3px rgba(0,0,0,0.3); }

.move-name { 
  flex-grow: 1; 
  font-size: 0.85rem; 
  line-height: 1; 
  padding: 0 4px; 
  white-space: nowrap;
  overflow: hidden; 
  text-overflow: ellipsis;
}
.move-damage { font-size: 0.8rem; flex-shrink: 0; }

.flavor-text { 
  font-size: 0.5rem; 
  font-style: italic; 
  line-height: 1.1; 
  border: 1px solid #ccc; 
  padding: 4px; 
  border-radius: 4px; 
  background: rgba(255,255,255,0.5); 
  margin: 4px 0;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.metric-box small { font-size: 0.5rem; text-transform: uppercase; }
.metric-box { font-size: 0.7rem; width: 33%; }
.type-icon-small { display: inline-block; width: 12px; height: 12px; border-radius: 50%; border: 1px solid #000; vertical-align: middle; }
.type-icon { display: inline-block; width: 20px; height: 20px; border-radius: 50%; border: 2px solid #fff; box-shadow: 1px 1px 3px rgba(0,0,0,0.5); }
.card-footer-info { font-size: 0.5rem; font-weight: bold; margin-top: auto; }
</style>