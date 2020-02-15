<template>
  <div id="app">
    <c-button @click="executeAnalysis" :disabled="loading" v-if="!doesFinishAnalysis">
      分析
    </c-button>

    <c-spinner
      v-if="loading"
    />
    <div class="chart">
      <line-chart
        :dataCollection="dataCollection"
        v-if="doesFinishAnalysis"
      ></line-chart>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import sentenceData from "./assets/sentence.json";

import CButton from "@/components/CButton.vue";
import CSpinner from "@/components/CSpinner.vue";
import LineChart from "@/components/LineChart.vue";

export default {
  name: "App",
  components: {
    CButton,
    CSpinner,
    LineChart
  },
  data() {
    return {
      // コーパス
      sentenceData: null,
      // コーパスのkey配列
      keys: [],
      loading: false,
      doesFinishAnalysis: false,
      dataCollection: {
        labels: [],
        datasets: [
          {
            label: "まい",
            borderColor: "#f87979",
            data: [1, 2, 6, 1],
            fill: false
          }
        ]
      }
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
      // 分析を実行
      return await Promise.all(
        this.keys.map(async key => {
          const result = await this.sentenceData[key].reduce(
            async (pacc, sentence) => {
              let acc = await pacc;
              const result = await this.sentiment(sentence, token);
              acc.push(result.data.result);
              return acc;
            },
            []
          );
          return result;
        })
      );
    },
    async executeAnalysis() {
      this.loading = true;
      // 分析実行
      try {
        const result = await this.postSentiment();
        console.log(result);
        result.forEach(r => {
          this.dataCollection.datasets[0].data.push(r[0].score)
        })
        console.log(this.dataCollection.datasets[0].data)
      } catch(error) {
        console.log(error)
      }
      // 終了時の表示変更
      this.doesFinishAnalysis = true;
      this.loading = false;
    }
  },
  created() {
    // 分析データを準備
    this.sentenceData = sentenceData;
    this.keys = Object.keys(this.sentenceData);
    // データセットにラベルを追加
    this.keys.forEach(key => {
      this.dataCollection.labels.push(key);
    });
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
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
