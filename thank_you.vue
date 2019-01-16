<template>
    <div class="page_container" id="thank_you_container">
        <div v-if="pageBanner" class="page_header" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
			<!--http://via.placeholder.com/1920x300-->
			<div class="site_container">
				<div class="header_content">
					<h1>Thank You</h1>
				</div>
			</div>
		</div>
        <div class="row site_container">
            Your subscription has been confirmed. You've been added to our list and will hear from us soon.
        </div>
    </div>
</template>
<style>
    #thank_you_container div{
        padding: 20px;
    }
</style>
<script>
    define(["Vue"], function(Vue) {
        return Vue.component("thank-you-component", {
            template: template, // the variable template will be injected
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findRepoByName'
                ]),
                pageBanner(){
                
            }
            },
            
        });
    });
</script>