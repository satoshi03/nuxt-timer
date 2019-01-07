<template>
  <v-layout
    column
    justify-center
    align-center>
    <h1>やるぞタイマー</h1>
    <h2>周りの人に宣言してやる気を奮い立たせよう</h2>
    <v-flex xs12 md6>
      <div v-if="ready" class="timer-set-panel">
      <v-radio-group label="やること" v-model="purpose" row>
        <v-radio
          label="勉強"
          color="orange darken-3"
          value="study"
        ></v-radio>
        <v-radio
          label="仕事"
          color="orange darken-3"
          value="work"
        ></v-radio>
        <v-radio
          label="読書"
          color="orange darken-3"
          value="read"
        ></v-radio>
        <v-radio
          label="プログラミング"
          color="orange darken-3"
          value="programming"
        ></v-radio>
        <v-radio
          label="休憩"
          color="orange darken-3"
          value="break"
        ></v-radio>
        <v-radio
          label="その他"
          color="orange darken-3"
          value="others"
        ></v-radio>
      </v-radio-group>
        <v-slider
           v-model="min"
           :max=120
           :step=1
           thumb-label="always"
           thumb-size=50
           label="時間"
           height=100
        ></v-slider>
        <v-textarea
          outline
          label="意気込みを書こう！"
          v-bind:value="message"
        ></v-textarea>
      </div>
    </v-flex>
    <v-btn v-if="ready" @click="startWithTweet()" round color="primary" target="_blank" v-bind:href="getStartTweetURL()">ツイートして始める</v-btn>
    <h1 class="timer" v-if="started" v-model="lestMin">{{ getLestMin }}</h1>
    <v-btn v-if="started" @click="toPending" target="_blank" round color="primary">一時停止</v-btn>
    <v-btn v-if="started" @click="finish" target="_blank" round color="primary">早めに終わった！</v-btn>
    <v-btn v-if="started" @click="failed" target="_blank" round color="primary">諦めた...</v-btn>
    <h1 class="timer" v-if="pending" v-model="lestMin">{{ getLestMin }}</h1>
    <h2 v-if="pending">一時停止中</h2>
    <v-btn v-if="pending" @click="restart" round color="primary">再開</v-btn>
    <h1 class="finishMessage" v-if="finished">お疲れ様でした</h1>
    <v-btn v-if="finished" @click="reset" target="_blank" v-bind:href="getFinishedTweetURL()" round color="primary">ツイートして戻る</v-btn>
  </v-layout>
</template>

<script>
export default {
  data () {
    return {
      tweetBaseUrl: "https://twitter.com/intent/tweet?text=",
      min: 60,
      lestMin: 60,
      pendingMin: 0,
      purpose: "study",
      startDateTime: undefined,
      pedingDateTime: undefined,
      timer: undefined,
      status: "ready",
      tweet: false,
      failure: false,
    }
  },
  head () {
    return {
      title: 'やるぞタイマー',
      meta: [
        { hid: 'description', name: 'description', content: '周りの人に宣言することでやる気を奮い立たせるタイマー'  }
      ]
    }
  },
  computed: {
    started() {
      return this.status == "started"
    },
    ready() {
      return this.status == "ready"
    },
    pending() {
      return this.status == "pending"
    },
    finished() {
      return this.status == "finished"
    },
    message() {
      var purposeDict = {
        "study": "勉強",
        "work": "仕事",
        "read": "読書",
        "programming": "プログラミング",
        "break": "休憩",
        "others": "何かしら"
      }
      return "今から" + this.min + "分間" + purposeDict[this.purpose] + "やるぞ！！"
    },
    getLestMin() {
      if (this.lestMin >= 1) {
        return "残り" + this.lestMin.toFixed(0) + "分"
      } else {
        return "残り" + (this.lestMin * 60).toFixed(0) + "秒"
      }
    },
  },
  methods: {
    startWithTweet() {
      this.tweet = true
      this.startCount()
    },
    startWithNoTweet() {
      this.tweet = false
      this.startCount()
    },
    startCount() {
      this.startDateTime = new Date()
      this.lestMin = this.min
      this.status = "started"
      this.timer = setInterval(this.updateLestOfMins, 1000)
    },
    getTweetURL(message) {
      return this.tweetBaseUrl + message + "&hashtags=やるぞタイマー"
    },
    getStartTweetURL() {
      return this.tweetBaseUrl + this.message + "&hashtags=やるぞタイマー"
    },
    getFinishedTweetURL() {
      if (!this.tweet) {
        return ""
      }
      if (this.failure) {
        var spent = this.min - this.lestMin
        return this.getTweetURL(spent.toFixed(0) + "分しかやれなかった... 正直ごめんと思ってる...")
      }
      if (this.lestMin <= 0) {
        return this.getTweetURL("宣言通り" + this.min.toFixed(0) + "分やったぞ！！")
      } else {
        var spent = this.min - this.lestMin
        return this.getTweetURL("宣言より早く" + spent.toFixed(0) + "分でやったぞ！！")
      }
    },
    updateLestOfMins() {
      if (this.status !== "started") {
        return
      }

      var cur = new Date()
      var diff = cur - this.startDateTime
      this.lestMin = this.min - (diff / 1000 / 60) + this.pendingMin
      if (this.lestMin <= 0) {
        this.finish()
      }
    },
    toReady() {
      this.status = "ready"
    },
    toPending() {
      this.pendingDateTime = new Date()
      this.status = "pending"
    },
    failed() {
      this.failure = true
      this.finish()
    },
    finish() {
      this.status = "finished"
      clearInterval(this.timer)
    },
    restart() {
      var cur = new Date()
      var diff = cur - this.pendingDateTime
      this.pendingMin += diff / 1000/ 60
      this.pendingDateTime = undefined
      this.status = "started"
    },
    reset() {
      clearInterval(this.timer)
      this.status = "ready"
      this.pendingMin = 0
      this.pendingDateTime = undefined
      this.tweet = false
      this.failure = false
    }
  }
}
</script>

<style>
.v-slider span {
  font-size: 20px;
}

h1 {
  color: #FF7043;
}

h2 {
  margin-top: 10px;
  margin-bottom: 10px;
  color: #666666;
}

h1 {
  display        : inline-block;
  color          : #ffffff;            /* 文字の色 */
  font-size      : 28pt;               /* 文字のサイズ */
  letter-spacing : 4px;                /* 文字間 */
  text-shadow    : 
       2px  2px 1px #d84315,
      -2px  2px 1px #d84315,
       2px -2px 1px #d84315,
      -2px -2px 1px #d84315,
       2px  0px 1px #d84315,
       0px  2px 1px #d84315,
      -2px  0px 1px #d84315,
       0px -2px 1px #d84315;        /* 文字の影 */
}

h1.timer {
  margin-top: 30px;
  margin-bottom: 30px;
  font-size: 65px;
}

h1.finishMessage {
  margin-top: 30px;
  margin-bottom: 30px;
  font-size: 42px;
}
</style>
