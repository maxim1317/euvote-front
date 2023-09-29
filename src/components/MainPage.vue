<template>
  <div v-if="game" class="container-flex px-0 pt-3" style="position: relative">
    <div class="centered px-3 pt-3" style="position: relative">
      <div class="pb-3">
        <div class="">
          <div
            class="w-100 mt-3"
            >
            <!-- <img src="http://127.0.0.1:8000/static/logo/logo.png" class="img-fluid logo" /> -->
            <h1
              style="color: white; font-family: 'GothamBook'; font-size: 5rem;"
            >DISCORDVISION</h1>
          </div>
          <div class="left-flex mt-5">
            <select class="form-select form-select-lg" @change="updatePlayer">
              <option value="" selected disabled>Please select a voter</option>
              <option v-for="name in names" :key="name.name">
                {{ name.name }}
              </option>
            </select>
            <button
              type="button"
              class="btn btn-light centered ms-2"
              style="height: 48px; min-width: 48px !important"
              @click="resetGame"
            >
              <i class="fa fa-refresh fs-3" aria-hidden="true"></i>
            </button>
          </div>
        </div>
        <div v-if="voter.name == 'win'" class="w-100 mt-2">
          <button type="button" class="btn btn-success w-100 fs-5" @click="finishGame">
            Finish Game
          </button>
        </div>
      </div>
    </div>

    <div class="w-100 mt-3" style="position: relative">
      <div class="centered w-100" style="position: relative">
        <transition-group name="flip-list" tag="ul" class="w-100 content-pane">
          <li
            class="p-0 m-0 mb-2 player-card"
            v-for="(participant, index) in participant_list"
            :key="participant.name"
            :class="{ 'bg-warning': participant.name == player_name }"
            :style="{ opacity: 0.7 + 0.3 * (1 - +participant.is_checked) }"
          >
            <PlayerCard
              :index="index"
              :participant="participant"
              :player_name="player_name"
              v-model:voter="voter"
              v-model:votes="vote_buff"
              :participants="participants"
              @checked="checked"
              @jingle="playJingle"
            />
          </li>
        </transition-group>
      </div>
    </div>
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
        if (delta > 9 && delta <= 11) {
          this.playBG("bg_10_8");
        } else if (delta > 6 && delta <= 9) {
          this.playBG("bg_8_6");
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
        win: "http://127.0.0.1:8000/static/audio/win.mp3",
        bg_1_0: "http://127.0.0.1:8000/static/audio/bg_1_0.mp3",
        bg_2_1: "http://127.0.0.1:8000/static/audio/bg_2_1.mp3",
        bg_5_3: "http://127.0.0.1:8000/static/audio/bg_5_3.mp3",
        bg_8_6: "http://127.0.0.1:8000/static/audio/bg_8_6.mp3",
        bg_10_8: "http://127.0.0.1:8000/static/audio/bg_10_8.mp3",
        bg_11_inf: "http://127.0.0.1:8000/static/audio/bg_11_inf.mp3",
        default: "http://127.0.0.1:8000/static/audio/bg.mp3",
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
        var audio = new Audio("http://127.0.0.1:8000/static/audio/jingle_100.mp3");
      } else if (type == "75") {
        audio = new Audio("http://127.0.0.1:8000/static/audio/jingle_75.mp3");
      } else {
        return;
      }
      if (this.bg_audio != null) {
        this.bg_audio.pause();
      }
      await this.awaitJingle(audio);
      if (this.bg_audio != null) {
        this.bg_audio.loop = true;
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
      var url = new URL("http://127.0.0.1:8000/game");
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
      var url = new URL("http://127.0.0.1:8000/game");
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
      var url = new URL("http://127.0.0.1:8000/reset");
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

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
ul {
  list-style-type: none;
  /* padding: 0;
  columns: 2;
  -webkit-columns: 2;
  -moz-columns: 2;bg-danger
  column-gap: 1em;
  margin-top: 1em;
  margin-bottom: 1em; */
}
.card {
  margin-bottom: 1vh;
  margin-right: 0vh;
  margin-left: 1vh;
  min-height: 7.1vh;
  max-height: 7.1vh;
  font-family: "GothamBook";
}

/* .form-select {
  font-family: "GothamBook";
}
option {
  font-family: "GothamBook";
} */

@font-face {
  font-family: "EC";
  src: url("http://127.0.0.1:8000/static/fonts/ec.ttf");
}

.flip-list-move {
  transition: transform 1.6s ease;
}

.container {
  position: relative;
  min-height: 87vh;
  max-height: 87vh;
}

.content-pane {
  position: absolute;
  top: 0;
  left: 0;
  /* max-height:100%; */
  max-height: calc(100vh - 330px);
  /* max-width: 100vw; */
  display: flex;
  padding-top: 0px;
  padding-left: 0px;
  flex-wrap: wrap;
  overflow-y: auto;
  align-content: flex-start;
  justify-content: flex-start;
  gap: 5px;
  /* background-color: rgb(236, 236, 236); */
}

@media (min-width: 900px) {
  .content-pane {
    flex-direction: column;
    align-content: center;
    justify-content: center;
  }
  .logo {
    max-width: 600px !important;
  }
}

@media (max-width: 1400px) {
  .content-pane {
    align-content: flex-start;
    justify-content: flex-start;
    /* left:0; */
  }
}

@media (max-width: 900px) {
  .content-pane {
    max-width: 100%;
    /* align-content: center; */
    justify-content: center;
  }
  .player-card {
    margin-right: 0;
  }
}

@media (max-width: 500px) {
  .content-pane {
    flex-direction: row;
    gap: 0px;
    padding: 0;
    top: 0px;
    max-height: calc(100vh - 300px);
  }

  .player-card {
    /* align-content: flex-start; */
    /* justify-content: flex-start; */
    padding: 0;
    max-width: 100%;
    /* margin-right:25px!important; */
  }
}
</style>