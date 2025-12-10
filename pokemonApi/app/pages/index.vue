<template>
  <v-row class="py-4 py-sm-4">
    <v-col lg="6" md="9" sm="9" cols="12" class="mx-auto">
      <v-row>
        <v-col lg="4" md="4" sm="12" cols="12">
          <Input
            v-model="search"
            label="Procurar"
            id="search"
            placeholder="Digite aqui"
          ></Input>
        </v-col>
        <v-col lg="4" md="4" sm="6" cols="6">
          <Select v-model="inputType" label="Tipos" :options="types" />
        </v-col>
        <v-col lg="4" md="4" sm="6" cols="6">
          <Select v-model="inputRegion" label="Região" :options="regions" />
        </v-col>
      </v-row>
    </v-col>
  </v-row>

  <!-- CARDS  -->
  <v-row>
    <v-col lg="9" md="10" sm="11" cols="12" class="mx-auto">
      <v-row :dense="xs">
        <v-col v-for="n in 12" cols="2" v-if="loading">
          <v-skeleton-loader
            class="border"
            type="image, article"
          ></v-skeleton-loader>
        </v-col>
        <v-col v-else-if="filteredData.length === 0">
          <v-alert
            title="Ops..."
            text="Nenhum pokemon encontrado"
            type="amber-lighten-1"
            closable
          ></v-alert>
        </v-col>
        <v-col
          cols="6"
          sm="4"
          md="3"
          v-for="pokemon in filteredData"
          :key="pokemon.id"
          v-else
        >
          <v-hover>
            <template v-slot:default="{ isHovering, props }">
              <v-card
                rounded="lg"
                v-bind="props"
                :elevation="isHovering ? 6 : 2"
                @click.prevent="openDialogInfoPokemon(pokemon)"
              >
                <v-img
                  :src="pokemon.sprites.other['official-artwork'].front_default"
                  :alt="pokemon.name"
                  class="mx-auto"
                  style="width: 60%"
                />
                <v-card-item class="border-t">
                  <div class="d-flex justify-center w-100 text-h5">
                    <p
                      :class="[
                        'text-subtitle-1 font-weight-black',
                        'text-' +
                          getTypeConfig(pokemon?.types?.[0]?.type.name).color,
                      ]"
                    >
                      #{{ String(pokemon.id).padStart(3, "0") }}
                    </p>
                    <p
                      class="text-subtitle-1 font-weight-black text-capitalize ms-2"
                    >
                      {{ pokemon.name }}
                    </p>
                  </div>
                  <v-row no-gutters class="mt-2">
                    {{ smAndDown }}
                    <v-col v-for="(item, id) in pokemon.types" :key="id">
                      <v-chip
                        :size="
                          xs
                            ? 'x-small'
                            : sm
                            ? 'small'
                            : mdAndUp
                            ? 'small'
                            : 'large'
                        "
                        text-color="white"
                        class="text-capitalize"
                      >
                        <v-icon
                          :icon="getTypeConfig(item.type.name).icon"
                          :color="getTypeConfig(item.type.name).color"
                          class="me-1"
                        ></v-icon>
                        {{ item.type.name }}
                      </v-chip>
                    </v-col>
                  </v-row>
                </v-card-item>
              </v-card>
            </template>
          </v-hover>
        </v-col>
      </v-row>
    </v-col>
  </v-row>
  <!-- CARDS  -->

  <v-dialog v-model="dialog" max-width="600">
    <v-card class="position-relative">
      <v-card-item>
        <v-row :dense="xs">
          <v-col cols="12" sm="6">
            <img
              class="elevation-2 ma-1"
              :src="
                pokemonShowed.sprites.other['official-artwork'].front_default
              "
              :alt="pokemonShowed.name"
              style="width: 100%"
            />
          </v-col>
          <v-col cols="12" sm="6">
            <div class="d-flex justify-space-between align-center">
              <span class="d-flex text-h5">
                <p
                  :class="[
                    'font-weight-black',
                    'text-' +
                      getTypeConfig(pokemonShowed?.types?.[0]?.type.name).color,
                  ]"
                >
                  #{{ String(pokemonShowed.id).padStart(3, "0") }}
                </p>
                <p class="font-weight-black text-capitalize ms-2">
                  {{ pokemonShowed.name }}
                </p>
              </span>
              <v-btn
                class="position-absolute top-0 right-0 ma-2"
                variant="tonal"
                icon="mdi-close"
                density="comfortable"
                @click.prevent="
                  {
                    dialog = false;
                    pokemonShowed.value = null;
                  }
                "
              ></v-btn>
            </div>
            <v-row no-gutters class="mt-2">
              <v-col cols="12">
                <p class="font-weight-bold text-subtitle-1">Tipo</p>
              </v-col>
              <v-col class="d-flex ga-3">
                <v-chip
                  v-for="(item, id) in pokemonShowed.types"
                  :key="id"
                  size="small"
                  text-color="white"
                  class="text-capitalize"
                >
                  <v-icon
                    :icon="getTypeConfig(item.type.name).icon"
                    :color="getTypeConfig(item.type.name).color"
                    class="me-1"
                  ></v-icon>
                  {{ item.type.name }}
                </v-chip>
              </v-col>
              <v-col cols="12" class="d-flex ga-4">
                <div>
                  <p class="font-weight-bold text-subtitle-1">Altura</p>
                  <v-chip
                    >{{
                      String(pokemonShowed.height / 10).replace(".", ",")
                    }}m</v-chip
                  >
                </div>
                <div>
                  <p class="font-weight-bold text-subtitle-1">Peso</p>
                  <v-chip
                    >{{
                      String(pokemonShowed.weight / 10).replace(".", ",")
                    }}kg</v-chip
                  >
                </div>
                <div>
                  <p class="font-weight-bold text-subtitle-1">Base Exp.</p>
                  <v-chip>{{ pokemonShowed.base_experience }}</v-chip>
                </div>
              </v-col>
              <v-col cols="12">
                <p class="font-weight-bold text-subtitle-1">Habilidades</p>
                <v-sheet class="d-flex ga-4">
                  <v-chip
                    v-for="(habilidade, id) in pokemonShowed.abilities"
                    :key="id"
                    >{{ habilidade.ability.name }}</v-chip
                  >
                </v-sheet>
              </v-col>
            </v-row>
          </v-col>
        </v-row>
      </v-card-item>
    </v-card>
  </v-dialog>
</template>

<script setup>
import { onMounted, ref, computed } from "vue";
import { useDisplay } from "vuetify";
import Select from "@/components/Select.vue";
import Input from "@/components/Input.vue";
import axios from "axios";

const data = ref([]);
const loading = ref(true);
const regions = ref([]);
const inputRegion = ref(null);
const types = ref([]);
const inputType = ref(0);
const search = ref("");
const dialog = ref(false);
const pokemonShowed = ref(null);

onMounted(() => {
  loadInfos();
  loadRegions();
  loadTypes();
});

const { xs, sm, mdAndUp } = useDisplay();

const filteredData = computed(() => {
  const term = (search.value || "").toLowerCase();

  return data.value.filter((pokemon) => {
    const matchName =
      pokemon.name.toLowerCase().includes(term) ||
      pokemon.id.toString().includes(term);

    // Trata Todos = 0
    const matchType =
      inputType.value === 0 || !inputType.value
        ? true
        : pokemon.types.some((t) =>
            t.type.url.includes(`/type/${inputType.value}/`)
          );

    const matchRegion =
      inputRegion.value === 0 || !inputRegion.value
        ? true
        : pokemon.region_id === inputRegion.value;

    return matchName && matchType && matchRegion;
  });
});
const typeConfig = {
  normal: { icon: "mdi-circle-slice-8", color: "grey-darken-1" },
  flying: { icon: "mdi-butterfly", color: "light-blue-lighten-2" },
  fire: { icon: "mdi-fire-circle", color: "orange-darken-1" },
  water: { icon: "mdi-water-circle", color: "blue-darken-4" },
  bug: { icon: "mdi-bug", color: "teal-darken-2" },
  poison: { icon: "mdi-necklace", color: "purple-darken-2" },
  electric: { icon: "mdi-lightning-bolt", color: "yellow-darken-1" },
  ice: { icon: "mdi-ice-cream", color: "cyan-accent-3" },
  grass: { icon: "mdi-grass", color: "green-darken-2" },
  ground: { icon: "mdi-elevation-rise", color: "brown-darken-1" },
  ghost: { icon: "mdi-ghost", color: "blue-grey-lighten-1" },
  rock: { icon: "mdi-bulma", color: "grey-darken-4" },
  psychic: { icon: "mdi-brain", color: "indigo-accent-4" },
  steel: { icon: "mdi-magnet", color: "grey-lighten-1" },
  fairy: { icon: "mdi-wizard-hat", color: "pink-darken-1" },
  fighting: { icon: "mdi-xmpp", color: "deep-orange-lighten-1" },
};
function openDialogInfoPokemon(pokemon) {
  dialog.value = true;
  pokemonShowed.value = pokemon;
}
function shuffleArray(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
}
function getTypeConfig(type) {
  return typeConfig[type] || { icon: "mdi-help-circle", color: "grey" };
}
function loadInfos() {
  const limit = 18;
  const offset = 0;

  axios
    .get(`https://pokeapi.co/api/v2/pokemon?limit=${limit}&offset=${offset}`)
    .then((res) => {
      return Promise.all(
        res.data.results.map((poke) =>
          axios.get(poke.url).then((details) =>
            axios
              .get(
                `https://pokeapi.co/api/v2/pokemon-species/${details.data.id}`
              )
              .then((species) => {
                let region_id = null;

                if (species.data.pokedex_numbers.length > 0) {
                  // criar um array de Promises para todas as pokedex associadas
                  const pokedexPromises = species.data.pokedex_numbers.map(
                    (p) => axios.get(p.pokedex.url)
                  );
                  return Promise.all(pokedexPromises).then(
                    (pokedexResponses) => {
                      // procurar a primeira pokedex que tenha region
                      for (const pdx of pokedexResponses) {
                        if (pdx.data.region) {
                          region_id = parseInt(
                            pdx.data.region.url.split("/").filter(Boolean).pop()
                          );
                          break; // achou, não precisa continuar
                        }
                      }
                      return {
                        id: details.data.id,
                        name: details.data.name,
                        sprites: details.data.sprites,
                        types: details.data.types,
                        base_experience: details.data.base_experience,
                        height: details.data.height,
                        weight: details.data.weight,
                        abilities: details.data.abilities,
                        region_id: region_id,
                      };
                    }
                  );
                }
                // caso não tenha pokedex associada
                return {
                  id: details.data.id,
                  name: details.data.name,
                  sprites: details.data.sprites,
                  types: details.data.types,
                  base_experience: details.data.base_experience,
                  height: details.data.height,
                  weight: details.data.weight,
                  abilities: details.data.abilities,
                  region_id: null,
                };
              })
          )
        )
      );
    })
    .then((pokemons) => {
      data.value = shuffleArray(pokemons);
    })
    .finally(() => {
      loading.value = false;
    })
    .catch((err) => {
      console.log(err);
    });
}
async function loadTypes() {
  axios
    .get("https://pokeapi.co/api/v2/type")
    .then((res) => Promise.all(res.data.results.map((t) => axios.get(t.url))))
    .then((responses) => {
      const apiTypes = responses.map((r) => ({
        id: r.data.id,
        name: r.data.name,
      }));
      types.value = [{ id: 0, name: "Todos" }, ...apiTypes];
      inputType.value = 0;
    })
    .catch(console.log);
}
async function loadRegions() {
  await axios
    .get("https://pokeapi.co/api/v2/region")
    .then((res) => {
      regions.value = [{ id: 0, name: "Todos" }].concat(
        res.data.results.map((r, idx) => ({
          id: idx + 1,
          name: r.name,
        }))
      );
      inputRegion.value = 0;
    })
    .catch((err) => console.log(err));
}
</script>
