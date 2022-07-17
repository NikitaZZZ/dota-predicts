<!-- Todo: delete hero from side -->
<template>
  <h1>Текущий выбранный ранг: {{ userInformation.selectRank.name }}</h1>
  <select
    class="form-select mb-3"
    id="choose-rank"
    v-show="userInformation.selectRank.number === 0"
    v-model="userInformation.selectRank.number"
    @change="getHeroesPercentWins"
  >
    <option v-for="(rank, index) in userInformation.ranks" :key="rank" :value="index">
      {{ rank }}
    </option>
  </select>

  <div
    class="pick"
    v-show="
      predictedMatch.selectedHeroesRadiant.length !== 0 ||
      predictedMatch.selectedHeroesDire.length !== 0
    "
  >
    <div class="container">
      <h3>Силы света</h3>

      <div class="container heroes mb-3">
        <div class="row">
          <heroCard
            v-for="hero in predictedMatch.selectedHeroesRadiant"
            :key="hero.heroName"
            :heroStats="hero"
          />
        </div>
      </div>
    </div>

    <div class="container mb-3">
      <h3>Силы тьмы</h3>

      <div class="container heroes mb-3">
        <div class="row">
          <heroCard
            v-for="hero in predictedMatch.selectedHeroesDire"
            :key="hero.heroName"
            :heroStats="hero"
          />
        </div>
      </div>

      <button class="btn btn-outline-primary mb-3" @click="predict">Посчитать</button>

      <h2 v-show="Math.floor(predictedMatch.chancesForWinRadiant) != 0">
        Шансы на победу сил света: {{ Math.floor(predictedMatch.chancesForWinRadiant) }}%
      </h2>
      <h2 v-show="Math.floor(predictedMatch.chancesForWinDire) != 0">
        Шансы на победу сил тьмы: {{ Math.floor(predictedMatch.chancesForWinDire) }}%
      </h2>
    </div>
  </div>

  <div
    class="search-div input-group mb-3"
    v-show="
      userInformation.selectRank.number != 0 &&
      Math.floor(predictedMatch.chancesForWinRadiant) === 0
    "
  >
    <input
      class="form-control"
      type="search"
      placeholder="Найти героя"
      v-model="userInformation.inputSearchHero"
    />

    <button class="btn btn-outline-primary" @click="getHeroesPercentWins">Clear</button>
  </div>

  <div class="container heroes mb-3" v-show="Math.floor(predictedMatch.chancesForWinRadiant) === 0">
    <div class="row">
      <heroCard
        v-for="hero in searchHero"
        :key="hero.heroName"
        :heroStats="hero"
        @click="selectHero(hero)"
      />
    </div>
  </div>
</template>

<script>
import Swal from 'sweetalert2';
import axios from 'axios';
import heroCard from './components/heroCard.vue';

export default {
  name: 'App',
  components: { heroCard },
  data() {
    return {
      userInformation: {
        ranks: [
          'Выберите ранг',

          'Рекрут',
          'Страж',
          'Рыцарь',
          'Герой',
          'Легенда',
          'Властелин',
          'Божество',
          'Титан',
        ],

        inputSearchHero: '',

        selectRank: {
          number: 0,
          name: '',
        },
      },

      predictedMatch: {
        heroesPercentWins: [],

        selectedHeroesRadiant: [],
        selectedHeroesDire: [],

        chancesForWinRadiant: 0,
        chancesForWinDire: 0,

        addedToRadiant: false,
        addedToDire: false,
      },
    };
  },

  methods: {
    predict() {
      let sumWinRadiant = 0;
      let sumWinDire = 0;

      this.predictedMatch.selectedHeroesRadiant.forEach((hero) => {
        sumWinRadiant = sumWinRadiant + hero.percentWins;
      });

      this.predictedMatch.selectedHeroesDire.forEach((hero) => {
        sumWinDire = sumWinDire + hero.percentWins;
      });

      Swal.fire({
        title: 'За какую сторону?',
        showDenyButton: true,
        confirmButtonText: 'Силы света',
        denyButtonText: 'Силы тьмы',
        allowOutsideClick: false,
        allowEscapeKey: false,
      }).then((result) => {
        if (result.isConfirmed) {
          this.predictedMatch.chancesForWinRadiant = (sumWinRadiant + 7) / 5;
          this.predictedMatch.chancesForWinDire = (sumWinDire - 7) / 5;
        } else {
          this.predictedMatch.chancesForWinRadiant = sumWinRadiant / 5;
          this.predictedMatch.chancesForWinDire = sumWinDire / 5;
        }
      });
    },

    selectHero(hero) {
      Swal.fire({
        title: 'Какая сторона?',
        showDenyButton: true,
        confirmButtonText: 'Силы света',
        denyButtonText: 'Силы тьмы',
        allowOutsideClick: false,
        allowEscapeKey: false,
      }).then((result) => {
        if (result.isConfirmed) {
          this.predictedMatch.selectedHeroesRadiant.push(hero);
        } else {
          this.predictedMatch.selectedHeroesDire.push(hero);
        }
      });
    },

    getHeroesPercentWins() {
      const selectRank = this.userInformation.selectRank;
      const ranks = this.userInformation.ranks;

      selectRank.name = ranks[selectRank.number];

      axios.get('https://api.opendota.com/api/heroStats').then((heroesStats) => {
        this.predictedMatch.heroesPercentWins = [];

        heroesStats.data.forEach((heroStats) => {
          const percentWins = Math.floor(
            (heroStats[`${selectRank.number}_win`] / heroStats[`${selectRank.number}_pick`]) * 100,
          );

          this.predictedMatch.heroesPercentWins.push({
            heroName: heroStats.localized_name,
            percentWins: percentWins,
          });
        });
      });
    },
  },

  computed: {
    searchHero() {
      return this.predictedMatch.heroesPercentWins.filter((hero) => {
        const heroName = hero.heroName.toLowerCase().replace(/\s/g, '');
        const inputSearchHero = this.userInformation.inputSearchHero
          .toLowerCase()
          .replace(/\s/g, '');

        return heroName.search(inputSearchHero) != -1;
      });
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.search-div {
  width: 300px;
  margin: auto;
}

#choose-rank {
  width: 400px;
  margin: auto;
}

@media screen and (max-width: 500px) {
  .heroes {
    width: 200px;
    margin: auto;
  }
}
</style>
