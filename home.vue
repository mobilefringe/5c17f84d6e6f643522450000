<template>
	<div class="row page_content" v-if="dataLoaded">
		<div class="banner_div">
			<div class="home-banner-container">
				<slick ref="slick" :options="slickOptions">
					<div class="" v-for="banner in banners" v-if="banners">
					    <a :href="banner.url">
						    <div class="home-banner" v-lazy:background-image="banner.image_url"></div>
						</a>
					</div>
				</slick>
			</div>
		</div>
		<div class="site_container">
		    <h2 style="display:none;">Home Page Features</h2>
		    <div>
		      <h3 class="home_page_title caps">{{$t("home_page.explore")}}</h3>
		    </div>
            <feature-masonry class="" :feature_items="feature_items" :locale="locale"></feature-masonry>
		    <!--<feature-masonry class="visible_phone" :feature_items="mobile_feature_items" :locale="locale"></feature-masonry>-->
            <div>
		      <h3 class="home_page_title caps" v-if="instaFeed && instaFeed.length > 0">{{$t("home_page.our_feed")}}</h3>
		    </div>
		    <div class="insta-feed-container">
                <div class="insta-feed-image " v-for="(item, index) in instaFeed">
                    <a :href="item.link" target="_blank">
                        <div class="insta-feed-background" v-bind:style="{ backgroundImage: 'url(' + item.images.standard_resolution.url + ')' }"></div>
                        <div class="insta_content">
                            <div class="insta_caption">
                                <p>{{ _.truncate(item.caption.text,{'length':75}) }}</p>
                                <div>
                                    <span>
                                        <i class="fa fa-heart"></i> {{ item.likes.count }}
                                    </span>
                                    <span>
                                        <i class="fa fa-comment"></i> {{ item.comments.count }}
                                    </span>
                                </div>
                            </div>
                        </div>
                    </a>
                </div>
            </div>
            
		</div>
	</div>
</template>

<script>
    define(["Vue", "vuex", "vue-meta", "vue!today_hours", "vue!search-component", 'vue!vue-slick', 'js-cookie', 'vue-lazy-load', "vue!feature_masonry"], function(Vue, Vuex, Meta, TodayHoursComponent, SearchComponent, slick, Cookies, VueLazyload, featureMasonry) {
        Vue.use(VueLazyload);
        return Vue.component("home-component", {
            template: template, // the variable template will be injected
            props:['locale'],
            data: function() {
                return {
                    suggestionAttribute: 'name',
                    search: '',
                    slickOptions: {
                        arrows: false,
                        autoplay: true,
                        autoplaySpeed: 5000,
                        cssEase: 'linear',
                        dots: true,
                        fade: true,
                        infinite: true,
                        slidesToShow: 1,
                        speed: 1600
                    },
                    dataLoaded: false,
                    show_popup: false,
                    popup: null,
                    formData : {},
                    instaFeed: null,
                    meta: {
                        meta_title: "",
                        meta_description: "",
                        meta_keywords: "",
                        meta_image: ""
                    }
                }
            },
            created () {
                this.loadData().then(response => {
                    this.dataLoaded = true;
                    this.popup = this.$store.state.popups[0];
                    
                    var socialFeed = response[4].data;
                    var social_feed = socialFeed.social.instagram;
                    this.instaFeed = _.slice(social_feed, [0], [4]);
                    
                    this.meta = this.findMetaDataByPath(this.$route.path);
                });
            },
            watch : {
                dataLoaded () {
                    var viewed = Cookies.get('popup_viewed');
                    if (this.popup !== null && this.popup !== undefined && viewed !== "true") {
                        Cookies.set('popup_viewed', "true");
                        viewed = Cookies.get('popup_viewed');
                        this.show_popup = true;
                        this.popup.image_url = "//assets.mallmaverick.com" + this.popup.photo_url;
                        document.getElementById('popup_backdrop').style.display = "block";
                    } else {
                        document.getElementById('popup_backdrop').style.display = "none";
                    }
                },
                formData () {
                    this.formData.name = this.formData.firstname + " " + this.formData.lastname; 
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedStores',
                    'findRepoBySlug',
                    'findRepoByName',
                    'findMetaDataByPath'
                ]),
                banners () { 
                    var banners = [];//this.$store.state.banners;
                    _.forEach(this.$store.state.banners, function(value, key) {
                        var today = new Date();
                        var start = new Date (value.start_date);
                        if (start <= today){
                            if (value.end_date){
                                var end = new Date (value.end_date);
                                if (end >= today){
                                    banners.push(value);  
                                }
                            } else {
                                banners.push(value);
                            }
                        }
                    })
                    return _.orderBy(banners, ['position'], ['asc']);
                },
                feature_items () {
                    var features = this.$store.state.feature_items;
                    _.forEach(features, function(value, key) {
                        if (_.includes([4,5], key)) {
                            value.masonry_class = "grid-item--height2 grid-item--width2";
                        } else {
                            value.masonry_class = " ";
                        }
                        
                        ifb(value.name === null || value.name === undefined || value.name.length == 0) {
                            value.no_hover_class = false;
                        } else {
                            value.no_hover_class = true;
                        }
                        
                        if (value.url ==  null ||value.url == undefined || value.url.length == 0 ) {
                            value.no_link = true;
                        }
                        
                        if (_.includes(value.url, '//')) {
                            value.do_anchor_tag = true;
                        } else {
                            value.do_anchor_tag = false;
                        }
                    });
                    return features;
                }
            },
            methods: {
                loadData: async function() {
                    try {
                        let results = await Promise.all([
                            this.$store.dispatch("getData", "banners"), 
                            this.$store.dispatch("getData", "feature_items"), 
                            this.$store.dispatch("getData", "promotions"), 
                            this.$store.dispatch("getData", "popups"), 
                            this.$store.dispatch('LOAD_PAGE_DATA', { url: this.property.mm_host +"/api/v4/eastyork/social.json" })
                        ]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                closePopup() {
                    this.show_popup = false;
                    document.getElementById('popup_backdrop').style.display = "none";
                }
            },
            metaInfo () {
                return {
                    title: this.meta.meta_title,
                    meta: [
                        { name: 'description', vmid: 'description', content: this.meta.meta_description },
                        { name: 'keywords',  vmid: 'keywords', content: this.meta.meta_keywords },
                        { property: 'og:title', vmid: 'og:title', content: this.meta.meta_title },
                        { property: 'og:description', vmid: 'og:description', content: this.meta.meta_description },
                        { property: 'og:image', vmid: 'og:image', content: this.meta.meta_image }
                    ] 
                }
            }
        })
    })
</script>