<template>
  <v-flex xs12 sm6>
    <v-card
      v-for="(result, index) in results"
      :key="index"
      class="search-result ma-3"
    >
      <!-- EXERCISE - Format search results -->
      <v-list>
        <v-list-tile>
          <!-- if no name, render 'unknown' -->
          <h1>Name: {{ result.name || "Unknown" }}</h1>
          <!-- if no aliases, hide -->
           <div v-if="result.aliases[0]">
            Also known as:
            <ul>
              <li v-for="alias of result.aliases">
                {{ alias }}
              </li>
            </ul>
          </div>
          <!-- if no allegiances, don't display -->
          <span v-if="result.house.length">
            allegiances:
            <ul>
              <li v-for="house of result.house">
                {{ house }}
              </li>
            </ul>
          </span>
          <!-- if no dob, print unknown -->
          <p>birth: {{ result.birth || "unknown" }}</p>

          <!-- hide if no date present -->
          <p v-if="result.death">died: {{ result.death }}</p>
          <!-- render unknown if no culture present -->
          <p>culture: {{ result.culture || "unknown" }}</p>
          <p>gender: {{ result.gender }}</p>
        
         
      
        </v-list-tile>
      </v-list>
    </v-card>
  </v-flex>
</template>

<script>
export default {
  name: "Results",
  props: {
    results: {
      type: Array,
      required: true,
    },
  },
};
</script>

<style scoped>
.search-result {
  min-height: 100px;
}
</style>
