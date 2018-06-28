<template>
    <div class="info">
        <h1>Your Spotify Info</h1>
        <div v-if="user_info !== null">
            <img :src="user_info.images[0].url" />
            <p>Name: {{user_info.display_name}}</p>
            <p>Email: {{user_info.email}}</p>
        </div>

        <div v-if="track_info !== null">
          <h3>You're currently listening to:</h3>
          <img :src="track_info.album.images[0].url" />
          <p>{{track_info.name}} by {{track_info.artists[0].name}}</p>
        </div>
    </div>
</template>

<script>
window.onSpotifyWebPlaybackSDKReady = () => {};

export default {
  name: "info",
  data: () => ({
    user_info: null,
    access_token: "",
    track_info: null
  }),

  async created() {
    const urlString = window.location.href;
    const url = new URL(urlString);
    this.access_token = url.searchParams.get("access_token");

    if (this.access_token) {
      this.user_info = await fetchAsync("https://api.spotify.com/v1/me", {
        headers: {
          Authorization: "Bearer " + this.access_token
        }
      });

      const { Player } = await waitForSpotifyWebPlaybackSDKToLoad();
      const sdk = new Player({
        name: "Web Playback SDK",
        volume: 1.0,
        getOAuthToken: callback => {
          callback(this.access_token);
        }
      });

      sdk.on("player_state_changed", state => {
        // Update UI with playback state changes
        //console.log("STATE CHANGED", state);
        this.track_info = state.track_window.current_track;
      });

      sdk.connect();
    }
  }
};

async function fetchAsync(url, options) {
  const response = await fetch(url, options);
  return await response.json();
}

async function waitForSpotifyWebPlaybackSDKToLoad() {
  return new Promise(resolve => {
    if (window.Spotify) {
      resolve(window.Spotify);
    } else {
      window.onSpotifyWebPlaybackSDKReady = () => {
        resolve(window.Spotify);
      };
    }
  });
}
</script>