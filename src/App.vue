<template>
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

    <button class="btn btn-outline-primary" @click="predict">Посчитать</button>

    <h2>Шансы на победу сил света: {{ Math.floor(predictedMatch.chancesForWinRadiant) }}%</h2>
    <h2>Шансы на победу сил тьмы: {{ Math.floor(predictedMatch.chancesForWinDire) }}%</h2>
  </div>

  <select v-model="selectRank.number" @change="getHeroesPercentWins">
    <option value="0" selected></option>
    <option value="1">Рекрут</option>
    <option value="2">Страж</option>
    <option value="3">Рыцарь</option>
    <option value="4">Герой</option>
    <option value="5">Легенда</option>
    <option value="6">Властелин</option>
    <option value="7">Божество</option>
    <option value="8">Титан</option>
  </select>

  <h1>Текущий выбранный ранг: {{ selectRank.name }}</h1>

  <div class="search-div input-group mb-3">
    <input
      class="form-control"
      type="search"
      placeholder="Найти героя"
      v-model="inputSearchHero"
      @input="searchHero"
    />

    <button class="btn btn-outline-primary" @click="getHeroesPercentWins">Clear</button>
  </div>

  <div class="container heroes mb-3">
    <div class="row">
      <heroCard
        v-for="hero in heroesPercentWins"
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
      inputSearchHero: '',
      heroesPercentWins: [],
      selectRank: {
        number: 0,
        name: '',
      },

      predictedMatch: {
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
      }).then((result) => {
        if (result.isConfirmed) {
          this.predictedMatch.selectedHeroesRadiant.push(hero);
        } else {
          this.predictedMatch.selectedHeroesDire.push(hero);
        }
      });
    },

    async getHeroesPercentWins() {
      switch (this.selectRank.number) {
        case '1':
          this.selectRank.name = 'Рекрут';
          break;
        case '2':
          this.selectRank.name = 'Страж';
          break;
        case '3':
          this.selectRank.name = 'Рыцарь';
          break;
        case '4':
          this.selectRank.name = 'Герой';
          break;
        case '5':
          this.selectRank.name = 'Легенда';
          break;
        case '6':
          this.selectRank.name = 'Властелин';
          break;
        case '7':
          this.selectRank.name = 'Божество';
          break;
        case '8':
          this.selectRank.name = 'Титан';
          break;
      }

      await axios.get('https://api.opendota.com/api/heroStats').then((heroesStats) => {
        this.heroesPercentWins = [];

        heroesStats.data.forEach((heroStats) => {
          this.heroesPercentWins.push({
            heroName: heroStats.localized_name,
            percentWins: Math.floor(
              (heroStats[`${this.selectRank.number}_win`] /
                heroStats[`${this.selectRank.number}_pick`]) *
                100,
            ),
          });
        });
      });
    },

    searchHero() {
      if (this.inputSearchHero === '') {
        this.getHeroesPercentWins();
      }

      this.heroesPercentWins = this.heroesPercentWins.filter(
        (hero) =>
          hero.heroName
            .toLowerCase()
            .replace(/\s/g, '')
            .search(this.inputSearchHero.toLowerCase().replace(/\s/g, '')) != -1,
      );
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
</style>
