<template >
  <v-layout align-center justify-center>
    <v-flex xs12 sm6 >
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
import parse from "parse-link-header"

/**
 * @link https://anapioficeandfire.com/Documentation#characters
 */
const API_ENDPOINT_CHARACTERS =
  "https://www.anapioficeandfire.com/api/characters";

export default {
  name: "SearchBarInput",
  props: {
    url: {
      type: String,
      required: true,
      default: 'base'
    }
  },
  data: () => ({
    searchInput: "",
    resultsLength: "",
    first:'',
    prev:'',
    next: '',
    last: '',
  }),
  methods: {
    /**
     * When the user submits their search request (hits enter) query the API endpoint
     */
    // set to async
    async submitSearch() {

      

      this.$emit("search-submitted");

      const endpoint = {
        base: API_ENDPOINT_CHARACTERS,
        first: this.first ,
        prev: this.prev,
        next: this.next , 
        last: this.last,
      }

      const response = await axios.get(endpoint[this.url], {
        params: {
          // EXERCISE - Implement query GET parameters to search by name. Watch for case sensitivity.

          name: this.searchInput,
          
          
        },
      });

      const links = parse(response.headers.link)
      
      this.first = links.first.url
      this.next = links.next ? links.next.url : null;
      this.prev = links.prev ? links.prev.url : null;
      this.last = links.last.url

      console.log(this.first, this.next, this.prev, this.last, 'url', this.url)

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
      this.resultsLength = response.data.length;
      this.$emit("search-responded", response.data);
    },
  },
};
</script>

<style scoped>



</style>
