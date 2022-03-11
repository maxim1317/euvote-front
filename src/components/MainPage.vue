<template>
  <div v-if="game" class="row container-fluid" style="margin-top: 10px">
    <transition-group class="col-8" name="flip-list" tag="ul">
      <li class="col-12" v-for="participant in participant_list" :key="participant.name">
        <div
          class="solid-card card d-flex"
          :class="{ 'bg-warning': participant.name == player_name }"
          :style="{ opacity: 0.7 + 0.3 * (1 - +participant.is_checked) }"
        >
          <!-- {{voter}} -->
          <PlayerCard
            :participant="participant"
            :player_name="player_name"
            v-model:voter="voter"
            v-model:votes="vote_buff"
            :participants="participants"
            @checked="checked"
            @jingle="playJingle"
          >
          </PlayerCard>
        </div>
      </li>
    </transition-group>
    <div class="col text-light selector">
      <div class="card-body">
        <div class="row">
          <img src="http://localhost:8000/static/logo/logo.png" class="img-fluid m-3">
        </div>
        <div class="row">
          <div class="col-10">
            <select class="form-select form-select-lg" @change="updatePlayer">
              <option value="" selected disabled>Please select a voter</option>
              <option v-for="name in names" :key="name.name">
                {{ name.name }}
              </option>
            </select>
          </div>
          <button type="button" class="col btn btn-outline-warning" @click="resetGame">
            <i class="fa fa-refresh" aria-hidden="true"></i>
          </button>
        </div>
      </div>
      <div v-if="voter.name == 'win'" class="row" style="padding: 15px">
        <button type="button" class="btn btn-success" @click="finishGame">Finish Game</button>
      </div>
    </div>

    <!-- </div> -->
  </div>
</template>

<script>
import PlayerCard from "./PlayerCard.vue";

export default {
  name: "MainPage",
  components: {
    PlayerCard,
  },
  data() {
    return {
      game: null,
      participants: null,
      names: [],
      audio_name: null,
      bg_audio: null,
      player_name: null,
      voter: null,
      vote_buff: {},
      waits_for_audio: false,
    };
  },
  mounted() {},
  computed: {
    participant_list() {
      if (this.game !== null) {
        return this.participants.filter((part) => part.name != "spectators");
      }
      return [];
    },
  },
  methods: {
    finishGame() {
      this.participants.forEach((element) => {
        element.is_checked = false;
      });
    },
    checked() {
      var sum = 0;
      this.participants.forEach((element) => {
        sum += +element.is_checked;
      });

      this.shuffle();

      console.log(this.voter);
      if (this.voter.name == "spectators") {
        var delta = this.participants.length - sum - 1;
        if (delta > 6 && delta <= 11) {
          this.playBG("bg_10_6");
        } else if (delta > 3 && delta <= 6) {
          this.playBG("bg_5_3");
        } else if (delta > 1 && delta <= 3) {
          this.playBG("bg_2_1");
        } else if (delta == 1) {
          this.playBG("bg_1_0");
        } else if (delta == 0) {
          this.playBG("win");
          this.voter = {
            name: "win",
            points: 0,
            available_votes: [],
          };
        } else {
          this.playBG("bg_11_inf");
        }
      }
    },
    shuffle() {
      function compare(a, b) {
        if (a.points > b.points) {
          return -1;
        }
        if (a.points < b.points) {
          return 1;
        }
        return 0;
      }
      this.participants.sort(compare);
    },
    playBG(bg) {
      const bgs = {
        win: "http://localhost:8000/static/audio/win.mp3",
        bg_1_0: "http://localhost:8000/static/audio/bg_1_0.mp3",
        bg_2_1: "http://localhost:8000/static/audio/bg_2_1.mp3",
        bg_5_3: "http://localhost:8000/static/audio/bg_5_3.mp3",
        bg_10_6: "http://localhost:8000/static/audio/bg_10_6.mp3",
        bg_11_inf: "http://localhost:8000/static/audio/bg_10_6.mp3",
        default: "http://localhost:8000/static/audio/bg.mp3",
      };
      var audio = null;
      if (bgs[bg] == undefined) {
        audio = bgs.default;
      } else {
        audio = bgs[bg];
      }
      if (audio != this.audio_name) {
        if (this.bg_audio != null) {
          this.bg_audio.pause();
          this.bg_audio.currentTime = 0;
        }
        this.audio_name = audio;
        this.bg_audio = new Audio(audio);
        this.bg_audio.loop = true;
        this.bg_audio.play();
      }
      if (this.waits_for_audio) {
        if (this.bg_audio != null) {
          this.bg_audio.pause();
          this.bg_audio.currentTime = 0;
        }
        this.bg_audio = new Audio(this.audio_name);
        this.bg_audio.play();
        this.bg_audio.loop = true;
        this.waits_for_audio = false;
      }
    },
    awaitJingle(audio) {
      return new Promise((res) => {
        audio.play();
        audio.onended = res;
      });
    },
    async playJingle(type) {
      if (type == "100") {
        var audio = new Audio("http://localhost:8000/static/audio/jingle_100.mp3");
      } else if (type == "75") {
        audio = new Audio("http://localhost:8000/static/audio/jingle_75.mp3");
      } else {
        return;
      }
      if (this.bg_audio != null) {
        this.bg_audio.pause();
      }
      await this.awaitJingle(audio);
      if (this.bg_audio != null) {
        this.bg_audio.play();
      }
    },
    unpackGame(game) {
      this.participants = game.participants;
      if (game.audio_name != null) {
        this.audio_name = game.audio_name;
        this.waits_for_audio = true;
        this.player_name = game.player_name;
        this.voter = game.voter;
        this.vote_buff = game.vote_buff;
      } else {
        this.voter = {
          name: "win",
          points: 0,
          available_votes: [],
        };
      }
      this.names = [];
      Object.values(game.participants).forEach((element) => {
        this.names.push({ name: element.name });
      });
      this.game = game;
      console.log(this.game);
    },
    loadGame() {
      var url = new URL("http://localhost:8000/game");
      fetch(url, { method: "GET" })
        .then((response) => response.json())
        .then((data) => this.unpackGame(data));
    },
    saveGame() {
      var game = this.game;

      game.audio_name = this.audio_name;
      game.player_name = this.player_name;
      game.voter = this.voter;
      game.vote_buff = this.vote_buff;

      console.log("saving game");
      console.log(game);
      var url = new URL("http://localhost:8000/game");
      fetch(url, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(game),
      }).then((response) => console.log(response.json()));
    },
    updatePlayer(event) {
      var selected = event.target.value;
      this.player_name = selected;
      console.log(event);
      if (event.srcElement != undefined) {
        this.playBG();
        this.shuffle();
        this.saveGame();
      }
      if (selected != null) {
        this.is_voting = true;
        this.voter = this.participants.find((element) => element.name == this.player_name);
      } else {
        this.is_voting = false;
      }
      // alert(this.voter)
    },
    resetGame() {
      var url = new URL("http://localhost:8000/reset");
      fetch(url, {
        method: "POST",
      }).then((response) => console.log(response.json()), this.loadGame());
    },
  },
  created() {
    this.loadGame();
  },
};
</script>

<style src="./vue3-multiselect.css"></style>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
ul {
  list-style-type: none;
  padding: 0;
  columns: 2;
  -webkit-columns: 2;
  -moz-columns: 2;
  column-gap: 1em;
  margin-top: 1em;
  margin-bottom: 1em;
}
.card {
  margin-bottom: 1vh;
  margin-right: 0vh;
  margin-left: 1vh;
  min-height: 7.1vh;
  max-height: 7.1vh;
  font-family: "GothamBook";
}

.form-select {
  font-family: "GothamBook";
}
option {
  font-family: "GothamBook";
}

@font-face {
  font-family: "EC";
  src: url("http://localhost:8000/static/fonts/ec.ttf");
}
h1 {
  color: #fff;
  font-size: 4em;
  font-family: "EC";
}

.solid-card {
  -webkit-column-break-inside: avoid;
  page-break-inside: avoid;
  break-inside: avoid;
}

.selector {
  min-height: 90vh;
  margin-left: 2vh;
}

.flip-list-move {
  transition: transform 0.4s ease;
}
</style>
