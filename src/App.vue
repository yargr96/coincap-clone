<template>
  <div id="app">
    <table>
      <thead>
        <tr>
          <th>Rank</th>
          <th>Name</th>
          <th>Price</th>
          <th>Market Cap</th>
          <th>Volume (24Hr)</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(item, key) in assets"
          :key="key"
        >
          <td>{{ item.rank }}</td>
          <td>{{ item.name }}</td>
          <td>{{ item.priceUsd }}</td>
          <td>{{ item.marketCapUsd }}</td>
          <td>{{ item.volumeUsd24Hr }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'app',
  data () {
    return {
      assets: {},
      ws: null
    }
  },
  
  created() {
    axios
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
          if (this.assets[item.id].priceUsd >= 1)
            this.assets[item.id].priceUsd = this.assets[item.id].priceUsd.toFixed(2);
        })

        return assets;
      })
      .then(assets => {
        let assetsList = [];
        assets.forEach(item => assetsList.push(item.id));
        assetsList = assetsList.reduce((sum, current, i, arr) => {
          if (i === arr.length - 1)
            return sum + current;
          return sum + current + ',';
        })
        
        console.log(assetsList);
        this.ws = new WebSocket(`wss://ws.coincap.io/prices?assets=${assetsList}`);
        this.ws.onmessage = e => {
          const changes = JSON.parse(e.data);
          console.log(changes);
          for (let key in changes) {
            // changes[key] = +changes[key];
            // if (changes[key] < 1)
            //   changes[key] = changes[key].toFixed(2);
            let price = +changes[key];
            if (price >= 1)
              price = price.toFixed(2);

            this.assets[key].priceUsd = price;
          }
        }
      })
  }
}
</script>

<style lang="scss">

</style>
