<template>
  <div>
    <AisInstantSearchSsr>
      <AisSearchBox />
      <AisHits />
    </AisInstantSearchSsr>
  </div>
</template>

<script>
import {
  AisInstantSearchSsr,
  AisSearchBox,
  AisHits,
  createServerRootMixin,
} from 'vue-instantsearch/vue3/es';
import algoliasearch from 'algoliasearch/lite';
import { renderToString } from '@vue/server-renderer';
import { createSSRApp } from 'vue';

const searchClient = algoliasearch(
  'latency',
  '6be0576ff61c053d5f9a3225e2a90f76'
);

function $cloneComponent(componentInstance, { mixins = [] } = {}) {
  const options = {
    serverPrefetch: undefined,
    fetch: undefined,
    _base: undefined,
    name: 'ais-ssr-root-component',
  };

  let app;

  const appOptions = Object.assign({}, componentInstance.$options, options);

  appOptions.mixins = [
    {
      beforeCreate() {
        this.$nuxt = componentInstance.$nuxt;
      },
    },
    ...appOptions.mixins,
    ...mixins,
  ];

  app = createSSRApp(appOptions);
  if (componentInstance.$router) {
    app.use(componentInstance.$router);
  }
  if (componentInstance.$store) {
    app.use(componentInstance.$store);
  }

  // https://stackoverflow.com/a/48195006/3185307
  app.$slots = componentInstance.$slots;
  app.$root = componentInstance.$root;

  return app;
}

export default {
  components: { AisInstantSearchSsr, AisSearchBox, AisHits },
  mixins: [
    createServerRootMixin({
      indexName: 'instant_search',
      searchClient,
      $cloneComponent,
    }),
  ],
  serverPrefetch() {
    return this.instantsearch
      .findResultsState({
        component: this,
        renderToString,
      })
      .then((algoliaState) => {
        this.$nuxt.payload.data.algoliaState = algoliaState;
      });
  },
  beforeMount() {
    const results = this.$nuxt.payload.data.algoliaState;

    this.instantsearch.hydrate(results);
  },
};
</script>
