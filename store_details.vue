<template>
	<div class="page_container store_dets_container" v-if="dataLoaded" id="store_dets_container">
		<div class="page_header" v-if="pageBanner" v-lazy:background-image="pageBanner.image_url">
			<!--http://via.placeholder.com/1920x300-->
			<div class="site_container">
				<div class="header_content">
					<h1>{{$t("stores_page.shopping")}}</h1>
				</div>
			</div>
		</div>
		<div class="site_container">
			<div class="row">
				<div class="col-sm-4 promo_logo_container hidden_phone">
					<div class="image_container details_store_image">
						<!--<img v-lazy="currentStore.store_front_url_abs" class="image"/>-->
						<div v-if="currentStore.no_store_logo" class="store_details_image center-block">
                            <div class="no_logo">
                                <p class="store_details_name">{{ currentStore.name }}</p>
                            </div>    
                        </div>
                        <img v-else class="store_details_image center-block" :src="currentStore.store_front_url_abs" :alt="currentStore.name + ' Logo'" />
					</div>
				</div>
				<div class="col-sm-8 promo_image_container text-left">
					<div class="col-sm-12 no_padding">
						<!--<png-map ref="pngmapref" v-bind:png-map-url="getPNGurl" :initial-position="'1250 1250'" :height="_.toNumber('625')" @updateMap="updatePNGMap"></png-map>-->
						<mapplic-map ref="mapplic_ref" :height="350" :minimap= "false" :deeplinking="false" :sidebar="false" :hovertip="true" :maxscale= "5" :storelist="mapStores" :floorlist="floorList" tooltiplabel="View Store Details" @updateMap="updateSVGMap"></mapplic-map>
					</div>
				</div>
			</div>
			<hr class="green_hr">
			<div class="row margin_30">
				<div class="col-sm-4">
					<div class="text-center">
					    <h4 class="event_store_name caps" v-if="locale=='en-ca'">{{currentStore.name}}</h4>
						<h4 class="event_store_name caps" v-else>{{currentStore.name_2}}</h4>
						<h4 v-if="currentStore.phone" class="store_dets_title"> <a :href="'tel:'+currentStore.phone">{{currentStore.phone}}</a></h4>
						<h4 v-if="currentStore.website" class="store_dets_title"> <a :href="'//'+currentStore.website" target="_blank">{{$t("stores_page.store_website")}}</a></h4>
						<h4 v-if="storeHours.length > 0 " class="store_dets_title">{{$t("stores_page.store_hours")}}</h4>
						<ul class="store_hours_list">
							<li v-if="storeHours" v-for="hour in storeHours" class="col-xs-12">
							    <span class="col-xs-5 text-left">{{hour.day_of_week | moment("dddd", timezone)}}</span>
								<span v-if="hour.is_closed" class="col-xs-7 text-left">Closed</span>
								<span v-else class="col-xs-7 text-left">{{hour.open_time | moment("h:mm A", timezone)}} - {{hour.close_time | moment("h:mm A", timezone)}}</span>
							</li>
						</ul>
					</div>
				</div>
				<hr class="green_hr visible_phone">
				<div class="col-sm-8 text-left">
					<h4 v-if="currentStore.rich_description" class="store_dets_title caps"> {{$t("stores_page.about_us")}}</h4>
					<div class="text-left promo_description">
						<p v-if="locale=='en-ca'" v-html="currentStore.rich_description"></p>
						<p v-else v-html="currentStore.rich_description_2"></p>
					</div>
					<div class="store_promo_container" v-if="promotions.length > 0">
						<div class="promo_container_title text-left caps"></div>
						<h4 v-if="currentStore.rich_description" class="store_dets_title caps margin_30">{{$t("promos_page.promotions")}}</h4>
						<div class="row store_promo_dets text-left" v-for="promo in promotions">
							<div class="col-sm-6 no_padding" >
								<div class="promo_div_image">
									<img v-lazy="promo.image_url" class="image" alt=""/>
								</div>
								<div class="store_promo_dets_container padding_tb_20">
								    <p class="promo_div_name" v-if="locale=='en-ca'">{{promo.name}}</p>
								    <p class="promo_div_name" v-else>{{promo.name_2}}</p>
    								<p class="promo_div_date"><i class="fa fa-calendar"></i>{{promo.start_date | moment("MMM D", timezone)}} - {{promo.end_date | moment("MMM D", timezone)}}</p>
    								<div class="text-center">
    								    <span class="store_dets_btn">
        									<router-link :to="'/promotions/'+promo.slug" class="" >{{$t("promos_page.read_more")}}</router-link>
        								</span>
    								</div>
    								
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>
<style>
	#png_map{
	    width:2500px;
		height: 2500px;
		min-width:2500px;
		min-height: 2500px;
	}
</style>
<script>
    define(['Vue', 'vuex', 'moment', "vue!mapplic-map", 'vue-lazy-load'], function(Vue, Vuex, moment, MapplicComponent, VueLazyload) {
        Vue.use(VueLazyload);
        return Vue.component("store-details-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    currentStore: null,
                    promotions : [],
                    jobs:[],
                    dataLoaded: false,
                    pageBanner : null ,
                    storeHours :[]
                }
            },
            props:['id', 'locale'],
            beforeRouteUpdate(to, from, next) {
                this.currentStore = this.findStoreBySlug(to.params.id);
                if (this.currentStore === null || this.currentStore === undefined){
                    this.$router.replace('/');
                }
                next();
            },
            created (){
                this.loadData().then(response => {
                    this.dataLoaded = true;
                    this.updateCurrentStore(this.id);
                    var temp_repo = this.findRepoByName('Stores Banner');
                    if(temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    }
                    else {
                        this.pageBanner = {};
                        this.pageBanner.image_url = "";
                    }
                });
            },
            watch: {
                currentStore: function() {
                    if ( _.includes(this.currentStore.store_front_url_abs, 'missing')) {
                           this.currentStore.no_store_logo = true;
                        } else {
                          this.currentStore.no_store_logo = false;
                        }
                    var vm = this;
                    var temp_promo = [];
                    var temp_job = [];
                    _.forEach(this.currentStore.promotions, function(value, key) {
                        var current_promo = vm.findPromoById(value);
                        current_promo.description_short = _.truncate(current_promo.description, {
                            'length': 70
                        });
                        temp_promo.push(current_promo);
                    });
                    _.forEach(this.currentStore.jobs, function(value, key) {
                        var current_job = vm.findJobById(value);
                        current_job.description_short = _.truncate(current_job.description, {
                            'length': 70
                        });
                        temp_job.push(current_job);

                    })
                    this.promotions = temp_promo;
                    this.jobs = temp_job;
                    
                    var storeHours = [];
                    var vm = this;
                    _.forEach(this.currentStore.store_hours, function (value, key) {
                        var hour = vm.findHourById(value);
                        if(hour.day_of_week === 0){
                            hour.order = 7;
                        }
                        else {
                            hour.order = hour.day_of_week;
                        }
                        storeHours.push(hour);
                    });
                    this.storeHours = _.sortBy(storeHours, [function(o) { return o.order; }]);
                }
            },
            
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedStores',
                    'findStoreBySlug',
                    'findPromoById',
                    'findJobById',
                    'findRepoByName',
                    'findHourById'
                ]),
                // getPNGurl () {
                //     return "https://www.mallmaverick.com" + this.property.map_url;
                // },
                // pngMapRef() {
                //     return this.$refs.pngmapref;
                // },
                
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
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch("getData","promotions"), this.$store.dispatch("getData", "jobs"),this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                updateCurrentStore (id) {
                    this.currentStore = this.findStoreBySlug(id);
                    if (this.currentStore === null || this.currentStore === undefined){
                        this.$router.replace('/');
                    }
                },
                dropPin(store) {
                    this.$refs.mapplic_ref.showLocation(store.svgmap_region);
                },
                updateSVGMap(map) {
                    this.map = map;
                     this.dropPin(this.currentStore);
                },
                checkImageURL(value) {
                  if (_.includes(value.image_url, "missing")) {
                    if (value.store === null || value.store === undefined) {
                      return "//codecloud.cdn.speedyrails.net/sites/5c17f84d6e6f643522450000/image/png/1545071987721/logo.png";
                    } else if (
                      value.store != null &&
                      value.store != undefined &&
                      _.includes(value.store.store_front_url_abs, "missing")
                    ) {
                      return "//codecloud.cdn.speedyrails.net/sites/5c17f84d6e6f643522450000/image/png/1545071987721/logo.png";
                    } else {
                      return value.store.store_front_url_abs;
                    }
                  } else {
                    return value.promo_image_url_abs;
                  }
                }
            }
        });
    });
</script>