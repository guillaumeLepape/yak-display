<template>
  <section class="hero is-fullheight">
    <div class="navbar-yaktoto">
      <div class="navbar-left">
        <template v-if="
            $store.getters.isAuthenticated &&
            $route.name !== 'login' && $route.name !== 'signup'"
          >
            <div class="navbar-item-custom">
              Utilisateur:&nbsp;<strong>{{ $store.getters.getUserName }}</strong>
            </div>
            <template v-if="
              $store.getters.isAuthenticated &&
              $route.name !== 'login' && $route.name !== 'signup' &&
              $store.getters.getUserName === 'admin'"
            >
              <a @click="computePoints" class="navbar-item-custom clickable">
                Calculer les points
              </a>
              <template v-if="displayStatus">
                <div class="navbar-item-custom success" v-if="pointsComputedProperly">
                  Points calculés &#10003;
                </div>
                <div class="navbar-item-custom error" v-else>
                  Erreur : les points n'ont pas été calculés &#10005;
                </div>
              </template>
            </template>
          </template>
      </div>
      <div class="navbar-right">
        <router-link
            to="/login" class="navbar-item-custom clickable"
            v-if="!($store.getters.isAuthenticated) && !($route.name === 'login')">
          Se connecter
        </router-link>
        <router-link
            to="/signup" class="navbar-item-custom clickable"
            v-if="!($store.getters.isAuthenticated) && !($route.name == 'signup')">
          Créer un compte
        </router-link>
        <router-link
            to="/logout" class="navbar-item-custom clickable"
            v-if="($store.getters.isAuthenticated)">
          Se déconnecter
        </router-link>
      </div>
    </div>

    <div class="body-yaktoto">
      <router-view class="body-yaktoto-page"/>
    </div>

  </section>
</template>

<script>
export default {
  name: 'GroupComponent',
  data() {
    return {
      displayStatus: false,
      pointsComputedProperly: false,
    };
  },
  methods: {
    computePoints() {
      this.$store.dispatch('computePoints')
        .then(() => {
          this.pointsComputedProperly = true;
          this.displayStatus = true;

          setTimeout(
            () => {
              this.displayStatus = false;
              this.pointsComputedProperly = false;
            },
            2000,
          );
        })
        .catch(() => {
          this.pointsComputedProperly = false;
          this.displayStatus = true;

          setTimeout(
            () => {
              this.displayStatus = false;
              this.pointsComputedProperly = false;
            },
            2000,
          );
        });
    },
  },
};
</script>

<style lang="css">
body {
  color: #4a4a4a;
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.5;
}

html {
  box-sizing: border-box;
  background-color: white;
  font-size: 16px;
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
  min-width: 300px;
  overflow-x: scroll;
  overflow-y: scroll;
  text-rendering: optimizeLegibility;
  -webkit-text-size-adjust: 100%;
  -moz-text-size-adjust: 100%;
  text-size-adjust: 100%;
}

body, button, input, select, textarea {
  font-family: BlinkMacSystemFont, -apple-system, "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue", "Helvetica", "Arial", sans-serif;
}

.hero {
  align-items: stretch;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.hero.is-fullheight {
  min-height: 100vh;
}

*, *::before, *::after {
  box-sizing: inherit;
}

.navbar-yaktoto {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
}

.navbar-left {
  border-bottom: solid;
  border-width: 1px;
  border-color: #53535321;
  padding: 0.5rem;
  grid-column: 2 / 6;
  text-align: left;
}

.navbar-right {
  border-bottom: solid;
  border-width: 1px;
  border-color: #53535321;
  padding: 0.5rem;
  grid-column: 6 / 8;
  text-align: right;
}

.body-yaktoto {
  flex-grow: 1;
  flex-shrink: 0;
  align-items: center;
  text-align: center !important;
  display: grid;
  grid-template-columns: repeat(8, 1fr);
}

.body-yaktoto-page {
  grid-column: 2 / 8;
}

.navbar-item-custom {
  align-items: center;
  padding: 0.5rem 0.75rem;
  color: black;
  display: inline;
  text-decoration: none;
}

.navbar-item-custom.clickable:hover {
  cursor: pointer;
  background-color: #fafafa;
}

.navbar-item-custom.success {
  color: green;
  font-weight: bold;
}

.navbar-item-custom.error {
  color: red;
  font-weight: bold;
}
</style>
