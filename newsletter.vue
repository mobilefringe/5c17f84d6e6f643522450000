<template>
    <div class="page_container" id="contact_us_container"> <!-- for some reason if you do not put an outer container div this component template will not render -->
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
                    <form  class="form-horizontal js-cm-form" action="https://mobilefringe.createsend.com/t/d/s/gmdyi/" method="post">
                    <!--id="subForm" action="form-submit" v-on:submit.prevent="validateBeforeSubmit"-->
                        <div class="form-group ">
                            <div class="col-sm-6 col-xs-12" >
                                <label class="label" for="cm-name">{{$t("newsletter_page.name")}}</label>
                                <input v-model="form_data.name" required class="form-control" name="cm-name" type="text" placeholder="Name" id="cm-name">
                            </div>
                        </div>
                        <div class="form-group">
                            <div class="col-sm-6 col-xs-12">
                                <label class="label" for="newsletter_email">{{$t("newsletter_page.email")}}</label>
                                <input v-model="form_data.email" required class="form-control js-cm-email-input" name="cm-gmdyi-gmdyi" type="email" placeholder="Email" id="newsletter_email">
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
                                <button class="contest_btn" type="submit" :disabled="formSuccess">{{$t("newsletter_page.subscribe")}}</button>
                            </div>
                        </div>
                    </form>
                    
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
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta", 'vee-validate', 'jquery', 'utility'], function(Vue, Vuex, moment, tz, VueMoment, Meta, VeeValidate, $, Utility) {
        Vue.use(Meta);
        Vue.use(VeeValidate);
        return Vue.component("newsletter-component", {
            template: template, // the variable template will be injected
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
            },
            watch : {
                $route () {
                    this.form_data.email = this.$route.query.email;
                    $("#newsletter_email").val(this.form_data.email);
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
                        let errors = this.errors;
                        if(errors && errors.items.length == 0){ 
                            if(errors.length > 0) {
                                console.log("Error", errors);
                                this.formError = true;
                                form.preventDefault();
                                form.target.action = "";
                            }
                            else {
                                this.campaignMonitorCall($('#subForm'), '92D4C54F0FEC16E5ADC2B1904DE9ED1A784F37613E32FEE3CD464815387D96C2B09750119836ACA3D9670C89A45608486B3A59848BC22C59DE7304A8B405ECFC'); 
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
                campaignMonitorCall(form, form_data_id) {
                    // Get e-mail value.
                    var vm = this;
                    email = $('input[type=email]', form).val();
        
                    // Build request data for tokenRequest.
                    request_data = "email=" + encodeURIComponent(email) + "&data=" + form_data_id;
        
                    // Prepare tokenRequest.
                    tokenRequest = new XMLHttpRequest();
                    tokenRequest.open('POST', 'https://createsend.com//t/getsecuresubscribelink', true);
                    tokenRequest.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
                    tokenRequest.send(request_data);
        
                    // Ready state.
                    tokenRequest.onreadystatechange = function() {
                        if (this.readyState === 4) {
                            if (this.status === 200) {
                                // Having token and new submit url we can create new request to subscribe a user.
                                subscribeRequest = new XMLHttpRequest();
                                subscribeRequest.open('POST', this.responseText, true);
                                subscribeRequest.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
                                subscribeRequest.send(form.serialize());
                                // On ready state call response function.
                                subscribeRequest.onreadystatechange = function() {
                                    if (subscribeRequest.readyState === 4) {
                                        if (_.includes(subscribeRequest.response, 'Thank You')) {
                                            vm.formSuccess = true;
                                        } else {
                                            vm.formError = true;
                                        }
                                    }
                                }
                            } else {
                                return false;
                            }
                        }
                    }
                }
            }
        });
    });
</script>
