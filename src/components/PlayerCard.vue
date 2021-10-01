<template>
  <div v-if="participants != null" class="">
    <div class="row align-items-center align-middle" style="padding: 0.3vh">
      <div class="col-1" style="margin-right: 10px">
        <svg
          v-if="participant.avatar === null"
          width="6vh"
          height="6vh"
          :data-jdenticon-value="participant.name"
        ></svg>
        <img
          v-else
          :src="participant.avatar"
          style="max-height: 6vh; max-width: 6vh"
          class="img-fluid float-start"
        />
      </div>
      <div class="col" style="margin-right: -100px">
        <p class="h4">{{ participant.name }}</p>
      </div>
      <div v-if="voter.name != 'win'" class="col-3">
        <multiselect
          v-if="voter.name != 'spectators'"
          v-model="selectedVote"
          :options="voter.available_votes"
          :close-on-select="true"
          :clear-on-select="false"
          :show-labels="false"
          placeholder="0"
          @update:model-value="updateVote"
        >
        </multiselect>
        <div v-else class="row">
          <input
            class="form-control col"
            type="number"
            v-model="selectedVote"
            @change="updateVote"
          />
          <button type="button" class="btn btn-outline-success col-3" @click="checkOut">
            <i class="fa fa-check"></i>
          </button>
        </div>
      </div>
      <div class="col-2">
        <p class="h4" style="text-align: right; padding-right: 12px">{{ participant.points }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import Multiselect from "@suadelabs/vue3-multiselect";

export default {
  name: "PlayerCard",
  components: {
    Multiselect,
  },
  data() {
    return {
      version: null,
      link: null,
      selectedVote: null,
      prev_vote: 0,
    };
  },
  props: {
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
        new_par.points = this.participant.points + selected - this.prev_vote;
        console.log("jingle", old, new_par.points);
        if (old < 100 && new_par.points >= 100) {
          console.log("jingle 100");
          this.$emit("jingle", "100");
        } else if (old < 75 && new_par.points >= 75) {
          console.log("jingle 75");
          this.$emit("jingle", "75");
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
<style src="./vue3-multiselect.css"></style>
<style scoped>
.card {
  margin-bottom: 1vh;
  margin-right: 0vh;
  margin-left: 1vh;
  min-height: 7em;
  max-height: 7em;
}

.h4 {
  font-size: 1.8em;
}

.bg-dark {
  background-color: #2c2f33;
}

.h4 {
  margin-bottom: -1px;
}
</style>
