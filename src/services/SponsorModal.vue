<template>
  <div :class="$style.sponsorModal">
    <vue-modal :show="show" @close="$emit('update:show', false)">
      <vue-button warn @click="$emit('update:show', false)">X</vue-button>
      <vue-input
        name="sponsorAmount"
        id="sponsorAmount"
        required
        placeholder="Enter Amount"
        validation="required"
        type="number"
        v-model="sponsorAmount"/>

      <vue-button primary @click.prevent.stop="clickHandler">Sponsor</vue-button>
    </vue-modal>
  </div>
</template>

<script>
  import {store} from '../store';
  import truffleContract from "truffle-contract";
  import EscrowContract from "../../contracts/build/contracts/Escrow.json";
  import DAIContract from "../../contracts/build/contracts/DAI.json";

  export default {
    name: 'SponsorModal',
    props: {
      show: {
        type: Boolean,
        required: true,
        default: false
      },
      job: {
        type: Object,
        required: true,
        default: () => {
        }
      }
    },
    data() {
      return {
        sponsorAmount: ''
      };
    },
    computed: {},
    methods: {
      clickHandler() {
        this.$emit('sponsorSubmit', this.sponsorAmount);
        this.sponsorAmount = '';
        this.sponsorAmountToEscrow();
      },
      async sponsorAmountToEscrow() {
        const Escrow = truffleContract(EscrowContract);
        const DAI = truffleContract(DAIContract);

        window.Escrow = Escrow;
        Escrow.setProvider(this.$store.state.web3.web3Instance().currentProvider);
        Escrow.defaults({from: this.$store.state.web3.web3Instance().eth.coinbase});
        DAI.setProvider(this.$store.state.web3.web3Instance().currentProvider);

        const EscrowInstance = await Escrow.deployed();
        const DAIInstance = await DAI.deployed();

        window.EscrowInstance = EscrowInstance;
        const pool = EscrowInstance.address;

        DAI.setProvider(this.$store.state.web3.web3Instance().currentProvider);
        DAI.defaults({from: this.$store.state.web3.web3Instance().eth.coinbase});

        const JobID = 0; // you need to get this from UI
        const payment = 10 * (10 ** 18);  // need to get token number from UI

        web3.eth.getAccounts( async(err, accounts) => {
          const sponsor = accounts[0];
          try {
            //await DAIInstance.transfer(pool, payment);

            await DAIInstance.approve(EscrowInstance.address, payment, {from :sponsor});

            //await EscrowInstance.register({from: sponsor});
            const result = await EscrowInstance.sponsorDAI(JobID, payment, {
              from: sponsor
            });

          } catch (err) {
            console.log(false, err);
          }
        })
      }
    }
  };
</script>

<style lang="scss" module>
  .sponsorModal {
    display: block;
  }
</style>
