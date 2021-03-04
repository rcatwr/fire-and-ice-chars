<template>
  <v-layout align-center justify-center>
    <v-flex xs12 sm6>
      <v-form @submit.prevent="submitSearch" class="light">
        <v-text-field
          class="inputField"
          label="Search Character Name"
          prepend-inner-icon="search"
          single-line
          solo
          clearable
          v-model="searchInput"
        ></v-text-field>
      </v-form>
    </v-flex>
  </v-layout>
</template>


<script>
import axios from "axios";
// utility for formatting the Link header in the api response:
// https://www.npmjs.com/package/parse-link-header
import parse from "parse-link-header";

/**
 * @link https://anapioficeandfire.com/Documentation#characters
 */
const API_ENDPOINT_CHARACTERS =
  "https://www.anapioficeandfire.com/api/characters";

export default {
  name: "SearchBarInput",
  data: () => ({
    searchInput: "",
    resultsLength: "",
    //store links here
    first: "",
    prev: "",
    next: "",
    last: "",
    // default value
    final: API_ENDPOINT_CHARACTERS,
  }),
  methods: {
    selectEndpointAndTrigger(trigger) {
      //triggered on the click event on the navigation tabs

      const endpoint = {
        base: API_ENDPOINT_CHARACTERS,
        first: this.first,
        prev: this.prev,
        next: this.next,
        last: this.last,
      };
      // sets the url here to plug into the request below
      this.final = endpoint[trigger];
      // call the search function
      this.submitSearch(trigger);
    },

    /**
     * When the user submits their search request (hits enter) query the API endpoint
     */
    // set to async/await pattern
    async submitSearch(trigger) {
      //hacky! checks that the if the method is called via submit its recieving an object.
      // if so -- sets to base url
      this.final =
        typeof trigger === "object" ? API_ENDPOINT_CHARACTERS : this.final;

      this.$emit("search-submitted");

      const response = await axios.get(this.final, {
        params: {
          // EXERCISE - Implement query GET parameters to search by name. Watch for case sensitivity.

          name: this.searchInput,
        },
      });

      // pull out the links 'next page', etc. out of response
      const links = await parse(response.headers.link);

      // set links on these props -if available
      this.first = await links.first.url;
      this.next = (await links.next) ? links.next.url : null;
      this.prev = (await links.prev) ? links.prev.url : null;
      this.last = await links.last.url;

      // object to pass over to App -- lets the tab interface know what links are available on the current result page
      const linksClicked = {
        first: this.first,
        prev: this.prev,
        next: this.next,
        last: this.last,
      };
      // send the object over to update parent state
      this.$emit("active-nav-links", linksClicked);

      // wait for the response from names api call, and pull out the allegiances array of urls
      const houses = await response.data.map((char) => char.allegiances);

      // returns an array of strings retrieved from the 'name' field at the 'house' endpoint
      // allegiences stred as array of urls in 'characters', so
      // loop through this array and call the api w/ axios each time
      // covers cases i.e. 'Sansa Stark' w/ 2 allegiences
      // The Promise.all() method is used to return a single resolved promise when iterating with map
      // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all

      const houseText = await Promise.all(
        // for each 'house' list of urls
        houses.map(async (item) => {
          return await Promise.all(
            // for each individual house url in each house list
            item.map(async (url) => {
              const resp = await axios.get(url);
              // return house name i.e. 'House Stark of Winterfell'
              return await resp.data.name;
            })
          );
        })
      );

      // await the return of the list of houses, and then modify the response.data list:
      // add a new property called "house", plug in the value of "houseText" based on its index.
      // use the .house property in App and Results components to render the allegiences as text strings.

      response.data.forEach((char, i) => {
        char["house"] = [];
        char["house"].push(houseText[i]);
      });
      // results length flags whether call returned 0/1/>1 results
      // transmit modified response data to App and Results
      this.resultsLength = response.data.length;
      this.$emit("search-responded", response.data);
    },
  },
};
</script>

<style scoped>
</style>
