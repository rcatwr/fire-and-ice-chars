<template>
  <v-app>
    <v-toolbar light app prominent color="blue lighten-2">
      <!-- listen for the events on searchBarInput -->
      <search-bar-input
        ref="searchinput"
        v-on:search-submitted="isLoading = true"
        v-on:search-responded="searchResponded"
        v-on:active-nav-links="setActiveLinks"
      />
      <template v-slot:extension>
        <v-layout align-center justify-center>
          <v-row v-if="results.length > 1">
            <v-btn text v-on:click="pageUpdate('first')"> First Page </v-btn>
            <v-btn
              v-on:click="pageUpdate('prev')"
              :disabled="!activeLinks.prev"
            >
              <v-icon color="blue darken-3" large>chevron_left</v-icon>
            </v-btn>
            <v-btn
              v-on:click="pageUpdate('next')"
              :disabled="!activeLinks.next"
            >
              <v-icon color="blue darken-3" large>chevron_right</v-icon>
            </v-btn>
            <v-btn text v-on:click="pageUpdate('last')"> Last Page </v-btn>
          </v-row>
        </v-layout>
      </template>
    </v-toolbar>

    <v-content>
      <v-layout justify-center>
        <!-- Loading spinner -->
        <v-progress-circular
          v-if="isLoading"
          indeterminate
          class="mt-5"
        ></v-progress-circular>
        <!-- Search results -->
        <results v-else :results="results" :init="init" />
      </v-layout>
    </v-content>
  </v-app>
</template>

<script>
import SearchBarInput from "./components/SearchBarInput";
import Results from "./components/Results";

export default {
  name: "App",
  components: {
    SearchBarInput,
    Results,
  },
  data: () => ({
    isLoading: false,
    // An array of Song of Ice and Fire Characters to display
    results: [],
    // app initialization flag
    init: true,
    //stores links for the next search if search via tab interface
    activeLinks: {},
  }),
  methods: {
    setActiveLinks(links) {
      // sets the list of active links to control page navigation
      // console.log('link', links)
      this.activeLinks = links;
    },

    /**
     * When the API responds, display search results
     * @param {object} response - API response
     */
    searchResponded(response) {
      this.isLoading = false;

      // EXERCISE - Implement logic to handle the API response.

      //  name, alias, gender, culture, date of birth, date of death (if present), and allegiances
      // formatted to an object and passed to the component's results array

      const characters = response.map((char) => ({
        name: char.name,
        birth: char.birth,
        death: char.death,
        // flatten this array
        house: char.house.flat(),
        culture: char.culture,
        gender: char.gender,
        aliases: char.aliases,
      }));

      this.init = false;

      this.results = characters;
      //console.log(characters.length); // eslint-disable-line no-console
    },
    pageUpdate(url) {
      // sends a string value i.e. 'next', 'prev', etc. to the searchinput component to set the
      // appropriate link to plug into the api request
      this.$refs.searchinput.selectEndpointAndTrigger(url);
    },
  },
};
</script>
