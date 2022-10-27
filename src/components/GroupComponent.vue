<template>
  <div class="grid-group">
    <div class="navbar-group">
      <GroupNavbar />
    </div>
    <div class="table-group">
      <h3 class="title">Groupe {{ $route.params.groupName }}</h3>
      <div class="box-group">
        <form v-on:submit.prevent="postGroup">
          <div class="grid-bet" v-for="match in groupResource" :key="match['id']">
            <div class="team-bet-1">{{ match["team1"]["description"] }}</div>
            <div class="input-bet-1">
              <input min="0" type="number"
                v-model="match['team1']['score']" :readonly="match['locked']">
            </div>
            <div class="input-bet-2">
              <input min="0" type="number"
                v-model="match['team2']['score']" :readonly="match['locked']">
            </div>
            <div class="team-bet-2">{{ match["team2"]["description"] }}</div>
          </div>
          <div class="div-button-group">
            <button type="submit" class="button-group">Valider</button>
            <div class="updated-properly" v-if="displayStatus && updateProperly.length !== 0 && updateProperly.every(v => v === true)">
              Résultats soumis &#10003;
            </div>
            <div class="not-updated-properly" v-else-if="displayStatus && updateProperly.some(v => v === false)">
              Erreur : Résultats non synchronisés &#10005;
            </div>
            <div class="updated-properly" v-else-if="displayStatus && updateProperly.length === 0">
              Aucuns changements observés &#10003;
            </div>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import _ from 'lodash';
import GroupNavbar from './GroupNavbar.vue';

export default {
  name: 'GroupComponent',
  components: {
    GroupNavbar,
  },
  data() {
    return {
      groupResource: [],
      // keep copy of group resource to send only PATCH /match of the updated matches
      groupResourceCopy: [],
      displayStatus: false,
      updateProperly: [],
    };
  },
  methods: {
    getGroup(groupName) {
      this.$store.dispatch('getGroup', { groupName })
        .then((res) => {
          this.groupResource = res.data.result;
          this.groupResourceCopy = _.cloneDeep(this.groupResource);
        });
    },
    postGroup() {
      this.displayStatus = false;

      const modifyBets = [];

      for (const [group, groupCopy] of _.zip(this.groupResource, this.groupResourceCopy)) {
        if (!_.isEqual(group, groupCopy)) {
          if (group.team1.score === '') {
            group.team1.score = null;
          }
          if (group.team2.score === '') {
            group.team2.score = null;
          }
          modifyBets.push(group);
        }
      }

      if (modifyBets.length !== 0) {
        this.$store.dispatch('patchBets', { bets: modifyBets })
          .then(() => {
            this.groupResourceCopy = _.cloneDeep(this.groupResource);
            this.updateProperly.push(true);
            this.displayStatus = true;

            setTimeout(
              () => {
                this.displayStatus = false;
                this.updateProperly.length = 0;
              },
              2000,
            );
          })
          .catch(() => {
            this.groupResource = _.cloneDeep(this.groupResourceCopy);
            this.updateProperly.push(false);
            this.displayStatus = true;

            setTimeout(
              () => {
                this.displayStatus = false;
                this.updateProperly.length = 0;
              },
              2000,
            );
          });
      } else {
        this.displayStatus = true;
        setTimeout(
          () => {
            this.displayStatus = false;
          },
          2000,
        );
      }
    },
  },
  beforeRouteUpdate(to, from, next) {
    this.getGroup(to.params.groupName);
    this.displayStatus = false;
    next();
  },
  created() {
    this.getGroup(this.$route.params.groupName);
  },
};

</script>

<style lang="css">
.grid-group {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  grid-gap: 10px;
}

@media screen and (max-width: 600px) {
  .grid-group {display: block;}
}

.navbar-group {
  grid-column: 1;
}

.table-group {
  grid-column: 2 / 7;
}

.title {
  font-size: 2rem;
  font-weight: 600;
  line-height: 1.125;
  text-align: center;
}

.box-group {
  border: solid;
  border-width: 1px;
  border-color: #53535321;
  border-radius: 6px;
  color: #535353;
  display: block;
  padding: 1.25rem;
  width: 100%;
}

input {
  width: 100%;
  padding: 0.5rem;
  border-radius: 6px;
  border-width: 1px;
  border-color: rgb(10 10 10 / 10%);
  border-style: solid;
}

.grid-bet {
  display: grid;
  grid-template-columns: repeat(14, 1fr);
  grid-gap: 10px;
  padding: 0.25rem;
}

.team-bet-1 {
  display: flex;
  justify-content: center;
  align-items: center;
  grid-column: 1 / 3;
}

.input-bet-1 {
  grid-column: 3 / 8;
}

@media screen and (max-width: 900px) {
  .team-bet-1 {grid-column: 1 / 5;}
  .input-bet-1 {grid-column: 5 / 8;}
}

.input-bet-2 {
  grid-column: 8 / 13;
}

.team-bet-2 {
  display: flex;
  justify-content: center;
  align-items: center;
  grid-column: 13 / 15;
}

@media screen and (max-width: 900px) {
  .input-bet-2 {grid-column: 8 / 11;}
  .team-bet-2 {grid-column: 11 / 15;}
}

.div-button-group {
  padding-top: 1rem;
  display: grid;
  grid-template-columns: repeat(9, 1fr);
}

.button-group {
  grid-column: 5;
  cursor: pointer;
  padding: 0.35rem;
  background-color: #363636;
  border-color: transparent;
  color: whitesmoke;
  border-radius: 4px;
  font-size: 1rem;
}

.button-group:active {
  border-color: #363636;
  background-color: white;
  color: #363636;
}

.updated-properly {
  text-align: right;
  padding: 0.35rem;
  grid-column: 6 / 10;
  color: green;
  font-weight: bold;
  border-color: green;
  font-size: 1rem;
}

.not-updated-properly {
  text-align: right;
  padding: 0.35rem;
  grid-column: 6 / 10;
  color: red;
  font-weight: bold;
  border-color: red;
  font-size: 1rem;
}
</style>
