<template>
  <v-app>
    <v-toolbar app color="blue lighten-5" light flat>
      <!-- listen for the events on searchBarInput -->
      <search-bar-input
        ref="searchinput"
        v-on:search-submitted="isLoading = true"
        v-on:search-responded="searchResponded"
        v-on:active-nav-links="setActiveLinks"
      />
      <template v-slot:extension>
        <!-- display the tab navigation when form submitted without a search term -- looks for more than one result in the api response -->
        <v-tabs
          v-if="results.length > 1"
          color="blue lighten-5"
          v-model="tab"
          align-with-title
        >
          <!-- set up click events on the tabs to pass values to pageUpdate-->
          <!-- tabs 'prev' and 'next' are deactivated if there are no links -- if first or last page of results from api -->
          <v-tab v-on:click="pageUpdate('first')"> first page </v-tab>
          <v-tab :disabled="!activeLinks.prev" v-on:click="pageUpdate('prev')">
            previous
          </v-tab>
          <v-tab :disabled="!activeLinks.next" v-on:click="pageUpdate('next')">
            next
          </v-tab>
          <v-tab v-on:click="pageUpdate('last')"> last </v-tab>
        </v-tabs>
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
