<template>
  <v-container v-if="results.length">
    <v-card
      
      v-for="(result, index) in results"
      :key="index"
      class="search-result mx-auto mb-2"
      max-width="600"
      
    >
      <!-- EXERCISE - Format search results -->
     
         <v-card-text>
          <!-- if no name, render 'unknown' -->
         <h1 class="display-2 blue--text text--darken-3 font-weight-light mb-2">  <v-icon class="account" color="blue darken-3" large >account_circle</v-icon> {{ result.name || "Name Unknown" }}</h1>
          <!-- if no aliases, hide -->
       
           <div class="mb-3" v-if="result.aliases[0]">
            <span class="text-uppercase font-weight-bold">aka:</span>
            <ul>
              <li class="subtitle-1 blue--text text--darken-2" :key="alias" v-for="alias of result.aliases">
                {{ alias }}
              </li>
            </ul>
           </div>
          <!-- if no allegiances, don't display -->
        
          <div class="mb-3" v-if="result.house.length">
            <span class="font-weight-bold text-uppercase">allegiances:</span>
            <ul class="body-1">
              <li class="body-1" v-for="house of result.house" :key=house>
                <v-icon class="blue--text">home</v-icon> {{ house }}
              </li>
            </ul>
          </div>
        
          <!-- if no dob, print unknown -->
          <p class="body-1"><strong class="text-uppercase">birth:</strong> {{ result.birth || "unknown" }}</p>

          <!-- hide if no date present -->
          <p class="body-1" v-if="result.death"><strong>died:</strong> {{ result.death }}</p>
          <!-- render unknown if no culture present -->
          <p  class="body-1"><strong class="text-uppercase">culture:</strong> {{ result.culture || "unknown" }}</p>
          <p class="body-1"><strong class="text-uppercase">gender:</strong> {{ result.gender }}</p>
        
      
      </v-card-text>
    </v-card>
  </v-container>
  <v-container v-else-if="init">
    <p class="message display-1 blue--text text--darken-3 font-weight-light">Search for a character!</p>
  </v-container>
    <v-container v-else>
    <p class="message display-1 blue--text text--darken-3 font-weight-light"><v-icon large class='err' color='red lighten-3'>error</v-icon> Oops! Please try again!</p>
  </v-container>
</template>

<script>
export default {
  name: "Results",
  props: {
    results: {
      type: Array,
      required: true,
    },
    init: {
      type: Boolean,
      required: true,
    },

  },
};
</script>

<style scoped>
/* .search-result {
  min-height: 100px; */
   li {
     list-style-type: none;
   }
   .account {
     line-height: 1.5em;
   }
   .message {
     text-align: center;
   }
   .err {
     line-height: 1.25em;
   }
   

</style>
