<template>
    <div class="info">
        <h1>Your Spotify Info</h1>
        <div v-if="user_info !== null">
            <img :src="user_info.images[0].url" />
            <p>Name: {{user_info.display_name}}</p>
            <p>Email: {{user_info.email}}</p>
        </div>
    </div>
</template>

<script>
export default {
  name: "info",
  data: () => ({
    user_info: null,
    access_token: ""
  }),

  created() {
    const urlString = window.location.href;
    const url = new URL(urlString);
    this.access_token = url.searchParams.get("access_token");

    if (this.access_token) {
      fetch("https://api.spotify.com/v1/me", {
        headers: {
          Authorization: "Bearer " + this.access_token
        }
      })
        .then(response => response.json())
        .then(response => {
          this.user_info = response;
        });
    }
  }
};
</script>