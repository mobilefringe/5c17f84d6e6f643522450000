<template>
	<div class="page_container" id="contact_us_container">
		<!-- for some reason if you do not put an outer container div this component template will not render -->
		<div  v-if="pageBanner" class="page_header" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
			<div class="site_container">
				<div class="header_content">
					<h1>{{$t("newsletter_page.newsletter")}}</h1>
				</div>
			</div>
		</div>
		<div class="site_container">
			<div class="row">
				<div class="col-md-12 contact_contents">
					<div id="send_contact_success" class="alert alert-success" role="alert" v-show="formSuccess">
						<span class="glyphicon glyphicon-ok" aria-hidden="true"></span>
						<span class="sr-only">{{$t("newsletter_page.success")}} : </span>
						{{$t("newsletter_page.thank_you_message")}}
					</div>
					<div id="send_contact_error" class="alert alert-danger" role="alert" v-show="formError">
						<span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
						<span class="sr-only">{{$t("newsletter_page.error")}} : </span>
						{{$t("newsletter_page.error_message")}}
					</div>
					<form class="form-horizontal js-cm-form" id="subForm" action="https://www.createsend.com/t/subscribeerror?description=" method="post" data-id="92D4C54F0FEC16E5ADC2B1904DE9ED1A6C853307DE1968F369EC40BE733D4D5F8458A5E09D247F6F2E7016EB4DAFD16534D29AAD20BB67ACEC9E0FEFEEEEA5A2" @submit.prevent="validateBeforeSubmit">
						<div class="form-group ">
							<div class="col-sm-6 col-xs-12" >
								<label class="label" for="fieldName">{{$t("newsletter_page.name")}}</label>
								<input v-model="form_data.name" required class="form-control" name="cm-name" type="text" placeholder="Name" id="fieldName">
							</div>
						</div>
						<div class="form-group">
							<div class="col-sm-6 col-xs-12">
								<label class="label" for="fieldEmail">{{$t("newsletter_page.email")}}</label>
								<input v-model="form_data.email" required class="form-control js-cm-email-input" name="cm-fuljdi-fuljdi" type="email" placeholder="Email" id="fieldEmail">
							</div>
						</div>
						<div class="form-group">
							<div class="col-xs-12">
								<label class="checkbox">
								<input name="agree_newsletter" required  type="checkbox">
								{{$t("newsletter_page.agree")}} {{property.name}}. 
								</label>
							</div>
						</div>
						<div class="form-group">
							<div class="col-xs-12">
								<button class="js-cm-submit-button contest_btn" type="submit" :disabled="formSuccess">{{$t("newsletter_page.subscribe")}}</button>
							</div>
						</div>
					</form>
				</div>
			</div>
			<div class="padding_top_40"></div>
		</div>
	</div>
</template>

<style>
    .form-group label {
        display: inline-block;
    }
    .checkbox {
        font-weight: normal;
            margin-left: 20px;
    }
</style>
<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta", 'vee-validate', 'jquery', 'utility', 'campaignMonitor'], function(Vue, Vuex, moment, tz, VueMoment, Meta, VeeValidate, $, Utility, campaignMonitor) {
        Vue.use(Meta);
        Vue.use(VeeValidate);
        return Vue.component("newsletter-component", {
            template: template, // the variable template will be injected
            props: ['id'],
            data: function() {
                return {
                    currentPage: null,
                    form_data : {},
                    formSuccess : false,
                    formError: false,
                    pageBanner: null
                }
            },
            created () {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Newsletter Banner');
                    if(temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    }
                    else {
                        this.pageBanner = {};
                        this.pageBanner.image_url = "";
                    }
                });    
            },
            mounted () {
                this.form_data.email = this.$route.query.email;
                $("#newsletter_email").val(this.form_data.email);
                if(this.$route.query.success == 'success') {
                    
                    this.formSuccess = true;
                    this.$router.replace('/newsletter');
                }
            },
            watch : {
                $route () {
                    this.form_data.email = this.$route.query.email;
                    $("#newsletter_email").val(this.form_data.email);
                    if(this.$route.query.success == 'success') {
                        
                        this.formSuccess = true;
                    }
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
                validateBeforeSubmit(form) {
                    this.$validator.validateAll().then((result) => {
                        if (result) {
                            let errors = this.errors;
                            
                            if(errors.length > 0) {
                                console.log("Error");
                                this.formError = true;
                            }
                            else {
                                form.preventDefault();
                                console.log("No Error", form);
                                var vm = this;
                                $.getJSON(
                                    form.target.action ,
                                    $(form.target).serialize(),
                                    function (data) {
                                    if (data.Status === 400) {
                                      vm.formError = true;
                                        console.log("ERROR");
                                    } else { // 200
                                        vm.formSuccess = true;
                                        console.log("SUCCESS");
                                    }
                                });
                                form.preventDefault();
                            }
                        }
                    })
                },
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([,this.$store.dispatch("getData", "repos")]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
            }
        });
    });
</script>
