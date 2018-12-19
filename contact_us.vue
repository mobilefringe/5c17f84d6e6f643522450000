<template>
    <div id="contact_us_container"> <!-- for some reason if you do not put an outer container div this component template will not render -->
        <div  v-if="pageBanner" class="page_header" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
			<div class="site_container">
				<div class="header_content">
					<h1>Contact Us</h1>
					<h2 style="display:none;">Scroll to  view contact details</h2>
					<h3 style="display:none;">Scroll to  view contact details</h3>
				</div>
			</div>
		</div>  
        <div class="margin_25_across padding_top_40 site_container">
            <div class="row"> 
                <div class="col-sm-6 text-left" v-if="currentPage">
                    <div class="text-left contact_us_body" v-html="currentPage.body"></div>
                </div> 
                <div class="col-sm-6 contact_contents">
                    <div class="description_text text_left caps">
                        CONTACT US FOR QUESTIONS, COMMENTS AND MORE INFORMATION.
                    </div>
                    <form class="form-horizontal padding_top_20" action="form-submit" @submit.prevent="validateBeforeSubmit">
                        <div class="form-group ">
                            <div class="col-sm-12" :class="{'has-error': errors.has('name')}">
                                <label class="label" for="contact_name">Name</label>
                                <input v-model="form_data.name" v-validate="'required|alpha_spaces'" class="form-control" :class="{'input': true}" name="name" type="text" placeholder="Name" data-vv-delay="500" id="contact_name">
                                <span v-show="errors.has('name')" class="form-control-feedback">{{ errors.first('name') }}</span>
                            </div>
                            <div class="col-sm-12" :class="{'has-error': errors.has('email')}">
                                <label class="label" for="contact_email">Email</label>
                                <input v-model="form_data.email" v-validate="'required|email'" class="form-control" :class="{'input': true}" name="email" type="email" placeholder="Email" data-vv-delay="500" id="contact_email">
                                <span v-show="errors.has('email')" class="form-control-feedback">{{ errors.first('email') }}</span>
                            </div>
                            <div class="col-xs-12" :class="{'has-error': errors.has('message')}">
                                <label class="label" for="contact_message">Message</label>
                                <textarea v-model="form_data.message" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="message" type="text" placeholder="Message" data-vv-delay="500" id="contact_message"></textarea>
                                <span v-show="errors.has('message')" class="form-control-feedback">{{ errors.first('message') }}</span>
                            </div>
                        </div>
                        <div class="form-group account-btn text-left m-t-10">
                            <div class="col-xs-12">
                                <button class="contact_btn" type="submit" :disabled="formSuccess">Submit</button>
                            </div>
                        </div>
                    </form>
                    
                    <div id="send_contact_success" class="alert alert-success text-left" role="alert" v-show="formSuccess">
                        <span class="glyphicon glyphicon-ok" aria-hidden="true"></span>
                        <span class="sr-only">Success</span>
                        Thank you for contacting us. A member from our team will contact you shortly.
                    </div>
                    <div id="send_contact_error" class="alert alert-danger text-left" role="alert" v-show="formError">
                        <span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
                        <span class="sr-only">Error:</span>
                        There was an error when trying to submit your request. Please try again later.
                    </div>
                    
                </div>
                
            </div>
        	
            <div style="padding-top: 40px;"></div>    
        </div>
        <div style="margin-bottom:-30px">
            <iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d1234.5887233230621!2d-79.3461581825161!3d43.70531345482354!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x89d4ccfb4f9d01f3%3A0xf925df5c38dba5e5!2sEast+York+Town+Centre!5e0!3m2!1sen!2sca!4v1545169043397" width="100%" height="300" frameborder="0" style="border:0" allowfullscreen></iframe>
        </div>
    </div>
</template>

<!--<style>-->
<!--    #contact_us_container .row{-->
<!--        margin-left:inherit;-->
<!--    }-->
<!--    #contact_us_container {-->
<!--        margin: 20px auto 0;-->
<!--    }-->
<!--    #contact_us_container .page_header {-->
<!--        margin-bottom:20px;-->
<!--    }-->
<!--    #contact_us_container .property_address {-->
<!--        margin-bottom:20px;-->
<!--    }-->
<!--    .form-group .form-control-feedback{-->
<!--        font-size: 12px;-->
        /*top:60px;*/
<!--        color: #F44336;-->
<!--        top: initial;-->
<!--        bottom: -27px;-->
<!--    }-->
<!--</style>-->

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta", 'vee-validate', 'utility'], function(Vue, Vuex, moment, tz, VueMoment, Meta, VeeValidate,Utility) {
        Vue.use(Meta);
        Vue.use(VeeValidate);
        return Vue.component("contact-us-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    form_data : {},
                    formSuccess : false,
                    formError: false,
                    validaNum: '',
                    correctValNum: null,
                    validNumError: false,
                    currentPage : null,
                    pageBanner: null
                }
            },
            created(){
                this.loadData().then(response => {
                    this.currentPage = response[0].data;
                    var temp_repo = this.findRepoByName('Contact Us Banner');
                    if(temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    } else {
                        this.pageBanner= {};
                        this.pageBanner.image_url = "";
                    }
                });
            },
            mounted () {
                //creating random validation num 
                this.correctValNum = Utility.rannumber();//this.rannumber;
                //ensuring the variables are created in this order for email
                this.form_data.name = null;
                this.form_data.email = null;
                this.form_data.subject = this.property.name + ' Contact Us Form';
                this.form_data.message = null;
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findRepoByName'
                ]),
            },
            methods: {
                validateBeforeSubmit() {
                    this.$validator.validateAll().then((result) => {
                        let errors = this.errors;
                        send_data = {};
                        send_data.form_data = JSON.stringify(Utility.serializeObject(this.form_data));
                        this.$store.dispatch("CONTACT_US", send_data).then(res => {
                            this.formSuccess = true;
                        }).catch(error => {
                            try {
                                if (error.response.status == 401) {
                                    console.log("Data load error: " + error.message);
                                    this.formError = true;
                                } 
                                else {
                                    console.log("Data load error: " + error.message);
                                    this.formError = true;
                                }
                            } 
                            catch (e) {
                                console.log("Data load error: " + error.message);
                                this.formError = true;
                            }
                        })
                        
                    })
                },
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch('LOAD_PAGE_DATA', {url: this.property.mm_host + "	/pages/eastyork-contact-us.json"}),this.$store.dispatch("getData", "repos")]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
            }
        });
    });
</script>
