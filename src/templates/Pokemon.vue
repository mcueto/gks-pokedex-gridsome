<template>
  <div>
    <!-- Gradient background -->
    <div
      :style="{
        background: `linear-gradient(to bottom, transparent 50% 75%, ${bgColor})`,
        position: 'fixed',
        left: 0,
        right: 0,
        top: 0,
        bottom: 0,
      }"
    />
    <!-- Container -->
    <content-wrapper>
      <!-- Cols for display -->
      <v-row>
        <v-col cols="8" offset="2" sm="5" offset-sm="0">
          <v-img
            :src="`/img/pokemon/${$page.pokemon.id}.svg`"
            :alt="`Image for ${$page.pokemon.name}`"
            style="width: 100%"
            :aspect-ratio="1"
            contain
          >
            <template v-slot:placeholder>
              <v-row class="fill-height ma-0" align="center" justify="center">
                <v-progress-circular indeterminate color="grey lighten-5" />
              </v-row>
            </template>
          </v-img>
        </v-col>
        <v-col cols="12" sm="7">
          <div class="display-1">{{ $page.pokemon.name }}</div>
          <div class="mb-4">
            <v-row dense>
              <v-col
                v-for="type in $page.pokemon.types"
                :key="type.id"
                class="flex-grow-0"
              >
                <poke-type-chip :type="type"></poke-type-chip>
              </v-col>
            </v-row>
          </div>
          <!-- Weight/height -->
          <div class="d-flex mb-2">
            <div class="mr-5 d-flex align-center">
              <v-icon class="mr-2">$height</v-icon>
              <span class="title font-weight-thin"
                >{{ $page.pokemon.height }} ft</span
              >
            </div>
            <div class="d-flex align-center">
              <v-icon class="mr-2">$weight</v-icon>
              <span class="title font-weight-thin"
                >{{ $page.pokemon.weight }} lbs</span
              >
            </div>
          </div>
          <!-- Description -->
          <div v-html="$page.pokemon.species.flavor_text"></div>
          <!-- Weaknesses -->
          <div class="title mt-2">Weaknesses</div>
          <v-row dense>
            <v-col
              v-for="factor in damageFactors"
              :key="factor.type.slug"
              class="flex-grow-0"
            >
              <poke-type-chip
                :type="factor.type"
                small
                :starred="factor.factor > 3.9"
              />
            </v-col>
          </v-row>
        </v-col>
      </v-row>
      <!-- Stats -->
      <v-row>
        <v-col
          v-for="stat in $page.pokemon.stats"
          :key="stat.name"
          cols="4"
          sm="2"
          class="text-center"
        >
          <v-progress-circular
            :value="stat.base"
            class="mb-1 darken-2"
            :color="color"
            size="52"
            width="6"
          >
            {{ stat.base }}
          </v-progress-circular>
          <div :style="{ color }">
            {{ stat.name }}
          </div>
        </v-col>
      </v-row>

      <!-- Evolution chain... -->
      <template v-if="isPartOfChain">
        <v-divider class="my-6"></v-divider>
        <div
          class="d-flex"
          :class="{
            'flex-row': !evChainVertical,
            'flex-column': evChainVertical,
            'justify-center': !evChainVertical,
            'align-center': evChainVertical,
          }"
        >
          <template v-for="(bucket, i) in buckets">
            <div
              :key="bucket[0].pokemon.slug"
              :class="{
                'd-flex': true,
                'align-center': true,
                'flex-column': !evChainVertical,
                'justify-center': !evChainVertical,
              }"
              style="max-width: 100%"
              class="overflow-auto pa-2"
            >
              <poke-list-card
                v-for="species in bucket"
                :key="species.pokemon.slug"
                :pokemon="species.pokemon"
                :elevation="species.pokemon.id == $page.pokemon.id ? 8 : 2"
                class="ma-2"
              />
            </div>
            <div
              v-if="i != buckets.length - 1"
              :key="i"
              class="flex-grow-0 d-flex px-2 py-2"
              :class="{
                'align-center': !evChainVertical,
              }"
            >
              <v-icon>{{ evChainVertical ? "$down" : "$right" }}</v-icon>
            </div>
          </template>
        </div>
      </template>
    </content-wrapper>

    <!-- Bottom navigation -->
    <v-bottom-navigation app grow>
      <v-btn
        :to="prevLink"
        width="100"
        :disabled="!prevLink"
        title="Previous Pokemon"
      >
        <span>{{ _get($page, "pokemon.prev_pokemon.name", "") }}</span>
        <v-icon>$prev</v-icon>
      </v-btn>
      <v-btn to="/" exact title="All Pokemon">
        <span>All</span>
        <v-icon>$list</v-icon>
      </v-btn>
      <v-btn
        :to="nextLink"
        :disabled="!nextLink"
        width="100"
        title="Next Pokemon"
      >
        <span>{{ _get($page, "pokemon.next_pokemon.name", "") }}</span>
        <v-icon>$next</v-icon>
      </v-btn>
    </v-bottom-navigation>
  </div>
</template>

<script>
import PokeTypeChip from "~/components/PokeTypeChip";
import PokeListCard from "../components/PokeListCard";
import { get } from "lodash";

export default {
  components: { PokeTypeChip, PokeListCard },

  /**
   * Page data
   */
  data() {
    return {
      isMounted: false,
    };
  },

  /**
   * Computed props
   */
  computed: {
    // Links to previous/next pokemon
    prevLink() {
      return "/" + get(this.$page, "pokemon.prev_pokemon.slug", "");
    },
    nextLink() {
      return "/" + get(this.$page, "pokemon.next_pokemon.slug", "");
    },
    // Color of the pokemon (with some tweaks for light colors)
    color() {
      const prefix = this.$vuetify.theme.dark ? "Light" : "Dark";

      const rgb = get(
        this.$page,
        `pokemon.species.colorPalette.${prefix}Vibrant.rgb`,
      ) ||
        get(this.$page, "pokemon.species.colorPalette.Vibrant.rgb") || [
          0,
          0,
          0,
        ];
      return `rgb(${rgb[0]}, ${rgb[1]}, ${rgb[2]})`;
    },
    // Background color for the pokemon
    bgColor() {
    	const prefix = this.$vuetify.theme.dark ? 'Dark' : 'Light';

      const rgb = get(
        this.$page,
        `pokemon.species.colorPalette.${prefix}Muted.rgb`,
      ) ||
        get(this.$page, `pokemon.species.colorPalette.${prefix}Vibrant.rgb`) || [
          0,
          0,
          0,
        ];
      return `rgb(${rgb[0]}, ${rgb[1]}, ${rgb[2]})`;
    },
    // Has evolution chain?
    isPartOfChain() {
      return this.$page.pokemon.species.evolution_chain.links.length > 1;
    },
    // Evolution chain buckets...
    buckets() {
      // Start buckets
      const buckets = [],
        links = this.$page.pokemon.species.evolution_chain.links;

      // First species
      const firstSpecies = links.find(link => !link.species.evolves_from)
        .species;
      buckets[0] = [firstSpecies];

      // Loop til we can't find anything more for a bucket
      let areDone = false;
      while (!areDone) {
        const lastBucket = buckets[buckets.length - 1],
          lastBucketIds = lastBucket.map(species => species.pokemon.id);

        const newBucket = links
          .filter(link =>
            lastBucketIds.includes(get(link, "species.evolves_from.id")),
          )
          .map(link => link.species);

        if (!newBucket.length) {
          areDone = true;
        } else {
          buckets.push(newBucket);
        }
      }

      return buckets;
    },

    // Should we display evolution chain vertically? (Small screens)
    evChainVertical() {
      return !this.isMounted || this.$vuetify.breakpoint.smAndDown;
    },

    // Damage factors, form of [{type: ..., factor: 2}, ...]
    damageFactors() {
      // Group factors
      const factors = {};
      for (let factor of this.$page.pokemon.damage_factors) {
        const slug = factor.damage_type.slug;

        // If we already have a record started, append the factor
        if (factors[slug]) {
          factors[slug].factor =
            (factors[slug].factor * parseInt(factor.damage_factor)) / 100;
        }
        // Else, start a new factor
        else {
          factors[slug] = {
            type: factor.damage_type,
            factor: parseInt(factor.damage_factor) / 100,
          };
        }
      }

      // Reduce to array, only include super effective ones
      return Object.values(factors).filter(factor => factor.factor > 1);
    },
  },

  /**
   * Component methods
   */
  methods: {
    _get: get,

    // Key listener
    keydownHandler(e) {
      if (/arrowright/i.test(e.key) || /^k$/i.test(e.key)) {
        this.$router.push(this.nextLink);
      } else if (/arrowleft/i.test(e.key) || /^j$/i.test(e.key)) {
        this.$router.push(this.prevLink);
      }
    },
  },

  /**
   * On mount, mark that we're mounted. (For display purposes...)
   */
  mounted() {
    this.isMounted = true;
    window.addEventListener("keydown", this.keydownHandler);
  },

  beforeDestroy() {
    window.removeEventListener("keydown", this.keydownHandler);
  },

  /**
   * Page meta
   */
  metaInfo() {
    return {
      title: this.$page.pokemon.name,
      meta: [
        {
          name: "Description",
          content: `Details about Pokemon ${this.$page.pokemon.name}`,
        },
      ],
    };
  },
};
</script>

<page-query>
  query ($id: ID!) {
    pokemon(id: $id) {
      id,
      name,
      types { id, name, slug }
      weight
      height
      stats { base, name }
      prev_pokemon { name, slug }
      next_pokemon { name, slug }
      species {
        flavor_text, color,
        colorPalette {
          Vibrant { rgb }
          Muted { rgb }
          DarkVibrant { rgb }
          LightVibrant { rgb }
          DarkMuted { rgb }
          LightMuted { rgb }
        }
        evolution_chain {
          links {
            species {
              pokemon {
                id, name, slug,
                species {
                  colorPalette {
                    Vibrant { rgb }
                    DarkMuted { rgb }
                    DarkVibrant { rgb }
                    LightVibrant { rgb }
                    LightMuted { rgb }
                  }
                }
              }
              evolves_from { id }
            }
          }
        }
      }
      damage_factors {
        damage_type { slug, name }
        damage_factor
      }
    }
  }
</page-query>
