<template>
<!-- https://codepen.io/AndrewThian/pen/QdeOVa -->
<div class="">
    <div class="input-group">
        <input @keydown="onInputChangeOrKeyDown" @change="onInputChangeOrKeyDown" v-model="searchBarInputString" class="searchBar border form-control" :placeholder="searchBarPlaceholderText" />
    </div>

    <div class="search-results-container hideSearchResults">
        <div>
            <button class="buttonHideSearchResults" @click="hideSearchResultsContainerAndRemoveSearchStringFromURL"><span class="visuallyhidden">Close search results</span>×</button>
            <h1 class="hideSearchResults m-3 mt-5 display-5 text-center">{{everythingAbout }} “{{searchBarInputString}}”</h1>

            <!-- SEARCH RESULT COPY BUTTON -->
            <p class="text-center">
                {{searchResultsCopyOption}} <button class="copyURLtoClipboard copyURLtoClipboardCardFromAddressBar" style="height: 1em;vertical-align: top;" title="Copy Link">Copy Link</button>
            </p>

            <div class="search-results" v-for="card in computedSearchCards" :key="card.Misconception" @click="$store.commit('showCardIntroFromURL', card['Unique URL'])">

                <!-- copy card URL to clipboard -->
                <button onclick="return false;" :data-misconception="card['Misconception']" :data-url="'card/' + card['Unique URL']" class="copyURLtoClipboard copyURLtoClipboard6 " style="float: right; width: 2em; height: 2em;vertical-align: top;" title="Copy Link">Copy Link</button>

                <!-- category -->
                <small @click="hideSearchResultsContainer" class="search-result category mb-4 pt-1 pl-2 pr-2 pb-0 display-5" :class="card.Category" style="display: inline-block; border-radius: 10px;">
                    <router-link :to="'/card/' + card['Unique URL']">{{card.Category}}</router-link>
                </small>
                <!-- misconception -->
                <h2 @click="hideSearchResultsContainer" style="cursor: pointer" class="w-1/4">
                    <router-link class="search-result" :to="'/card/' + card['Unique URL']">{{ card.Misconception }}</router-link>
                </h2>

                <!-- misconception longer -->
                <p v-for="item in card['Misconception Elaborate']" v-bind:key="item" @click="hideSearchResultsContainer" class="search-result ml-4 w-3/4" style="cursor: pointer">
                    {{ item }}
                </p>

                <div v-if="(card['Youtube Video Id'])" class="clearfix">
                    <!-- https://stackoverflow.com/a/2068371/9749918 -->
                    <img @click="hideSearchResultsContainer" style="cursor: pointer;" class="rounded mx-auto d-block img-fluid" :src="'https://img.youtube.com/vi/' + card['Youtube Video Id'] + '/0.jpg'" alt="">
                </div>

                <!-- <video style="max-width: 100%; width: 100%;" v-if="card['Self Hosted Video']" :src="path + card['Self Hosted Video']" controls preload="none" playsinline></video> -->
                <!-- <video style="max-width: 100%; width: 100%;" v-if="card['Self Hosted Video']" :src="#" :data-src="path + card['Self Hosted Video']" controls preload="none" playsinline></video> -->

                <!-- go to card -->
                <p @click="hideSearchResultsContainer" style="cursor: pointer" class="w-1/4">
                    <router-link class="search-result" :to="'/card/' + card['Unique URL']">{{goToCard}}</router-link>
                </p>

                <!-- result snippet -->
                <p @click="hideSearchResultsContainer" class="search-result" style="font-size: 1em;" v-html="card.searchResultSnippet">
                    <router-link :to="'/card/' + card['Unique URL']">{{card.searchResultSnippet}}</router-link>
                </p>
            </div>
        </div>
    </div>
</div>
</template>

<script>
import store from "../store/store";
import {
    disableBodyScrollMixin
} from "./mixins/disableBodyScroll";

export default {
    name: "Search",
    mixins: [disableBodyScrollMixin],
    components: {},
    data() {
        return {
            searchBarInputString: '',
            cards: [],
            searchResultSnippet: "",
            goToCard: language.searchGoToCard,
            searchBarPlaceholderText: language.searchBarPlaceholderText,
            everythingAbout: language.everythingAbout,
            searchResultsCopyOption: language.searchResultsCopyOption,
            path: process.env.VUE_APP_MEDIA_LOCATION
        }
    },
    computed: {
        getCards: function () {
            return this.$store.state.theJSON;
        },
        computedSearchCards: function () {
            return this.searchCards();
        },
        // not used ATM
        routeQuery: function () {
            // return this.$route;
            // return this.$route.query.search;
            return this.$route.query;
        }
    },
    watch: {
        getCards(newValue, oldValue) {
            this.cards = newValue;
        },
        // not used ATM
        routeQuery(newValue, oldValue) {
            
            if (this.searchBarInputString !== undefined) {
                this.showSearchResultsContainer();
            } else {
                this.hideSearchResultsContainer;
            }

            console.log('this.searchBarInputString NU IN GEBRUIK: ', this.searchBarInputString);
            // console.log('this.$route.params.search GAAN GEBRUIKEN: ', this.$route.params.search);
            // console.log('this.$route.query GAAN GEBRUIKEN: ', this.$route.query.search);

        }
    },
    mounted: function () {
        this.disableBodyScroll(".search-results-container"); //mixin
        this.setSearchTermFromUrlQueryParams();

        // https://shubhamjain.co/til/vue-shortcuts/
        this._keyListener = function (e) {
            if (e.key === "Escape") {
                e.preventDefault();
                this.hideSearchResultsContainerAndRemoveSearchStringFromURL();
            }
        };
        document.addEventListener('keydown', this._keyListener.bind(this));
    },
    beforeDestroy() {
        document.removeEventListener('keydown', this._keyListener);
    },
    methods: {
        hideSearchResultsContainerAndRemoveSearchStringFromURL() {
            this.hideSearchResultsContainer();
            // the router push should not run inside hideSearchResultsContainer because it does not play well with the URL handling when opening a card
            this.$router.push("/");
        },
        onInputChangeOrKeyDown() {
            this.searchCards();
            this.showSearchResultsContainer();
        },
        searchCards() {
            var allKeys = this.$store.state.allKeys;

            // this updates the URL with what is entered in search field
            // router should only push when search is not empty, to avoid a redirect to /search
            if (this.searchBarInputString !== undefined) {
                if (this.searchBarInputString !== "") {
                    this.$router.push({
                        // path: '/',
                        query: {
                            search: this.searchBarInputString.toLowerCase() // https://forum.vuejs.org/t/how-to-remove-one-query-string-from-url/39176/3
                        }
                    }).catch(err => {}) //https://stackoverflow.com/a/58747480

                    return this.cards.filter(card => {
                        var results = false;

                        // go through all columns, Misconception, Misconception Elaborate, Short Answer etc
                        for (let i = 0; i < allKeys.length; i++) {
                            // if (typeof card[allKeys[i]] === "string" && card[allKeys[i]] !== "") {
                            if (card[allKeys[i]] !== undefined) {

                                // NOTE: the search is done in almost all columns, except the ones where there is created an array out of strings separated by commas
                                if (card[allKeys[i]].toString().toLowerCase().includes(this.searchBarInputString.toLowerCase()) === true) {
                                    // if a match is found, then this entry should be shown
                                    // https://stackoverflow.com/a/494046
                                    var replace = this.searchBarInputString;
                                    var re = new RegExp((replace), "gi");
                                    card.searchResultSnippet = card[allKeys[i]].toString().replace(re, "<em>" + replace + "</em>");

                                    // remove links
                                    // https://stackoverflow.com/a/960178
                                    card.searchResultSnippet = card.searchResultSnippet.replace(/<a\b[^>]*>(.*?)<\/a>/i, "");

                                    // remove URLs
                                    // https://stackoverflow.com/a/23571059
                                    card.searchResultSnippet = card.searchResultSnippet.replace(/(?:https?|ftp):\/\/[\n\S]+/g, '');

                                    results = true;
                                }
                            }
                        }
                        return results;
                    })
                }
            }
        },
        // Sends the URL query parameter value to the search input field
        setSearchTermFromUrlQueryParams() {
            this.searchBarInputString = this.$route.query.search;

            if (this.searchBarInputString !== undefined) {
                this.searchCards();
                document.querySelector('.navbar-toggler').click();
                this.showSearchResultsContainer();
            }
        },
        hideSearchResultsContainer() {
            var searchResultsContainer = document.querySelector(".search-results-container");
            var searchResultsContainerH1 = document.querySelector(".search-results-container h1");

            if (searchResultsContainer !== null) {
                searchResultsContainer.classList.add('hideSearchResults');
            }
            if (searchResultsContainerH1 !== null) {
                searchResultsContainerH1.classList.add('hideSearchResults');
            }
        },
        showSearchResultsContainer() {
            document.querySelector(".search-results-container").classList.remove('hideSearchResults');
            document.querySelector(".search-results-container h1").classList.remove('hideSearchResults');
        }
    }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->

<style lang="scss" scoped>
.searchBar {
    background: $search-bar-background;
    color: $search-bar-color !important;
}

.searchBar:focus {
    background: $search-bar-focus-background;
    color: $search-bar-focus-color;

}

.search-results {
    padding: 2em;
    border: 1px dashed $search-results-color;
    border-radius: 10px;
    margin: 2em auto;
    max-width: 50em;
}

.search-results a {
    color: $search-results-color;
}

.search-results-container {
    position: fixed;
    top: 10em;
    padding-bottom: 12em;
    padding-left: 1em;
    padding-right: 1em;
    left: 0;
    font-size: 1em;
    color: $search-results-color;
    background: $search-results-background;
    overflow: scroll;
    height: 100%;
    width: 100%;
    box-shadow: 0px 0px 37px 0px rgba(0, 0, 0, 1);
}

.input-group {
    margin-top: 1em;
}

/* Medium devices (tablets, 768px and up) The navbar toggle appears at this breakpoint */
@media (min-width: 768px) {
    .input-group {
        margin-top: 0;
    }

    .search-results-container {
        top: 3em;
    }
}

.search-results-container h1 {
    font-size: 1.7em;
}

.search-results-container h2 {
    font-size: 1.2em;
}

.hideSearchResults {
    display: none;
}

.buttonHideSearchResults {
    font-size: 2em;
    position: fixed;
    right: -0.3em;
    top: 4.2em;
    cursor: pointer;
    background: transparent;
    color: $button-hide-search-results-color;
}

/* Medium devices (tablets, 768px and up) The navbar toggle appears at this breakpoint */
@media (min-width: 768px) {
    .buttonHideSearchResults {
        top: 1em;
        right: 0;
    }
}
</style>
