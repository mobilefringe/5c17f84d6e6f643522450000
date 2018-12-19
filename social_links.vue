<template>
    <div>
        <a href="https://www.facebook.com/EastYorkTownCentre" target="_blank"><i class="fa fa-facebook" aria-hidden="true"></i><p style="display:none">Facebook</p></a>
        <a href="https://www.twitter.com/EYTCshops" target="_blank"><i class="fa fa-twitter" aria-hidden="true"></i><p style="display:none">Twitter</p></a>
        <a href="https://www.instagram.com/" target="_blank"><i class="fa fa-instagram" aria-hidden="true"></i><p style="display:none">Instagram</p></a>
         
    </div>
    
</template>
<style>
    .lb-data .lb-number, .lb-nav {
        display:none!important;
    }
</style>
<script>
    define(["Vue", "vuex", "lightbox"], function(Vue, Vuex, Lightbox) {
        Vue.use(Lightbox);
        return Vue.component("social-links", {
            template: template, // the variable template will be injected,
        });
    });
</script>