<template>
  <div id="app">
    <div class="search-bar">
      <input type="text" v-model="query" placeholder="Inserisci il film" />
      <button @click="searchMovies">Cerca</button>
    </div>
    <div id="result">
      <div v-if="movies.length === 0 && !loading">Nessun film trovato.</div>
      <div v-if="loading">Caricamento in corso...</div>
      <div v-for="movie in movies" :key="movie.id" class="movie">
        <h3>{{ movie.title }}</h3>
        <p><strong>Titolo:</strong> {{ movie.original_title }}</p>
        <p><strong>Lingua:</strong> {{ movie.original_language }}</p>
        <p><strong>Voto:</strong> {{ movie.vote_average }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      query: '',
      movies: [],
      loading: false,
    };
  },
  methods: {
    async searchMovies() {
      if (!this.query) return;
      this.loading = true;
      const apiKey = '4f7d8fd0260e38a057cf0c8401bb109f';
      const url = `https://api.themoviedb.org/3/search/movie?api_key=${apiKey}&query=${encodeURIComponent(this.query)}`;

      try {
        const response = await fetch(url);
        const data = await response.json();
        this.movies = data.results;
      } catch (error) {
        console.error('Errore nella ricerca dei film:', error);
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style lang="scss">
#app {
  font-family: Arial, sans-serif;
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;

  .search-bar {
    margin-bottom: 20px;

    input {
      padding: 8px;
      font-size: 16px;
    }

    button {
      padding: 8px 12px;
      font-size: 16px;
      margin-left: 8px;
      cursor: pointer;
    }
  }

  #results {
    max-width: 600px;
    width: 100%;

    .movie {
      border: 1px solid #ddd;
      padding: 10px;
      margin-bottom: 10px;
      border-radius: 5px;

      h3 {
        margin: 0;
        font-size: 18px;
      }

      p {
        margin: 5px 0;
      }
    }
  }
}
</style>
