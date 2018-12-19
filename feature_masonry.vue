<template>
    <div v-masonry transition-duration="0.3s" item-selector=".grid-item" >
        <div v-masonry-tile class="item feature_item_container" >
            <div v-for="feature in feature_items" :class="'grid-item ' + feature.masonry_class ">
                <router-link v-if="feature.url" :to="feature.url">
                	<div :class="{ 'gallery-image' : feature.no_hover_class}"> 
                	    <img :src="feature.image_url" :alt="'click to learn more about ' +feature.name">
        				<div class="figcaption">
        					<h3 v-if="locale=='en-ca'" class="hvr-underline-from-center"> {{feature.name}} </h3>
        					<h3 v-else> {{feature.name_2}} </h3>
        					<!--<div class=""></div>-->
        				</div>
                	</div>
            	</router-link>
            	<div v-else>
            	    <div :class="{ 'gallery-image' : feature.no_hover_class}"> 
                	    <img :src="feature.image_url" :alt="'click to learn more about ' +feature.name">
        				<div class="figcaption">
        					<h3 v-if="locale=='en-ca'"> {{feature.name}} </h3>
        					<h3 v-else> {{feature.name_2}} </h3>
        				</div>
                	</div>
        		</div>
            </div>
        </div>
    </div>
</template>
<style>
    .lb-data .lb-number, .lb-nav {
        display:none!important;
    }
</style>
<script>
    define(["Vue", "vuex",  'masonry' , 'vue-masonry-plugin'], function(Vue, Vuex, masonry, VueMasonryPlugin) {
        Vue.use(VueMasonryPlugin.default);
        return Vue.component("feature-masonry", {
            template: template, // the variable template will be injected,
            props:['feature_items', 'locale']
        });
    });
</script>