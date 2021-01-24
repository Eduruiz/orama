<template>
  <div id="app">
    <main>
      <header class="header">
        <div class="grid-container header__content">
          <div class="grid-x">
            <div class="cell">
              <h1>Lista de Fundos de Investimento</h1>
              <p>Conheça nossa lista de fundos</p>
            </div>
          </div>
        </div>
      </header>

      <form>
        <label for="search">
          Search
          <input
          type="search"
          id="search"
          v-model="filters.simple_name.value">
        </label>

        <label for="risk">
          Risk
          <input
            type="number"
            id="risk"
            v-model.number="filters.fund_risk_profile.value">
        </label>

        <label for="initial-application">
          initial-app
          <input
            type="number"
            id="initial-application"
            v-model.number="filters.minimum_initial_application.value">
        </label>
      </form>

      <template >
        <ul>
          <li v-for="(fundValue, fundTitle, key) in groupedFunds" :key="key">
            {{ fundTitle }}
            <ul>
              <li v-for="(internValue, interTitle, key) in fundValue" :key="key">
                {{ interTitle }}
                <ul>
                  <li v-for="(fund, key) in internValue" :key="key">
                    {{ fund.simple_name }}
                  </li>
                </ul>
              </li>
            </ul>
          </li>
        </ul>
      </template>
    </main>
  </div>
</template>

<script>

import _ from 'lodash';

export default {
  name: 'App',
  mounted() {
    this.getData();
  },
  data() {
    return {
      loading: false,
      error: false,

      fundsLimit: 10,
      funds: null,

      filters: {
        simple_name: {
          type: 'string',
          value: null,
          obj_property: 'simple_name',
        },
        fund_risk_profile: {
          type: 'max',
          value: null,
          obj_property: 'specification.fund_risk_profile.score_range_order',
        },
        minimum_initial_application: {
          type: 'max',
          value: null,
          obj_property: 'operability.minimum_initial_application_amount',
        },
        fund_type: {
          type: 'exact_match',
          value: null,
          obj_property: 'specification.fund_macro_strategy.name',
        },
      },
    };
  },
  computed: {
    activeFilters() {
      return Object.values(this.filters)
        .filter((filterObjects) => (filterObjects.value !== null && filterObjects.value !== ''));
    },
    groupedFunds() {
      const fundsGroupedByType = _.groupBy(this
        .filteredFunds, (fund) => fund.specification.fund_type);

      return _.mapValues(fundsGroupedByType, (group) => _.groupBy(group, (fund) => fund
        .specification.fund_main_strategy.name));
    },
    filteredFunds() {
      let filtered;

      if (this.activeFilters.length > 0) {
        filtered = this.funds;

        this.activeFilters.forEach((filter) => {
          if (filter.type === 'string') {
            filtered = filtered
              .filter((fund) => this
                .normalizeString(_.get(fund, filter.obj_property))
                .includes(this.normalizeString(filter.value)));
          }

          if (filter.type === 'max') {
            filtered = filtered
              .filter((fund) => Number(_.get(fund, filter.obj_property)) <= filter.value);
          }

          // if (filter.type === 'exact_match') {
          //   filtered = filtered
          //     .filter((fund) => Number(_.get(fund, filter.obj_property)) === filter.value);
          // }
        });

        return filtered;
      }

      return this.funds;
    },
  },
  methods: {
    getData() {
      const url = `https://s3.amazonaws.com/orama-media/json/fund_detail_full.json?limit=${this.fundsLimit}&offset=0&serializer=fund_detail_full`;

      fetch(url)
        .then(this.error = null)
        .then(this.loading = true)
        .then((response) => response.json())
        .then((jsonData) => {
          this.funds = jsonData;
          return true;
        })
        .then(() => {
          this.loading = false;
          return true;
        })
        .catch((err) => {
          console.error(err);
          this.error = {};
          this.error.message = 'failed to fetch';
          this.error.retryFunction = this.getData;
          this.error.retryText = 'Retry loading funds';
          this.loading = false;
        });
    },
    normalizeString(string) {
      return string.normalize('NFD').replace(/[\u0300-\u036f]/g, '').toLowerCase();
    },
  },
};
</script>

<style lang="sass">
  @import "./assets/scss/index.scss"
</style>
