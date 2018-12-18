<template>
    <div v-if="currentPage">
        <div v-if="pageBanner" class="page_header" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
			<!--http://via.placeholder.com/1920x300-->
			<div class="site_container">
				<div class="header_content">
					<h1 v-if="locale=='en-ca'">{{currentPage.title}}</h1>
					<h1 v-else>{{currentPage.title_2}}</h1>
					<h2 style="display:none;">Scroll to  view page details</h2>
					<h3 style="display:none;">View page details</h3>
				</div>
			</div>
		</div>
		<div class="site_container inside_page_content page_content">
            <div class="margin_side_20" >
                <div class="row event_container"  v-if="accessibilityData"  v-for="promo in accessibilityData">
					<div class="col-sm-12 col-md-12 event_dets_container">
						<h4 class="event_name caps">{{promo.notice_title}}</h4>
						<div class="event_thick_line"></div>
						<p class="event_dates">{{promo.service_completed_date | moment("MMMM D, YYYY", timezone)}}</p>
						<p class="event_desc" v-html="promo.notice_text_approved"></p>
						
					</div>
					<div class="col-sm-12">
						<hr>
					</div>
				</div>
                <div v-if="accessibilityData" style="padding-top:20px;"></div> 
                <div class="page_body description_text text_left" v-if="locale=='en-ca'" v-html="currentPage.body"></div>
                <div class="page_body description_text text_left" v-else v-html="currentPage.body_2"></div>
            </div>
        </div>
        
    </div>
    <!--Pages Banner-->
</template>
<style>
    .page_title {
        /*border-top:1px solid #aea99e;*/
        border-bottom:1px solid #aea99e;
        height: 35px;
        line-height: 35px;
    }
    #pages_container img{
        width: 100%;
        height: auto;
    }
    .acc_title {
        margin-top:20px;
    }
    img {
        max-width: 100%;
    }
</style>
<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment"], function(Vue, Vuex, moment, tz, VueMoment) {
        return Vue.component("page-details-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    success_subscribe: false,
                    currentPage: null,
                    pageBanner : null,
                    accessibilityData : null
                }
            },
            props:['id', 'locale'],
            beforeRouteUpdate(to, from, next) {
                next();
                this.updatePageData(to.params.id);
            },
            created(){
               this.updatePageData(this.id);
               if(this.id == "bramaleacitycentre-accessibilty" ){
                   this.updateAccessibilityData();
               }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findRepoByName'
                ])
            },
            methods: {
                loadData: async function(id) {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch('LOAD_PAGE_DATA', {url: this.property.mm_host + "/pages/" + id + ".json"}),this.$store.dispatch("getData", "repos")]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                updatePageData (id) {
                    this.loadData(id).then(response => {
                        if(response == null || response == undefined) {
                            this.$router.replace('/');
                        }
                        this.currentPage = response[0].data;
                        var temp_repo = null;
                        //Add custom banners for indivial pages 
                        temp_repo = this.findRepoByName('Pages Banner');
                        
                        if(temp_repo && temp_repo.images) {
                            this.pageBanner = temp_repo.images[0];
                        } else {
                            this.pageBanner = {};
                            this.pageBanner.image_url = ""
                        }
                    });
                },
                updateAccessibilityData() {
                    var vm = this;
                    url = "//acc.speeker.co/get_display_templates?site_id=1";
                    $.getJSON(url).done(function(data){
                        var acc_data = data;
                        acc_data.map(item => {
                            item.notice_text_approved = _.replace(item.notice_text_approved, '<br/><br/>', '<br/>');
                        });
                        
                        vm.accessibilityData =  _.sortBy(acc_data, [function(o) { return o.service_completed_date; }]).reverse();
                        console.log("accessibilityData",vm.accessibilityData)
                    });
                }
            }
        });
    });
</script>