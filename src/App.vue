<template>
  <div class="container">
    <div id="autocomplete" />
  </div>
</template>

<script lang="jsx">
import { h, Fragment, render, onMounted } from 'vue';
import { autocomplete, getAlgoliaResults } from '@algolia/autocomplete-js';
import algoliasearch from 'algoliasearch/lite';
import { createQuerySuggestionsPlugin } from '@algolia/autocomplete-plugin-query-suggestions';

import '@algolia/autocomplete-theme-classic';

import { createElement } from './utils/createElement';
import ProductItem from './components/ProductItem.vue';

const appId = 'latency';
const apiKey = '6be0576ff61c053d5f9a3225e2a90f76';
const searchClient = algoliasearch(appId, apiKey);

const querySuggestionPluginInCategory = createQuerySuggestionsPlugin({
  searchClient,
  indexName: 'instant_search_demo_query_suggestions',
  transformSource({ source }) {
    return {
      ...source,
      onSelect({ setIsOpen }) {
        setIsOpen(true);
      },
    };
  },
  });
console.log(222222);


export default {
  name: 'App',
  setup() {
    onMounted(() => {
      autocomplete({
        plugins: [querySuggestionPluginInCategory],
        container: '#autocomplete',
        placeholder: 'Search',
        getSources({ query }) {
          return [
            {
              sourceId: 'products',
              getItems() {
                return getAlgoliaResults({
                  searchClient,
                  queries: [
                    {
                      indexName: 'instant_search',
                      query,
                    },
                  ],
                });
              },
              templates: {
                item({ item, components }) {
                  return (
                    <ProductItem item={item} highlight={components.Highlight} />
                  );
                },
              },
            },
          ];
        },
        renderer: {
          createElement,
          Fragment,
          render,
        },
      });
    });
  },
};
</script>

<style>
.container {
  margin: 0 auto;
  max-width: 640px;
  width: 100%;
}
</style>
