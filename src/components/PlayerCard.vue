<template>
  <div class="left-flex p-0 m-0 px-2" style="height: 55px">
    <div
      class="centered bg-success fs-4 text-white fw-bold me-1"
      style="min-height: 45px !important; min-width: 45px !important"
    >
      {{ index + 1 }}
    </div>
    <div class="p-0 m-0 bg-white me-2 centered" style="min-width: 45px; min-height: 45px">
      <svg
        v-if="participant.avatar === null"
        width="45px"
        height="45px"
        class="p-0 m-0"
        :data-jdenticon-value="participant.name"
      ></svg>
      <img
        v-else
        :src="participant.avatar"
        style="max-height: 45px; max-width: 45px"
        class="img-fluid float-start p-0 m-0"
      />
    </div>
    <!-- style="height:45px;min-width:240px!important;max-width:240px!important" -->
    <div class="bg-danger left-flex px-3 text-white fw-bold player-nickname" style="height: 45px">
      <span class="fs-3 text-nowrap longtext">{{ participant.name }}</span>
    </div>
    <div class="bg-danger" v-if="voter.name != 'win'" style="max-height: 45px !important">
      <select
        class="form-select form-select-lg shadow-none rounded-0 border-0 text-dark"
        style="height: 45px; min-width: 80px; max-width: 80px"
        v-if="voter.name != 'spectators'"
        @update:model-value="updateVote"
        v-model="selectedVote"
      >
        <option class="text-dark" style="display: none" disabled value="">Votes</option>
        <option v-for="vote in voter.available_votes" :key="vote">
          {{ Number(vote) }}
        </option>
      </select>
      <div
        class="row shadow-none rounded-0 border-0 text-dark"
        style="height: 45px; min-width: 150px; max-width: 150px"
        v-else
      >
        <input
          class="col form-control text-dark"
          type="number"
          style="height: 45px; min-width: 80px; max-width: 80px; margin-right: 10px; font-weight: bold; font-size: 1.2rem;"
          v-model="selectedVote"
          @change="updateVote"
        />
        <button
          type="button"
          class="col btn btn-outline-success"
          style="height: 45px; min-width: 45px; max-width: 45px"
          @click="checkOut"
        >
          <i class="fa fa-check"></i>
        </button>
      </div>
    </div>
    <div class="centered ms-2 bg-white" style="min-width: 45px; min-height: 45px">
      <span class="fs-4">
        {{ participant.points }}
      </span>
    </div>
  </div>
</template>

<script>
export default {
  name: "PlayerCard",
  data() {
    return {
      version: null,
      link: null,
      selectedVote: null,
      prev_vote: 0,
    };
  },
  props: {
    index: Number,
    participants: Object,
    participant: Object,
    player_name: String,
    voter: Object,
    votes: Object,
  },
  computed: {
    isSelected() {
      console.log(this.player_name);
      if (this.player_name == this.participant.name) {
        return true;
      }
      return false;
    },
  },
  watch: {
    player_name: {
      handler: function (newVal, oldVal) {
        if (oldVal != newVal) {
          this.prev_vote = 0;
          if (this.votes[this.voter.name] == undefined) {
            this.selectedVote = null;
          } else if (this.votes[this.voter.name][this.participant.name] == undefined) {
            this.selectedVote = null;
          } else {
            this.selectedVote = this.votes[this.voter.name][this.participant.name].vote;
            this.prev_vote = this.selectedVote;
          }
        }
      },
    },
  },
  methods: {
    checkOut() {
      var par = this.participant;
      par.is_checked = !par.is_checked;
      this.$emit("update:participant", par);
      var participants = this.participants;
      var i = participants.findIndex((element) => element.name == this.participant.name);
      participants[i] = par;
      this.$emit("update:participants", participants);
      this.$emit("checked");
    },
    updateVote(selected) {
      if (isNaN(selected)) {
        selected = parseInt(selected.target.value);
        // alert(selected)
      }
      if (selected != null) {
        console.log("111");
        var old = this.participant.points;
        var new_par = this.participant;
        console.log("wtf", this.participant.points, selected, this.prev_vote);
        new_par.points = this.participant.points + Number(selected) - Number(this.prev_vote);
        console.log("jingle", old, new_par.points);
        if (selected >= 100 && !this.participant.j_100_played) {
          console.log("jingle 100");
          this.$emit("jingle", "100");
          new_par.j_100_played = true;
        } else if (selected >= 75 && !this.participant.j_75_played) {
          console.log("jingle 75");
          this.$emit("jingle", "75");
          new_par.j_75_played = true;
        }
        this.$emit("update:participant", new_par);
        this.prev_vote = selected;
        var vote = {
          voted_for: this.participant.name,
          vote: selected,
        };
        var new_votes = this.votes;

        if (new_votes[this.voter.name] == undefined) {
          new_votes[this.voter.name] = {};
          new_votes[this.voter.name][this.participant.name] = {};
        }
        new_votes[this.voter.name][this.participant.name] = vote;
        this.$emit("update:votes", new_votes);
        console.log("votes");
        console.log(new_votes);

        var participants = this.participants;
        var player_i = participants.findIndex((element) => element.name == this.voter.name);
        console.log("player_i", player_i);
        participants[player_i].voted_for[this.participant.name] =
          new_votes[this.voter.name][this.participant.name].vote;
        this.$emit("update:participants", participants);
      } else {
        console.log("222");
        new_par = this.participant;
        new_par.points = this.participant.points - this.prev_vote;
        // alert(new_par.points);
        this.$emit("update:participant", new_par);
        this.prev_vote = 0;
        new_votes = this.votes;
        if (new_votes[this.voter.name] == undefined) {
          new_votes[this.voter.name] = {};
        }
        new_votes[this.voter.name][this.participant.name] = {};
        this.$emit("update:votes", new_votes);
      }
    },
  },

  mounted() {
    let recaptchaScript = document.createElement("script");
    recaptchaScript.setAttribute(
      "src",
      "https://cdn.jsdelivr.net/npm/jdenticon@3.1.0/dist/jdenticon.min.js"
    );
    document.head.appendChild(recaptchaScript);
  },
  created() {},
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.bg-dark {
  background-color: #2c2f33;
}

.longtext {
  max-width: 280px !important;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.player-nickname {
  min-width: 300px;
  max-width: 300px;
}

@media (max-width: 500px) {
  .player-nickname {
    min-width: 200px;
    max-width: 200px;
  }
  .longtext {
    max-width: 170px;
  }
}
</style>