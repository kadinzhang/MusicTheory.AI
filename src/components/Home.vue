<template>
  <div>
    <section class="hero">
      <v-container class="content">
        <v-layout row class="main">
          <v-flex sm6 class="logo">
            <p>
              <v-icon color="primary">queue_music</v-icon>MusicTheory.ai
            </p>
          </v-flex>
          <v-flex sm6 class="slogan">
            <p>
              Intelligent
              <br />Music Creation
            </p>
          </v-flex>
        </v-layout>
        <v-layout row>
          <v-flex sm4 class="step s1">
            <p>Upload Audio</p>
            <div class="r1">
              <AudioSVG />
            </div>
          </v-flex>
          <v-flex sm4 class="step s2">
            <p>Define Progression</p>
            <h1 class="primary--text">
              I – V –
              <span>vi</span> – IV
            </h1>
          </v-flex>
          <v-flex sm4 class="step s3">
            <p>Get Generated Music</p>
            <div class="r2">
              <Loader />
            </div>
          </v-flex>
        </v-layout>
        <v-layout row class="go">
          <v-flex sm12>
            <v-btn outline color="white">
              <router-link to="app" style="text-decoration: none; color: white">Get Started</router-link>
            </v-btn>
            <v-btn outline color="white" @click="scrollDown()">Learn More</v-btn>
          </v-flex>
        </v-layout>
      </v-container>
    </section>
    <section class="information">
      <p class="lead">How it Works</p>
      <p class="leadtxt">
        MusicTheory.ai is currently under development by the Hopkins AI club. It is a project to use
        <a
          href="https://magenta.tensorflow.org/"
        >Magenta.js</a> neural networks for polyphonic music transcription to convert raw audio to sheet music,
        and then expand upon it with progression-trained LSTM models.
        Models are created with
        <a
          href="https://js.tensorflow.org/"
        >Tensorflow.js</a>, allowing for fast,
        GPU-accelerated inference on the browser.
      </p>
    </section>
    <v-container>
      <v-layout row class="transcription">
        <v-flex sm7 class="visual"></v-flex>
        <v-flex sm5 class="text"></v-flex>
      </v-layout>
      <!-- <v-layout row class="generation"></v-layout> -->
    </v-container>
  </div>
</template>

<script>
import AudioSVG from "./ui/AudioSVG";
import ProgressionSVG from "./ui/ProgressionSVG";
import Loader from "./ui/Loader";

export default {
  name: "Home",
  components: {
    AudioSVG,
    ProgressionSVG,
    Loader
  },
  data() {
    return {};
  },
  methods: {
    scrollDown() {}
  },
  mounted: function() {
    var tl = new TimelineMax();
    tl.from(".logo", 1, { y: 20, opacity: 0 })
      .from(
        ".slogan",
        1,
        {
          y: 20,
          opacity: 0
        },
        "-=0.7"
      )
      .from(
        ".s1",
        1,
        {
          y: 20,
          opacity: 0
        },
        "-=0.5"
      )
      .from(
        ".s2",
        1,
        {
          y: 20,
          opacity: 0
        },
        "-=0.5"
      )
      .from(
        ".s3",
        1,
        {
          y: 20,
          opacity: 0
        },
        "-=0.5"
      )
      .from(
        ".go",
        1,
        {
          y: 20,
          opacity: 0
        },
        "-=0.2"
      );
  }
};
</script>

<style lang="scss" scoped>
.hero {
  height: 60vh;
  min-height: 500px;
  background: #192629 !important;
  display: flex;
  align-items: center;
  justify-content: center;
  .content {
    max-width: 1000px;
    .main {
      margin-bottom: -20px;
      color: white;
    }
    .logo {
      margin-bottom: 70px;
      p {
        margin-top: -27px;
        text-align: right;
        font-size: 30px;
        padding: 20px;
        font-weight: 200;
        i {
          font-size: 34px;
        }
      }
    }
    .slogan {
      p {
        border-left: 1px white solid;
        padding: 0 20px;
        line-height: 32px;
        font-size: 30px;
        font-weight: 400;
      }
    }
    .step {
      .r1 {
        margin-top: 10px;
      }
      .r2 {
        margin-top: -19px;
      }
      text-align: center;
      p {
        color: white;
        opacity: 0.8;
        font-size: 18px;
        font-weight: 200;
        margin-bottom: 0;
      }
      h1 {
        margin-top: 28px;
        font-size: 35px;
      }
    }
    .go {
      text-align: center;
    }
  }
}
.information {
  width: 580px;
  text-align: center;

  margin: 90px auto;
  margin-bottom: 190px;
  .lead {
    font-size: 30px;
    font-weight: normal;
  }
  .leadtxt {
    font-weight: 300;
    font-size: 18px;
    a {
      text-decoration: none;
    }
  }
}
</style>
