<template>
  <div id="app" :class="$style.app">
    <navbar/>
    <hello-metamask/>
    <router-view :class="$style.content"/>
    <br>
    <br>
    <travay-footer />
    <br>
  </div>
</template>

<script>
  import { mapActions, mapMutations, mapGetters } from 'vuex'
  import HelloMetamask from '@/components/hello-metamask'
  import Navbar from '@/components/navbar'
  import TravayFooter from '@/components/travay-footer'
  import SignInModal from './services/SignInModal'
  import { doChangeLocale }  from './util/i18n';
  import {store} from './store';

  export default {
  name: 'App',
  beforeCreate () {
    console.log('registerWeb3 Action dispatched from app.vue');
    this.$store.dispatch('registerWeb3');
    // console.log('contracts instance', this.$store.state.contractInstance());
  },
  components: {
    'navbar': Navbar,
    'hello-metamask': HelloMetamask,
    'travay-footer': TravayFooter,
    'signInModal': SignInModal
  },
  computed: {
    ...mapGetters('signInModal', ['userId'])
  },
  methods: {
    ...mapActions('signInModal', ['openLoginModal', 'saveUserInStorage']),
    signInClicked() {
      this.navBarClose();
      this.openLoginModal();
    },
    localeSwitch(locale) {
      doChangeLocale(locale)
        .catch((error) => console.log(error));

      //this.changeLocale(locale);
      this.navBarClose();
    },
    navBarClose() {
      EventBus.$emit('navbar.close');
    }
  },
  created() {
    try {
      const userData = localStorage.getItem('userData');
      if (userData) {
        this.saveUserInStorage(JSON.parse(userData));
      }
    } catch (err) {
      console.log('err when trying to get user data from storage', err);
    }
  }
}
</script>

<style lang="scss" module>
  @import "./theme/reset";
  @import "./theme/typo";
  @import "./theme/global";
  @import url($google-font);

  .app {
    min-height:     100vh;
    display:        flex;
    flex-direction: column;
  }

  .content {
    flex: 1;
    margin-top: 80px;
  }

</style>
