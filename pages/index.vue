<template>
  <v-layout align-center justify-center style="marginTop: 30px; marginBottom: 30px;">
    <v-flex xs12 sm8 md4>
      <v-card class="elevation-12">
        <v-card-text style="paddingBottom: 0px;">
            <v-text-field
              label="Name"
              v-model="myTweet.sender"
            ></v-text-field>
            <v-text-field
              multi-line
              rows="2"
              label="Feedback"
              v-model="myTweet.tweet"
            ></v-text-field>
        </v-card-text>
        <v-card-actions style="paddingTop: 0px;">
          <v-spacer></v-spacer>
          <v-btn color="blue lighten-2 white--text" @click="postTweet" style="marginBottom: 20px; marginRight: 20px; marginTop: 0px;">Tweet</v-btn>
        </v-card-actions>
      </v-card>

      <v-flex style="marginTop: 30px;" v-if="tweets">
        <div class="title grey--text text--darken-1">Tweets</div>
        <v-card v-for="tweet in tweets" :key="tweet._id" style="padding: 10px; margin: 10px 0px;">
          <v-card-title primary-title>
            <div>
              <h3 class="headline mb-0">{{ tweet.sender | isAnonymous }}</h3>
              <div v-html="highlight(tweet.tweet)"></div>
            </div>
          </v-card-title>
          <v-card-actions style="paddingLeft: 30px;">
            <v-btn flat icon color="blue lighten-2" @click="likeTweet(tweet)">
              <v-icon>star</v-icon>&nbsp;&nbsp;{{ tweet.likes }}
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-flex>
    </v-flex>
  </v-layout>
</template>

<script>
  export default {
    data () {
      return {
        myTweet: {
          tweet: undefined,
          sender: undefined
        },
        tweets: []
      }
    },
    filters: {
      isAnonymous (name) {
        if (!name) {
          return 'anonymous'
        }

        if (name.length === 0) {
          return 'anonymous'
        }

        return name
      }
    },
    methods: {
      postTweet () {
        this.$axios.post('https://retrospective-33c0a.firebaseio.com/retro.json', this.myTweet)
          .then(({ data }) => {
            this.$axios.get(`https://retrospective-33c0a.firebaseio.com/retro/${data.name}.json`)
              .then(({ data: postedTweet }) => {
                postedTweet._id = data.name
                this.tweets.push(postedTweet)
                this.myTweet.tweet = undefined
              })
              .catch(error => console.error(error))
          })
          .catch(error => console.error(error))
      },
      likeTweet (tweet) {
        this.$axios.get(`https://retrospective-33c0a.firebaseio.com/retro/${tweet._id}.json`)
          .then(({ data: postedTweet }) => {
            if (!postedTweet.likes) {
              postedTweet.likes = 1
            } else {
              postedTweet.likes++
            }
            this.$axios.patch(`https://retrospective-33c0a.firebaseio.com/retro/${tweet._id}.json`, {
              likes: postedTweet.likes
            }).then(({ data }) => {
              this.$set(this.tweets, this.tweets.indexOf(tweet), { _id: tweet._id, ...postedTweet })
            })
              .catch(error => console.error(error))
          })
          .catch(error => console.error(error))
      },
      highlight (tweetMessage) {
        const hashtag = /\B(#[a-zA-Z0-9]+\b)/g
        return tweetMessage.replace(hashtag, (tag) => {
          return `<span class="blue--text text--lighten-2 body-2">${tag}</span>`
        })
      }
    },
    created () {
      this.$axios.get('https://retrospective-33c0a.firebaseio.com/retro.json')
        .then(({ data }) => {
          Object.entries(data).forEach(([key, value]) => {
            if (data.hasOwnProperty(key)) {
              this.tweets.push({ _id: key, ...value })
            }
          })
        })
        .catch(error => console.error(error))
    }
  }
</script>
