<!--restaurants-->
<template>
	<div v-if="dataloaded">
		<div class="page_header" v-if="pageBanner" v-lazy:background-image="storeBanner.image_url">
			<div class="site_container">
				<div class="header_content">
					<h1>{{$t("stores_page.restaurants")}}</h1>
				</div>
			</div>
		</div>
		<div class="site_container page_content">
			<div class="row bold">
				<div class="col-sm-6 col-md-4">
					<div class="store_search" >
						<search-component :list="restaurants" :placeholder="$t('stores_page.find_your_store')" suggestion-attribute="name" v-model="search_result" @select="onOptionSelect" class="text-left">
							<template slot="item" scope="option" class="manual">
								<article class="media">
									<p>
										<strong>{{ option.data.name }}</strong>
									</p>
								</article>
							</template>
						</search-component>
						<img src="//codecloud.cdn.speedyrails.net/sites/5a6a54eb6e6f647da51e0100/image/png/1517497861636/search_icon_2x.png" class="pull-right" id="store_search_img" alt="">
					</div>
				</div>
				<div class="col-sm-6 col-md-4">
					<div class="store_search" >
						<a class="directory_link" href="/stores">
							<div class="promotions_header_container directory_btn">{{$t("stores_page.view_stores")}}</div>
						</a>
					</div>
				</div>
				<div class="col-md-4 col-sm-12 hidden_phone">
					<div class="store_search" >
						<a class="directory_link" href="/map">
							<div class="promotions_header_container directory_btn">{{$t("stores_page.view_map")}}</div>
						</a>
					</div>
				</div>
			</div>
			<div class="row">
				<div id="store_list_container">
					<store-masonry :filteredStores="filteredStores"></store-masonry>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
    define(["Vue", "vuex", "vue-select", "vue!search-component", "vue-lazy-load", "vue!store-masonry"], function(Vue, Vuex, VueSelect,  SearchComponent,VueLazyload, StoreMasonry) {
        Vue.use(VueLazyload);
        return Vue.component("restaurants", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    filteredStores: null,
                    dataloaded: false,
                    pageBanner : null,
                    search_result : null,
                }
            },
            created (){
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Restaurants Banner');
                    if(temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    }
                    else {
                        this.pageBanner = {};
                        this.pageBanner.image_url = "";
                    }
                    this.filteredStores = this.restaurants;
                    this.dataloaded = true;
                });
            },
            methods: {
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch("getData", "categories"), this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                onOptionSelect(option) {
                    this.search_result = "";
                    this.$router.push("/stores/" + option.slug);
                },
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedStores',
                    'findRepoByName',
                    'storesByCategoryName',
                ]),
                restaurants() {
                    var category_ids = [5724, 5721, 5723, 5722, 5706];
                    var filtered_restaurants =_.filter( this.processedStores, function(o) {
                        return _.intersection(category_ids, o.categories).length > 0;
                    });
                    
                    filtered_restaurants.map(store => {
                       if (_.includes(store.store_front_url_abs, 'missing')) {
                            store.store_front_url_abs = this.property.default_logo_url;
                        }
                    });
                    return filtered_restaurants;
                }
            }
        });
    });
</script>