<template>
  <div id="app">
    <c-button
      @click="postSentiment"
      :disabled="loading"
    >
      分析
    </c-button>
    {{ result }}
  </div>
</template>

<script>
import axios from "axios";
import sentenceData from "./assets/sentence.json"

import CButton from "@/components/CButton.vue"

export default {
  name: "App",
  components: {
    CButton
  },
  data() {
    return {
      sentenceData: null,
      result: {},
      loading: false
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
      // cotohaのトークンを取得
      const token = await this.auth(
        process.env.VUE_APP_CLIENT_ID,
        process.env.VUE_APP_CLIENT_SECRET
      );
      // 感情分析を実行
      Object.keys(this.sentenceData).forEach(async key => {
        const result = await this.sentenceData[key].reduce(async (pacc, sentence) => {
          let acc = await pacc
          const result = await this.sentiment(sentence, token)
          acc.push(result)
          return acc
        }, [])
        this.$set(this.result, key, result)
      })
    }
  },
  created() {
    this.loading = true;
    this.sentenceData = sentenceData
    this.loading = false
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