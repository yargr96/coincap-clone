<template>
  <div id="app" class="app">
    <div 
      class="assets-table"
    >
      <div class="assets-table__head">
        <div class="assets-table__cell assets-table__cell_rank">Rank</div>
        <div class="assets-table__cell assets-table__cell_name">Name</div>
        <div class="assets-table__cell assets-table__cell_price">Price</div>
        <div class="assets-table__cell assets-table__cell_market">Market Cap</div>
        <div class="assets-table__cell assets-table__cell_volume">Volume (24Hr)</div>
      </div>
      <div class="assets-table__body-wrapper">
        <div 
          class="assets-table__body"
          @mouseenter="onMouseEnter"
          @mouseleave="onMouseLeave"
          @scroll="onScroll"
        >
          <div class="assets-table__row"
            v-for="(item, key) in assetsFormated"
            :key="key"
          >
            <div class="assets-table__cell assets-table__cell_rank">{{ item.rank }}</div>
            <div class="assets-table__cell assets-table__cell_name">{{ item.name }}</div>
            <div class="assets-table__cell assets-table__cell_price">${{ item.priceUsd }}</div>
            <div class="assets-table__cell assets-table__cell_market">${{ item.marketCapUsd }}</div>
            <div class="assets-table__cell assets-table__cell_volume">${{ item.volumeUsd24Hr }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'app',

  data () {
    return {
      assets: {},
      ws: null,
      tableHovered: false,
      scrollProps: {
        scrollHeight: null,
        scrollTop: null,
        clientHeight: null,
      }
    }
  },

  computed: {
    assetsFormated() {
      let formated = {};
      for (let key in this.assets) {
        formated[key] = {};
        formated[key].rank = this.assets[key].rank;
        formated[key].name = this.assets[key].name;
        formated[key].priceUsd = formatNumber(this.assets[key].priceUsd);
        formated[key].marketCapUsd = formatNumber(this.assets[key].marketCapUsd);
        formated[key].volumeUsd24Hr = formatNumber(this.assets[key].volumeUsd24Hr);
      }

      return formated;
    },
  },

  methods: {
    getAssets() {
      return axios
      .get('https://api.coincap.io/v2/assets?limit=15')
      .then(res => {
        console.log(res.data.data);
        const assets = res.data.data;
        assets.forEach(item => {
          this.$set(this.assets, item.id, {
            rank: item.rank,
            name: item.name,
            priceUsd: +item.priceUsd,
            marketCapUsd: +item.marketCapUsd,
            volumeUsd24Hr: +item.volumeUsd24Hr,
          });
        })

        return assets;
      })
    },
    updatePrice(changes) {
      for (let key in changes) {
        this.assets[key].priceUsd = changes[key];
      }
    },
    onMouseEnter(e) {
      this.tableHovered = true;
      e.target.style.maxHeight = "auto";
    },
    onMouseLeave(e) {
      this.tableHovered = false;
    },
    onScroll() {
      console.log("SCROLL");
    }
  },
  
  created() {
    this.getAssets()
      .then(assets => {
        let assetsList = [];
        assets.forEach(item => assetsList.push(item.id));
        assetsList = assetsList.reduce((sum, current, i, arr) => {
          if (i === arr.length - 1)
            return sum + current;
          return sum + current + ',';
        })
        
        this.ws = new WebSocket(`wss://ws.coincap.io/prices?assets=${assetsList}`);
        this.ws.onmessage = e => this.updatePrice(JSON.parse(e.data));
      })
  },
}

function formatNumber(num) {
  num = +num;
  if (isNaN(num))
    throw new Error("Неверный формат числа");

  if (num <= 0)
    num = "0.00";
  else if (num < 1)
    num = num.toFixed(8);
  else if (num < 1e6)
    num = num.toFixed(2);
  else if (num < 1e9)
    num = (num / 1e6).toFixed(2) + "m";
  else if (num >= 1e9)
    num = (num / 1e9).toFixed(2) + "b";

  return num;
}
</script>

<style lang="scss">
  $head-height: 40px;

  * {
    font-family: sans-serif;
    box-sizing: border-box;
  }

  html, body {
    margin: 0;
    padding: 0;
  }

  .app {
    width: 100%;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 0 10px;
  }

  .assets-table {
    width: 100%;
    max-width: 740px;
    box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 15px -3px;
    color: rgba(0, 0, 0, 0.9);
    text-align: right;
    border-radius: 3px;
    position: relative;
    padding-top: $head-height;
    height: 90vh;

    &__body {
      width: 740px;

      &-wrapper {
        max-height: 100%;
        overflow-y: hidden;
        overflow-x: hidden;

        &:hover {
          overflow-y: auto;
        }
      }
    }

    &__head, &__row {
      display: flex;
    }

    &__head {
      background-color: #eee;
      color: rgba(0, 0, 0,.6);
      border-radius: 3px 3px 0 0;
      position: absolute;
      left: 0;
      top: 0;
      right: 0;
      z-index: 10;
      height: $head-height;
      align-items: center;
      padding: 0 20px 0 10px;
    }

    &__row {
      padding: 40px 20px 40px 10px;
      border-bottom: 1px solid rgba(0, 0, 0,.05);
    }

    &__cell {
      &_rank {
        width: 15%;
        text-align: center;
      }
      &_name {
        width: 25%;
        text-align: left;
      }
      &_price {
        width: 20%;
      }
      &_market {
        width: 20%;
      }
      &_volume {
        width: 20%;
      }
    }
  }

  .fade-enter-active, .fade-leave-active {
    transition: opacity .2s;
  }

  .fade-enter, .fade-leave-to {
    opacity: 0;
  }

  @media (min-width: 576px) and (max-width: 767px) {
    .assets-table {
      max-width: 540px;

      &__body {
        width: 540px;
      }
    }
  }

  @media (max-width: 575px) {
    .assets-table__body {
      width: 100%;
    }

    .assets-table__cell {
      &_rank {
        display: none;
      }
      &_market {
        display: none;
      }
      &_volume {
        display: none;
      }
      &_name {
        width: 50%;
        text-align: center;
      }
      &_price {
        width: 50%;
        text-align: center;
      }
    }
  }
</style>
