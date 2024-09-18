<template>
  <div id="app">
    <div class="search-bar">
      <input type="text" v-model="query" placeholder="Inserisci il nome del film o della serie TV" />
      <button @click="searchMoviesAndSeries">Cerca</button>
    </div>
    <div id="results">
      <div v-if="moviesAndSeries.length === 0 && !loading">Nessun film o serie TV trovati.</div>
      <div v-if="loading">Caricamento in corso...</div>
      <div v-for="item in moviesAndSeries" :key="item.id" class="media-item">
        <img :src="getPosterUrl(item.poster_path)" alt="Copertina" class="poster" />
        <h3>{{ item.title }}</h3>
        <p><strong>Titolo:</strong> {{ item.original_title }}</p>
        <p><strong>Lingua:</strong> <img :src="getFlagUrl(item.original_language)" :alt="item.original_language" class="flag" /></p>
        <p><strong>Voto:</strong> 
          <span v-html="getStars(item.vote_average)"></span>
        </p>
        <p><strong>Tipo:</strong> {{ item.media_type === 'movie' ? 'Film' : 'Serie TV' }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      query: '',
      moviesAndSeries: [],
      loading: false,
    };
  },
  methods: {
    async searchMoviesAndSeries() {
      if (!this.query) return;
      this.loading = true;

      const apiKey = '4f7d8fd0260e38a057cf0c8401bb109f';
      const movieUrl = `https://api.themoviedb.org/3/search/movie?api_key=${apiKey}&query=${encodeURIComponent(this.query)}`;
      const tvUrl = `https://api.themoviedb.org/3/search/tv?api_key=${apiKey}&query=${encodeURIComponent(this.query)}`;

      try {
        const [movieResponse, tvResponse] = await Promise.all([
          fetch(movieUrl),
          fetch(tvUrl)
        ]);

        const movies = await movieResponse.json();
        const tvSeries = await tvResponse.json();

        const mappedMovies = movies.results.map(movie => ({
          id: movie.id,
          title: movie.title || movie.name,
          original_title: movie.original_title || movie.original_name,
          original_language: movie.original_language,
          vote_average: movie.vote_average,
          poster_path: movie.poster_path,
          media_type: 'movie'
        }));

        const mappedTvSeries = tvSeries.results.map(tv => ({
          id: tv.id,
          title: tv.name,
          original_title: tv.original_name,
          original_language: tv.original_language,
          vote_average: tv.vote_average,
          poster_path: tv.poster_path,
          media_type: 'tv'
        }));

        this.moviesAndSeries = [...mappedMovies, ...mappedTvSeries];
      } catch (error) {
        console.error('Errore nella ricerca:', error);
      } finally {
        this.loading = false;
      }
    },

    getPosterUrl(posterPath) {
      const baseUrl = 'https://image.tmdb.org/t/p/';
      const size = 'w500';
      return posterPath ? `${baseUrl}${size}${posterPath}` : 'https://via.placeholder.com/500x750?text=Nessuna+Immagine';
    },

    getFlagUrl(languageCode) {
      const langToCountry = {
        en: 'us',
        it: 'it',
        fr: 'fr',
        es: 'es',
        de: 'de',
        ja: 'jp',
        ko: 'kr',
        zh: 'cn',
        pt: 'pt',
        ru: 'ru'
      };

      const countryCode = langToCountry[languageCode] || 'unknown';
      return countryCode !== 'unknown'
        ? `https://flagcdn.com/16x12/${countryCode}.png`
        : 'https://via.placeholder.com/16x12?text=?';
    },

    getStars(voteAverage) {
      const voteInFive = Math.ceil(voteAverage / 2);
      let starsHtml = '';

      for (let i = 0; i < 5; i++) {
        if (i < voteInFive) {
          starsHtml += '<i class="fas fa-star"></i>';
        } else {
          starsHtml += '<i class="far fa-star"></i>';
        }
      }

      return starsHtml;
    }
  }
};
</script>

<style lang="scss">
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css');

#app {
  max-width: 800px;
  margin: 0 auto;
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

    .media-item {
      display: flex;
      align-items: center;
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

      .poster {
        width: 100px;
        height: 150px;
        object-fit: cover;
        margin-right: 15px;
      }

      .flag {
        width: 16px;
        height: 12px;
        margin-left: 5px;
      }

      i.fas.fa-star, i.far.fa-star {
        color: #FFD700;
        margin-right: 2px;
      }
    }
  }
}
</style>
