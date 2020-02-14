<template>
  <div id="app">
    <input type="text" v-model="sentence" />
    <button @click="postSentiment">分析</button>
    {{ result }}
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "App",
  data() {
    return {
      sentence: "",
      result: {}
    };
  },
  methods: {
    async auth(clientId, clientSecret) {
      const tokenUrl = "https://api.ce-cotoha.com/v1/oauth/accesstokens";
      const headers = {
        "Content-Type": "application/json;charset=UTF-8"
      };

      const body = {
        grantType: "client_credentials",
        clientId: clientId,
        clientSecret: clientSecret
      };

      const request = await axios({
        method: "post",
        url: tokenUrl,
        headers: headers,
        data: body
      });

      return request.data.access_token;
    },
    async sentiment(sentence, accessToken) {
      const url = process.env.VUE_APP_BASE_URL + "nlp/v1/sentiment";
      const headers = {
        "Content-Type": "application/json;charset=UTF-8",
        Authorization: `Bearer ${accessToken}`
      };
      const body = {
        sentence: sentence
      };

      const request = await axios({
        method: "post",
        url: url,
        headers: headers,
        data: body
      });
      return request;
    },
    async postSentiment() {
      const token = await this.auth(
        process.env.VUE_APP_CLIENT_ID,
        process.env.VUE_APP_CLIENT_SECRET
      );
      this.result = await this.sentiment(this.sentence, token);
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>