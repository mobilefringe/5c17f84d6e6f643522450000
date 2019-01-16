<template>
	<div id="find_us_container">
	    <div  v-if="pageBanner" class="page_header" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')', marginBottom:'0px'}">
			<div class="site_container">
				<div class="header_content">
					<h1>Find Us</h1>
				</div>
			</div>
		</div>  
		<!-- for some reason if you do not put an outer container div this component template will not render -->
		<div>
			<div class="row text-left">
					<iframe src="https://www.google.com/maps/embed?pb=!1m14!1m8!1m3!1d1405.0774986332171!2d-113.45766032569233!3d53.52063222563502!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x0%3A0xc85d0c9c289d88ad!2sBonnie+Doon+Shopping+Centre!5e0!3m2!1sen!2sca!4v1517707745783" width="100%" height="450" frameborder="0" style="border:0" allowfullscreen></iframe>
                <div class="text-left site_container padding_tb_30" v-if="currentPage" v-html="currentPage.body"></div>
					
			</div>
			<div class="padding_top_40"></div>
		</div>
	</div>
</template>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta", 'vee-validate'], function(Vue, Vuex, moment, tz, VueMoment, Meta, VeeValidate) {
        Vue.use(Meta);
        Vue.use(VeeValidate);
        return Vue.component("find-us-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    success_subscribe: false,
                    currentPage: null,
                    pageBanner: null
                }
            },
            created(){
                this.loadData().then(response => {
                    this.currentPage = response[0].data;
                    var temp_repo = this.findRepoByName('Contact Us Banner');
                    if(temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    }
                    else {
                        this.pageBanner = {};
                        this.pageBanner.image_url = "";
                    }
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findRepoByName'
                ]),
                full_address() {
                    var address = this.property.address1 +','+this.property.city +','+ this.property.country +','+this.property.province_state +','+this.property.province_state;
                    return address.replace(/ /g,"+");
            
                }
            },
            methods: {
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch('LOAD_PAGE_DATA', {url: this.property.mm_host + "	/pages/bonniedoon-contact-us.json"}),this.$store.dispatch("getData", "repos")]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
            }
        });
    });
</script>