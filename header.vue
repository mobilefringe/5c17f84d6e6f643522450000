<template>
    <div class="sticky">
    	<div class="top_bar">
    		<div class="site_container">
    			<div class="row top_bar_wrapper">
    				<div class="col-sm-4">
    					<div class="mobile_site_logo visible_phone">
    					    <router-link to="/"><img :src="property_logo" :alt="property.name"/></router-link>
    				    </div>
    					<div id="home_hours_container" class="hidden_phone" v-if="todays_hours">
    						<p class="open_now"><span v-if="todays_hours.is_closed == null || !todays_hours.is_closed ">{{$t("header.open_today")}}</span><span v-else>{{$t("header.closed")}}</span> <span v-if="todays_hours.is_closed == null || !todays_hours.is_closed "><span style="margin:0 20px">|</span> {{todays_hours.open_time | moment("h:mma", timezone)}} - {{todays_hours.close_time | moment("h:mma", timezone)}} </span></p>
    					</div>
    				</div>
    				<div class="col-sm-8 hidden_phone text-right">
    					<div class="header_social">
    					<transition name="custom-classes-transition" enter-active-class="animated slideInRight" leave-active-class="animated slideOutRight">
        					<div id="search_component_wrapper" class="inline_block" v-if="showSearch">
        					    <div class="search_component_container" >
                                    <search-component :list="searchList" placeholder="Search" :suggestion-attribute="suggestionAttribute" :keys="keys" v-model="search_result" @select="onOptionSelect" :autocomplete="false" :minMatchCharLength="3" :tokenize="true" class="text-left">
                                        <template slot="item" scope="option" class="manual">
                                            <article class="media">
                                            <p>
                                                {{ option.data.name }}
                                            </p>
                                            </article>
                                        </template>
                                    </search-component>
                                </div>
        					</div>
        				</transition>
    					    <div class="search_icon inline_block solid_bg" @click="showSearch = !showSearch"> <i class="fa fa-search"></i></div>
    					    <social-links class="inline_block solid_bg"></social-links>
    					</div>
    					<router-link id="signup" to="/rewards_club">{{$t("header.sign_up")}}</router-link>
    					<a href="https://play.google.com/store/apps/details?id=ca.wearecircus.bcc" target="_blank">
    					    <p class="accessibility"></p>
    					    <img class="app_badge" src="//codecloud.cdn.speedyrails.net/sites/5c0581a36e6f643f53050000/image/png/1532627380000/googleplay.png" alt="Google Play Store" />
    					</a>
    					<a href="https://itunes.apple.com/ca/app/bramalea-city-centre/id1042094590?mt=8" target="_blank">
    					    <p class="accessibility"></p>
    					    <img class="app_badge" src="//codecloud.cdn.speedyrails.net/sites/5c0581a36e6f643f53050000/image/png/1532627498000/applestore.png" alt="Apple App Store" />
    					</a>
    				</div>
    				<div id="menu-icon" @click="show_mobile_menu = !show_mobile_menu" :class="{ open: show_mobile_menu}">
    					<span></span>
    					<span></span>
    					<span></span>
    					<span></span>
    				</div>
    				<div class="mobile_nav_container visible_phone">
    				    <transition name="custom-classes-transition" enter-active-class="animated slideInRight" leave-active-class="animated slideOutRight">
    						<nav id="mobile_nav" v-show="show_mobile_menu">
    							<ul>
    								<div class="mobile_menu_site_logo">
    									<router-link to="/"><img :src="property_logo" :alt="property.name"/></router-link>
    								</div>
    								<li v-for="(item,key) in menu_items" class="menu_item">
    							        <router-link :to="item.href" v-if="item.sub_menu == undefined">{{$t(item.name)}}</router-link>
    							        <div v-else>
    							            <b-card no-body class="mb-1">
                                                <b-card-header header-tag="header" class="p-1" role="tab">
                                                    <b-btn block @click="toggleSubMenu(item, $event)" :class="item.show_sub_menu ? 'collapsed' : null" :aria-controls="$t(item.name)" :aria-expanded="item.show_sub_menu ? 'true' : 'false'">
                                                        {{$t(item.name)}}
                                                        <i v-if="item.show_sub_menu"  class="fa fa-minus"></i>
                                                        <i v-else  class="fa fa-plus"></i>
                                                    </b-btn>
                                                </b-card-header>
                                                <b-collapse v-model="item.show_sub_menu" :id="item.id" :visible="item.show_sub_menu" :accordion="$t(item.name)" role="tabpanel" class="accordion_body">
                                                    <b-card-body v-for="sub_menu in item.sub_menu">
                                                        <p class="card-text"><router-link :to="sub_menu.href">{{$t(sub_menu.name)}}</router-link></p>
                                                    </b-card-body>
                                                </b-collapse>
                                            </b-card>
    							        </div>
    							        
    							    </li>
    							</ul>
    							<div class="small_hr"></div>
    							<div class="tel_num" v-if="property && property.contact_phone">
                                    <a :href="'tel:'+property.contact_phone">{{property.contact_phone}}</a>
                                </div>
                                <div>
                                   <p style="display:block"> {{property.address1}}</p>
                                    <p style="display:block">{{property.city}}, {{property.postal_code}} {{property.province_state}}</p>
                                </div>
    							<div class="header_social">
    							    <social-links></social-links>
    							</div>
    							<div>
    							    <a href="https://play.google.com/store/apps/details?id=ca.wearecircus.bcc" target="_blank">
    					                <p class="accessibility"></p>
					                    <img class="app_badge" src="//codecloud.cdn.speedyrails.net/sites/5c0581a36e6f643f53050000/image/png/1532627380000/googleplay.png" alt="Google Play Store" />
					                </a>
					                <a href="https://itunes.apple.com/ca/app/bramalea-city-centre/id1042094590?mt=8" target="_blank">
    					                <p class="accessibility"></p>
					                    <img class="app_badge" src="//codecloud.cdn.speedyrails.net/sites/5c0581a36e6f643f53050000/image/png/1532627498000/applestore.png" alt="Apple App Store" />
					                </a>
					            </div>
    							<div class="small_hr"></div>
    						</nav>
    					</transition>
    				</div>
    			</div>
    		</div>
    	</div>
    	<div class="menu_bar hidden_phone">
    		<div class="site_container">
    			<div class="nav_container hidden_phone">
    				<div class="site_logo">
    					<router-link to="/"><img :src="property_logo" :alt="property.name"/></router-link>
    				</div>
    				<div class="top_nav hidden_phone">
    					<nav id="primary_nav">
    						<ul>
    						    <li v-for="(item, index) in menu_items" class="menu_item" :class="{dropdown : item.sub_menu,open : item.open_dropdown}" >
    						        <!-- @keyup.enter="toggleDropdown(index)" -->
    						        <router-link v-if="item.sub_menu" to="#" data-toggle="dropdown" class="dropdown-toggle" aria-haspopup="true" :aria-expanded="item.open_dropdown">{{$t(item.name)}}</router-link>
    						        <router-link v-else :to="item.href">{{$t(item.name)}}</router-link>
    						        <ul v-if="item.sub_menu" class="dropdown-menu" :class="{ show: item.open_dropdown }">
    						            <li v-for="sub_menu in item.sub_menu" class="dropdown_item">
    						                <router-link :to="sub_menu.href">{{$t(sub_menu.name)}}</router-link>
    						            </li>
    								</ul>
    						    </li>
    						</ul>
    					</nav>
    				</div>
    			</div>
    		</div>
    	</div>
    </div>
</template>

<script>
    define(["Vue", "vuex", 'vue!social_links.vue', "bootstrap-vue", 'json!menu_items.json', 'jquery', 'vue!search-component'], function (Vue, Vuex, SocialLinks, BootstrapVue, MenuItems, $, SearchComponent) {
        Vue.use(BootstrapVue);
        return Vue.component("header-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    dataLoaded: false,
                    show_mobile_menu: false,
                    property_logo: "//codecloud.cdn.speedyrails.net/sites/5c17f84d6e6f643522450000/image/png/1545071987721/logo.png",
                    menu_items: MenuItems,
                    showSearch: false,
                    search_result: null,
                    suggestionAttribute: "name",
                    keys: ["name", "description", "tags", "keywords", "store.name"]
                }
            },
            watch: {
                $route: function() {
                    // hide dropdown when route changes
                    _.forEach(this.menu_items, function(value, key) {
                        value.open_dropdown = false;
                    });
                    this.show_mobile_menu = false; //close menu when navigating to new page
                },
                show_mobile_menu: function() {
                    if(this.show_mobile_menu === true){
                        document.body.classList.add("no-scroll");
                    } else if (this.show_mobile_menu === false) {
                        document.body.classList.remove("no-scroll");
                    }
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'getTodayHours',
                    'processedStores',
                    'processedEvents',
                    'processedPromos',
                    'processedJobs'
                ]),
                searchList() {
                    var events = this.processedEvents;
                    _.forEach(events, function (value, key) {
                        if (_.includes(value.eventable_type, 'Property')) {
                            value.is_store = false;
                        } else {
                            value.is_store = true;    
                        }
                    });
                    var promos = this.processedPromos;
                    _.forEach(promos, function (value, key) {
                        if (_.includes(value.promotionable_type, 'Property')) {
                            value.is_store = false;
                        } else {
                            value.is_store = true;    
                        }
                    });
                    var jobs = this.processedJobs;
                    _.forEach(jobs, function (value, key) {
                        if (_.includes(value.jobable_type, 'Property')) {
                            value.is_store = false;
                        } else {
                            value.is_store = true;    
                        }
                    });
                    var stores = this.processedStores;
                    _.forEach(stores, function (value, key) {
                        value.is_store = true;    
                    });
                    
                    var list = _.union( stores, events, promos, jobs );
                    return list;
                },
                todays_hours() {
                    return this.getTodayHours;
                }
            },
            methods: {
                toggleDropdown(index) {
                    this.menu_items[index].open_dropdown = true;
                },
                toggleSubMenu(item, event) {
                    this.$nextTick(function() {
                        
                        item.show_sub_menu = !item.show_sub_menu;
                        if(item.show_sub_menu) {
                            event.target.classList.add("collapsed");
                            document.getElementById(item.id).classList.add("show");
                        }
                        else {
                            event.target.classList.remove("collapsed");
                            document.getElementById(item.id).classList.remove("show");
                        }
                    });
                },
                onOptionSelect(option) {
                    ga('send', 'event', 'Search Keywords', 'search', this.search_result);
                    this.$router.push({
                        name: "search-results",
                        query: { searchQuery: this.search_result },
                        params: { results: option }
                    });
                    this.$nextTick(function() {
                        this.search_result = "";
                    });
                },
            }
            
        });
    });
</script>