<template>
	<div v-if="dataloaded">
	    <div class="page_header" v-if="pageBanner" :style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
			<!--http://via.placeholder.com/1920x300-->
			<div class="site_container">
				<div class="header_content">
					<h1>{{$t("stores_page.map")}}</h1>
				</div>
			</div>
		</div>
		<div class="site_container">
			<div class="col-sm-3   padding_top_20">
			    <div class="store_search map_store_search hidden_phone" >
					<search-component :list="allStores" :placeholder="$t('stores_page.search_stores')" suggestion-attribute="name" v-model="search_result" @select="onOptionSelect" class="text-left">
						<template slot="item" scope="option" class="manual">
							<article class="media">
								<p>
									<strong>{{ option.data.name }}</strong>
								</p>
							</article>
						</template>
					</search-component>
					<i class="fa fa-search pull-right search_icon"></i>
					<!--<img src="//codecloud.cdn.speedyrails.net/sites/5a6a54eb6e6f647da51e0100/image/png/1517497861636/search_icon_2x.png" class="pull-right" id="store_search_img" alt="">-->
				</div>
                <div class="map_directory text-center hidden_phone">
                    <!--<h3 class="map_title caps">{{$t("stores_page.find_store")}}</h3>-->
                    <div id="stores_container" class="directory_list text-left">
                        <li v-for="store in allStores" class="pointer">
							<p class="directory_store_name caps" v-on:click="dropPin(store)">{{store.name}}</p>
						</li>
                    </div>
                </div>
                <div class="visible_phone">
                    <!--<p class="text_left">Find Store :</p>-->
					<div class="alphabet-dd visible_phone" >
					    <v-select :options="allStores" label="name" :searchable="false" :on-change="dropPin" id="mobile_alpha_list" :placeholder="$t('stores_page.select_a_store')"></v-select>
				    </div>
                </div>
            </div>
            <div class="col-sm-9">
            <div class="padding_top_20 map light_border">
				<!--<png-map ref="pngmap_ref" :png-map-url="getPNGurl" :initial-position="'700 450'" @updateMap="updatePNGMap"></png-map>-->
				<mapplic-map ref="mapplic_ref" :height="650" :minimap= "false" :deeplinking="false" :sidebar="false" :hovertip="true" :maxscale= "5" :storelist="mapStores" :floorlist="floorList" tooltiplabel="View Store Details"></mapplic-map>
			</div>
            </div>
		</div>
	</div>
</template>
<style>
	#png_map{
	    width:1310px;
		height: 983px;
		min-width:1310px;
		min-height: 983px;
	}
</style>
<script>
    define(["Vue", "vuex", "vue-select", "vue!search-component", "vue!mapplic-map"], function(Vue, Vuex, VueSelect, SearchComponent, MapplicComponent) {
        return Vue.component("stores-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    listMode: "alphabetical",
                    selectedCat: null,
                    selectedAlpha: "All",
                    alphabet: ["All", "A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z"],
                    filteredStores: null,
                    dataloaded: false,
                    mobile_store: false,
                    windowWidth: 0,
                    storeBanner : null,
                    search_result : null,
                }
            },
            created (){
                this.loadData().then(response => {
                    this.dataloaded = true;
                    this.filteredStores = this.allStores;

                    var temp_repo = this.findRepoByName('Directory Banner');
                    if(temp_repo) {
                        this.pageBanner = temp_repo.images[0];
                    } else {
                        this.pageBanner = {};
                        this.pageBanner.image_url = "";
                    }
                });
            },
            watch: {
                windowWidth: function() {
                    if (this.windowWidth <= 768) {
                        this.mobile_store = true;
                    } else {
                        this.mobile_store = false;
                    }
                },
            },
            mounted() {
                this.$nextTick(function() {
                    window.addEventListener('resize', this.getWindowWidth);
                    //Init
                    this.getWindowWidth();
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedStores',
                    'findRepoByName'
                ]),
                allStores() {
                    var all_stores = this.processedStores;
                    _.forEach(all_stores, function(value, key) {
                        value.zoom = 2;
                    });
                    return all_stores;
                },
                mapStores() {
                    var all_stores = this.processedStores;
                    _.forEach(all_stores, function(value, key) {
                        value.zoom = 2;
                        if(value.svgmap_region == null){
                            value.svgmap_region = value.id;
                        }
                    });
                    return all_stores;
                },
                storeNames () {
                    return _.map(this.processedStores, 'name');
                },
                getSVGMap(){
                  return "//mallmaverick.com"+this.property.svgmap_url;  
                },
                floorList () {
                    var floor_list = [];
                    var floor_1 = {};
                    floor_1.id = "first-floor";
                    floor_1.title = "Level One";
                    floor_1.map = this.getSVGMap;
                    floor_1.z_index = null;
                    floor_1.show = true;
                    floor_list.push(floor_1);
                    
                    return floor_list;
                }
            },
            methods: {
                loadData: async function() {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "categories"), this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                dropPin(store) {
                    this.$refs.mapplic_ref.showLocation(store.svgmap_region);
                },
                getWindowWidth(event) {
                    this.windowWidth = window.innerWidth;
                },
                onOptionSelect(option) {
                    this.search_result = "";
                    this.dropPin(option);
                },
            },
            beforeDestroy: function() {
                window.removeEventListener('resize', this.getWindowWidth);
            },
        });
    });
</script>