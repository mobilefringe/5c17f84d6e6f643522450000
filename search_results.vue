<template>
	<div>
        <div v-if="dataLoaded" v-cloak>
            <div v-if="pageBanner" class="page_header" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
    			<div class="site_container">
    				<div class="header_content">
    					<h1>Seach Results</h1>
    					<h2 style="display:none;">Scroll to  view page details</h2>
    					<h3 style="display:none;">View page details</h3>
    				</div>
    			</div>
    		</div>
            <div class="site_container margin_30">
        		<div class="text-left" v-if="searchResults && searchResults.length > 0" id="searchResults">
                    <p class="search_result_title">Found {{searchResults.length}} results matching "{{searchQuery}}"</p>
        			<div v-for="(result,index) in searchResults" :key="index">
                        <div class="row result_container_row">
                            <div v-if="result.is_store" class="col-sm-3 store_details_image center-block">
                                <div v-if="(result.store && _.includes(result.store.store_front_url_abs,'missing')) || (!result.store)">
                                    <div class="no_logo">
                                        <img src="//codecloud.cdn.speedyrails.net/sites/5b88438d6e6f641e8d3c0000/image/png/1536092029690/transparent_logo.png">
                                        <p class="store_details_name">
                                            <span v-if="result.store_front_url_abs">{{result.name}}</span>
                                            <span v-else>{{ result.store.name }}</span>
                                        </p>
                                    </div>    
                                </div> 
                                <div v-else>
                                    <img v-if="result.store" class="result_logo" :src="result.store.store_front_url_abs"/>
                                    <img v-else-if="result.store_front_url_abs" class="result_logo" :src="result.store_front_url_abs"/>
                                </div>
                            </div>
                            <div v-else class="col-sm-3 store_details_image center-block">
                                <img class="result_logo" :src="property.default_logo_url"/>    
                            </div>
                            <div class="col-sm-9 search_result_content">
                                <h3>{{result.name}}</h3>
                                <p>{{truncated(result.description)}}</p>
                                <router-link v-if="result.store_front_url_abs" class="result_link hvr-icon-forward" :to="{ name: 'storeDetails', params:{ id:result.slug }}">
                                    <i class="fa fa-caret-right hvr-icon"></i> View Store Details
                                </router-link>
                                <router-link v-else-if="result.promo_image_url_abs" class="result_link hvr-icon-forward" :to="{ name: 'promotionDetails', params: { id: result.slug }}">
                                    <i class="fa fa-caret-right hvr-icon"></i> View Promotion Details 
                                </router-link>
                                <router-link v-else-if="result.event_image_url_abs" class="result_link hvr-icon-forward" :to="{ name: 'eventDetails', params: { id: result.slug }}">
                                    <i class="fa fa-caret-right hvr-icon"></i> View Event Details
                                </router-link>
                                <router-link v-else-if="result.jobable_id" class="result_link hvr-icon-forward" :to="{ name: 'jobDetails', params: { id: result.slug }}">
                                    <i class="fa fa-caret-right hvr-icon"></i> View Job Details
                                </router-link>
                            </div>
                        </div>                
                    </div>
        		</div>
                <div class="page_container text-left" v-else> 
                    <h4>Sorry, there are no results that match your query. Please try again.</h4>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    define(["Vue", "vuex", "jquery"], function(Vue, Vuex, $) {
        return Vue.component("search-results", {
            template: template, // the variable template will be injected
            data() {
                return {
                    searchResults: null,
                    searchQuery: null,
                    dataLoaded: true,
                    
                }
            },
            beforeRouteUpdate(to, from, next) {
                this.$nextTick(function() {
                    this.updateResults();
                });
                next();
            },
            created() {
                this.updateResults();
                if (
                    this.$route.params.results == null &&
                    this.$route.params.results == undefined
                ) {
                    this.$router.push("/");
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findRepoByName'
                ]),
                pageBanner() {
                    var pageBanner = null;
                    var temp_repo = this.findRepoByName('Pages Banner');
                    if(temp_repo && temp_repo.images) {
                        pageBanner = temp_repo.images[0];
                    }
                    else {
                        pageBanner = {};
                        pageBanner.image_url = "";
                    }
                    return pageBanner;
                }
            },
            methods: {
                truncated(string) {
                    return _.truncate(string, {
                        length: 150,
                        separator: " "
                    });
                },
                updateResults() {
                    if (
                        this.$route.query.searchQuery !== null &&
                        this.$route.query.searchQuery !== undefined &&
                        this.$route.query.searchQuery.length > 0
                    ) {
                        if (
                            this.$route.params.results !== null &&
                            this.$route.params.results !== undefined &&
                            Array.isArray(this.$route.params.results)
                        ) {
                            this.searchResults = this.$route.params.results;
                            this.searchQuery = this.$route.query.searchQuery;
                        }
                    } else {
                        this.$router.push("/");
                    }
                }
            }
        });
    });
</script>