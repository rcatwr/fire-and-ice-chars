<template>
  <v-layout align-center justify-center>
    <v-flex xs12 sm6>
      <v-form @submit.prevent="submitSearch">
        <v-text-field
          label="Search"
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

/**
 * @link https://anapioficeandfire.com/Documentation#characters
 */
const API_ENDPOINT_CHARACTERS =
  "https://www.anapioficeandfire.com/api/characters";

export default {
  name: "SearchBarInput",
  data: () => ({
    searchInput: "",
  }),
  methods: {
    /**
     * When the user submits their search request (hits enter) query the API endpoint
     */
    // set to async
    async submitSearch() {
      this.$emit("search-submitted");

      const response = await axios.get(API_ENDPOINT_CHARACTERS, {
        params: {
          // EXERCISE - Implement query GET parameters to search by name. Watch for case sensitivity.

          name: this.searchInput,
          pageSize: 20,
        },
      });

      // wait for the response from names api call, and pull out the allegiances array of urls
      const houses = await response.data.map((char) => char.allegiances);

      // returns an array of strings retrieved from the 'name' field at the 'house' endpoint
      // allegiences is an array of urls in 'characters', so
      // loop through this array and call the api w/ axios each time
      // covers cases i.e. 'Harrold Hardyng' w/ 2 allegiences
      // The Promise.all() method is used to return a single resolved promise when iterating with map

      const houseText = await Promise.all(
        houses.map(async (item) => {
         //console.log(item);
          return await Promise.all(
            item.map(async (url) => {
              const resp = await axios.get(url);
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
        
      // transmit modified response data to App and Results
      this.$emit("search-responded", response.data);
    },
  },
};
</script>

<style scoped></style>
