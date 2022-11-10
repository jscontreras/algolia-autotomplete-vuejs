<template>
  <div class="container">
    <div id="autocomplete" />
  </div>
</template>

<script lang="jsx">
import { h, Fragment, render, onMounted } from "vue";
import { autocomplete, getAlgoliaResults } from "@algolia/autocomplete-js";
import algoliasearch from "algoliasearch/lite";
// Import Plugins
import { createQuerySuggestionsPlugin } from "@algolia/autocomplete-plugin-query-suggestions";
import { createLocalStorageRecentSearchesPlugin } from "@algolia/autocomplete-plugin-recent-searches";

import "@algolia/autocomplete-theme-classic";

import { createElement } from "./utils/createElement";
import ProductItem from "./components/ProductItem.vue";

const appId = "latency";
const apiKey = "6be0576ff61c053d5f9a3225e2a90f76";
const searchClient = algoliasearch(appId, apiKey);

// Query Suggestion Plugin
const querySuggestionPluginInCategory = createQuerySuggestionsPlugin({
  searchClient,
  indexName: "instant_search_demo_query_suggestions",
  transformSource({ source }) {
    return {
      ...source,
      onSelect({ setIsOpen }) {
        setIsOpen(true);
      },
    };
  },
});

// Recent Searches Plugin
const recentSearchesPlugin = createLocalStorageRecentSearchesPlugin({
  key: "navbar",
});

export default {
  name: "App",
  setup() {
    onMounted(() => {
      autocomplete({
        // Add Plugins
        plugins: [querySuggestionPluginInCategory, recentSearchesPlugin],
        container: "#autocomplete",
        placeholder: "Search for Products",
        getSources({ query }) {
          return [
            {
              sourceId: "products",
              getItems() {
                return getAlgoliaResults({
                  searchClient,
                  queries: [
                    {
                      indexName: "instant_search",
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
