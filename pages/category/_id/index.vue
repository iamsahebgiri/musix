<template>
  <div>
    <div class="topbox light pb-3 pt-3">
      <TopBar />
      <Shortcuts></Shortcuts>
    </div>
    <div v-if="isLoading" class="d-flex justify-content-center">
      <div class="spinner-border text-primary" role="status">
        <span class="sr-only">Loading...</span>
      </div>
    </div>
    <div class="content d-flex flex-wrap">
      <Album v-for="item in categoryPlaylist" :key="item.id" v-bind:item="item" />
    </div>
    <div class="spacer end"></div>
  </div>
</template>
<script>
import Cookies from "js-cookie";
import axios from "axios";

import HomeComponent from "~/components/HomeComponent";
import SongPlaylist from "~/components/SongPlaylist";
import Album from "~/components/Album";
import Shortcuts from "~/components//Shortcuts";
import TopBar from "~/components//TopBar";

export default {
  data() {
    return {
      categoryId: this.$nuxt._route.params.id,
      isLoading: true,
      categoryPlaylist: []
    };
  },
  components: {
    Shortcuts,
    TopBar,
    SongPlaylist,
    Album
  },
  methods: {
    getData() {
      const config = {
        headers: {
          Authorization: `Bearer ${Cookies.get("token")}`
        }
      };
      axios
        .get(
          `https://api.spotify.com/v1/browse/categories/${this.categoryId}/playlists?country=IN`,
          config
        )
        .then(res => {
          this.categoryPlaylist = res.data.playlists.items;
          this.isLoading = false;
          console.log(res.data);
        })
        .catch(err => {
          console.log(err);
        });
    }
  },
  mounted() {
    if (Cookies.get("token") === undefined) {
      const now = new Date();
      now.setTime(now.getTime() + 1 * 3600 * 1000);
      axios
        .get("https://soundsky.netlify.app/.netlify/functions/getAccessToken")
        .then(res => {
          Cookies.set("token", res.data, { expires: now });
          this.getData();
        });
    } else {
      this.getData();
    }
  }
};
</script>
