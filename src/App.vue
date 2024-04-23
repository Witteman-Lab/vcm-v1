<template>
  <v-container id="ourApp" fluid>
    <v-layout>
      <v-app-bar height="60" app color="white" elevation="7" fixed>
        <div class="w-25 ml-5">
          <v-img
            height="50"
            src="./assets/logo.svg"
            class="logo-image"
            @click="handleLogoClick"
          ></v-img>
        </div>
        <v-spacer></v-spacer>
        <div class="mt-5 mr-10">
          <v-select
            variant="outlined"
            flat
            :items="languageItems"
            v-model="language"
            menu-icon=""
            base-color="#398064"
            color="#398064"
          ></v-select>
        </div>
      </v-app-bar>
      <v-main id="main">
        <slider-range
          :language="language"
          :uid="uid"
          :returnUrl="returnUrl"
        ></slider-range>
      </v-main>
    </v-layout>
    <div class="d-flex justify-center mt-15">
      <v-btn
        class="montserratSemiBoldBtn"
        color="#398064"
        variant="outlined"
        rounded
        x-large
        v-if="this.returnUrl !== null"
      >
        <span :style="{ display: isMobile ? 'none' : 'inline-block' }"
          ><a style="color: #398064" :href="this.returnUrl"
            >Retour au sondage</a
          ></span
        >
      </v-btn>
    </div>
  </v-container>
</template>

<script>
import SliderRange from "./components/SliderRange.vue";

const languageItems = ["en", "fr"]; // Define language items as a constant

export default {
  components: {
    SliderRange,
  },
  /**
   * Method: data
   * Description: Returns the initial data for the component.
   * @returns {object} - An object containing the initial data properties.
   */
  data() {
    return {
      language: "",
      returnUrl: "",
      uid: "",
      languageItems, // Use the constant for language items
    };
  },
  methods: {
    /**
     * Method: handleLogoClick
     * Description: Reloads the current page when the logo is clicked.
     */
    handleLogoClick() {
      window.location.reload();
    },
  },
  watch: {
    /**
     * Method: language
     * Description: Updates the language setting and URL parameters when the language is changed.
     * @param {string} newLanguage - The new language code.
     */
    language(newLanguage) {
      localStorage.setItem("language", newLanguage);
      const urlParams = new URLSearchParams(window.location.search);
      urlParams.set("lang", newLanguage);
      window.history.replaceState(
        {},
        "",
        `${window.location.pathname}?${urlParams}`
      );
    },
    /**
     * Method: returnUrl
     * Description: Updates the return URL setting and URL parameters with the new return URL.
     * @param {string} newReturnUrl - The new return URL.
     */
    returnUrl(newReturnUrl) {
      localStorage.setItem("returnUrl", newReturnUrl);
      const urlParams = new URLSearchParams(window.location.search);
      urlParams.set("returnUrl", newReturnUrl);
      window.history.replaceState(
        {},
        "",
        `${window.location.pathname}?${urlParams}`
      );
    },
    /**
     * Method: uid
     * Description: Updates the user ID setting and URL parameters with the new user ID.
     * @param {string} newUid - The new user ID.
     */
    uid(newUid) {
      localStorage.setItem("uid", newUid);
      const urlParams = new URLSearchParams(window.location.search);
      urlParams.set("uid", newUid);
      window.history.replaceState(
        {},
        "",
        `${window.location.pathname}?${urlParams}`
      );
    },
  },
  /**
   * Hook: created
   * Description: Called when the component is created. Retrieves language, returnUrl, and uid from local storage or URL parameters.
   */
  created() {
    // Retrieve language from local storage on component creation
    const urlParams = new URLSearchParams(window.location.search);
    this.language =
      urlParams.get("lang") || localStorage.getItem("language") || "en";
    this.returnUrl =
      urlParams.get("returnUrl") || localStorage.getItem("returnUrl") || "";
    this.uid = urlParams.get("uid") || localStorage.getItem("uid") || "";
  },
};
</script>

<style scoped lang="scss">
#ourApp {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3d50;
}

@media only screen and (max-width: 675px) {
  #main {
    margin-top: 100px;
  }
}

.logo-image {
  cursor: pointer; /* Apply the pointer cursor style */
}

a,
a::before,
a::after {
  text-decoration: none;
  color: #398064;
}
</style>
