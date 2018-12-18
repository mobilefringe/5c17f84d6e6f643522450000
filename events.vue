<template>
	<div v-if="dataloaded">
		<div class="page_header" v-if="promoBanner" v-lazy:background-image="promoBanner.image_url">
			<!--http://via.placeholder.com/1920x300-->
			<div class="site_container">
				<div class="header_content caps">
					<h1>{{$t("events_page.events")}}</h1>
					<h2 style="display:none;">Scroll to  view events</h2>
					<h3 style="display:none;">View all events below</h3>
					
				</div>
			</div>
		</div>
		<div class="site_container page_content" id="events_container">
			<div  v-if="events.length > 0">
				<!--<paginate name="events" v-if="events" :list="events" class="paginate-list margin-60" :per="4">-->
					<!--<div class="row event_container" v-for="(promo,index) in paginated('events')" :class="{ 'last': index === (paginated('events').length - 1) }">-->
		
            		<div class="row event_container" v-if="showMore > index" v-for="(promo,index) in promos">				
            		    <div class="col-sm-6 col-md-4 event_image_container">
							<!--<router-link :to="'/events/'+ promo.slug" class="event_learn_more">-->
							<img :src="promo.store.store_front_url_abs"  class="event_image image" :alt="'Click here to view ' + promo.name"/>
							<!--</router-link>-->
						</div>
						<div class="col-sm-6 col-md-8 event_dets_container">
							<h4 class="event_name caps" v-if="locale=='en-ca'">{{promo.name}}</h4>
							<h4 class="event_name caps" v-else>{{promo.name_2}}</h4>
							<div class="event_thick_line"></div>
							<p class="event_dates">{{promo.start_date | moment("MMM D", timezone)}} - {{promo.end_date | moment("MMM D", timezone)}}</p>
							<p class="event_desc" v-if="locale=='en-ca'">{{promo.description_short}}</p>
							<p class="event_desc" v-else>{{promo.description_short_2}}</p>
						
							<div class="text-right  col-sm-6" v-if="promo" style="padding:0">
								<router-link :to="'/events/'+ promo.slug" class="event_learn_more pull-left hvr-icon-wobble-horizontal" :aria="promo.name">
								    {{$t("events_page.read_more")}} <i class="fa fa-angle-right hvr-icon" aria-hidden="true"></i>
							    </router-link>
								<social-sharing :url="shareURL(promo.slug)" :title="promo.title" :description="promo.body" :quote="_.truncate(promo.description, {'length': 99})" twitter-user="BCCstyle" :media="promo.image_url" inline-template >
									<div class="blog-social-share pull_right">
										<div class="social_share">
											<network network="facebook">
												<i class="fa fa-facebook social_icons" aria-hidden="true"></i>
											</network>
											<network network="twitter">
												<i class="fa fa-twitter social_icons" aria-hidden="true"></i>
											</network>
										</div>
									</div>
								</social-sharing>
							</div>
						</div>
						<div class="col-sm-12">
							<hr>
						</div>
					</div>
				<!--</paginate>-->
			</div>
			<div id="no_events" class="row" v-else>
				<div class="col-md-12">
					<p>{{$t("events_page.no_event_message")}}</p>
				</div>
			</div>
			<!--<div class="row margin-60">-->
			<!--	<div class="col-md-12">-->
			<!--		<paginate-links for="events" :async="true" :limit="5" :show-step-links="true"></paginate-links>-->
					<!--<paginate-links for="currentSelection" :async="true" :simple="{ next: 'Next »', prev: '« Back' }"></paginate-links>-->
			<!--	</div>-->
			<!--</div>-->
			<div class="show_more">
              <div
                class="pointer"
                v-if="promos && showMore <= promos.length"
                @click="loadMoreStores()"
              >{{$t("promos_page.more_promos")}}</div>
            </div>
		</div>
	</div>
</template>

<style>
    .events_container .date_bar{
        /* Today: */
        background: #D3D3D3;
        height: 40px;
        line-height: 40px;
        margin: auto;
        text-align: center;
    }
    .events_container .date_bar .fa{
        cursor: pointer;
    }
    .events_container .current_date{
        color: #636363;
        padding: 0 10px;
    }
    .events_container .all_dates {
        border-bottom: 1px solid #aea99e;
    }
    .events_container .all_dates span {
        font-size: 16px;
        color: #000000;
        letter-spacing: 1.5px;
        height: 30px;
        line-height: 30px;
        padding: 0 5px;
        cursor: pointer;
    }
    .events_container .all_dates [class*="date_"]:focus, [class*="date_"]:hover { 
        background-color: #D3D3D3;
    }
    .events_container .all_dates span.active { 
        background-color: #bababa;
    }
    .events_container .promo_dets {
        border-bottom: 1px solid #aea99e;
    }
    .events_container .row.is-table-row {
        margin: 0;
    }
    .events_container .row.is-table-row [class*="col-"] {
        padding:0;
    }
    .events_container .feature_read_more {
        width : auto;
    }
    .events_container .social_share network {
        display:inline-block;
        width: 24px;
        height: 24px;
        cursor: pointer;
    }
    .events_container .social_share .social_icons{
        width : 24px;
        height : 24px;
        display:inline;
        margin: 0 2px;
    }
    
</style>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta", "vue-lazy-load", "vue-paginate"], function(Vue, Vuex, moment, tz, VueMoment, Meta, VueLazyload, VuePaginate) {
        Vue.use(Meta);
        Vue.use(VueLazyload);
        Vue.use(VuePaginate);
        return Vue.component("events-component", {
            template: template, // the variable template will be injected
            props:['locale'],
            data: function() {
                return {
                    dataloaded: false,
                    promoBanner: null,
                    // paginate: ['events'],
                    incrementBy: 5,
                    showMore: 5,
                    promos: null
                }
            },
            created() {
                this.loadData().then(response => {
                    this.dataloaded = true;
                    
                    var temp_repo = this.findRepoByName('Events Banner');
                    if(temp_repo) {
                        this.promoBanner = temp_repo.images[0];
                    }
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedEvents',
                    'findRepoByName',
                ]),
                events() {
                    var vm = this;
                    var temp_promo = [];
                    _.forEach(this.processedEvents, function(value, key) {
                        today = moment().tz(vm.timezone);
                        webDate = moment(value.show_on_web_date).tz(vm.timezone);
                        if (today >= webDate) {
                            value.description_short = _.truncate(value.description, {
                                'length': 150
                            });
                            value.description_short_2 = _.truncate(value.description_2, {
                                'length': 150
                            });
                            if (value.store != null && value.store != undefined && _.includes(value.store.store_front_url_abs, 'missing')) {
                                value.store.store_front_url_abs = vm.property.default_logo_url;
                            }
                            else if (value.store == null || value.store == undefined) {
                                value.store = {};
                                value.store.store_front_url_abs =  vm.property.default_logo_url;
                            }
                            temp_promo.push(value);
                        }
                    });
                    temp_promo = _.sortBy(temp_promo, ['created_at', 'start_date']).reverse();
                    return temp_promo;
                },
            },
            methods: {
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch("getData", "events"), this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                shareURL(slug){
                    var share_url = "http://bramaleacitycentre.com/events/" + slug;
                    return share_url;
                },
                loadMoreStores() {
                  if (this.showMore <= this.promos.length) {
                    var num = this.showMore + this.incrementBy;
                    this.showMore = num;
                  }
                }
            }
        });
    });
</script>