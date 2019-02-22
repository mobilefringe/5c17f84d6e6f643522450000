<template>
    <div v-if="dataloaded">
        <div class="margin_60"></div>
		<div class="site_container inside_page_content page_content">
		    <div class="margin_60 main_post_container clearfix" v-if="firstPost" v-for="post in firstPost">
                <div class="main_post_image">
                    <img :src="post.image_url" :alt="'Blog Post: ' + post.title">
                </div>
                <div class="main_post_content">
                    <h2 class="main_post_heading caps">{{ post.title }}</h2>
                    <p class="post_dates">{{ post.publish_date | moment("MMM DD, YYYY", timezone) }}</p>
                    <div class="post_text" v-html="post.body_short"></div>
                    <router-link :to="{ name: 'postDetails', params: { id: post.slug }}" class="post_read_more"  :aria="post.title">
					   {{ $t("blog_page.read_post") }} <i class="fa fa-angle-right" aria-hidden="true"></i>
				    </router-link>
				    <social-sharing :url="shareURL(post.slug)" :title="post.title" :description="post.body_short" :quote="_.truncate(post.body, {'length': 99})" :twitter-user="$root.twitter_user" :media="post.image_url" inline-template >
						<div class="post_list_share">
							<div class="social_share">
								<network network="facebook">
									<i class="fa fa-facebook social_icons" aria-hidden="true"></i>
								</network>
								<network network="twitter">
									<i class="fa fa-twitter social_icons" aria-hidden="true"></i>
								</network>
							</div>
						</div>
					</social-sharing>
                </div>
            </div>
            <div class="post_container" v-if="posts" v-for="post in posts">
                <div class="post_image">
                    <img :src="post.image_url" :alt="'Blog Post: ' + post.title">
                </div>
                <div class="post_content">
                    <h2 class="post_heading caps">{{ post.title }}</h2>
                    <p class="post_dates">{{ post.publish_date | moment("MMM DD, YYYY", timezone) }}</p>
                    <div class="post_text" v-html="post.body_short"></div>
                    <router-link :to="{ name: 'postDetails', params: { id: post.slug }}" class="post_read_more"  :aria="post.title">
					   {{ $t("blog_page.read_post") }} <i class="fa fa-angle-right" aria-hidden="true"></i>
				    </router-link>
			        <social-sharing :url="shareURL(post.slug)" :title="post.title" :description="post.body" :quote="_.truncate(post.description, {'length': 99})" :twitter-user="$root.twitter_user" :media="post.image_url" inline-template >
						<div class="post_list_share">
							<div class="social_share">
								<network network="facebook">
									<i class="fa fa-facebook social_icons" aria-hidden="true"></i>
								</network>
								<network network="twitter">
									<i class="fa fa-twitter social_icons" aria-hidden="true"></i>
								</network>
							</div>
						</div>
					</social-sharing>
                </div>
            </div>
            <button class="contact_btn" v-if="!noMorePosts" @click="handleButton">Load More</button>
            <p v-if="noPosts">No More Posts</p>
        </div>
    </div>
</template>
<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-lazy-load", "vue-social-sharing"], function (Vue, Vuex, moment, tz, VueMoment, VueLazyload, SocialSharing) {
        Vue.use(VueLazyload);
        return Vue.component("news-component", {
            template: template, // the variable template will be injected
            data: function () {
                return {
                    dataloaded: false,
                    posts: [],
                    morePosts: [],
                    morePostsFetched: false,
                    noMorePosts: false,
                    noPosts: false
                }
            },
            created() {
                this.loadData().then(response => {
                    this.firstPost
                    this.posts
                    this.dataloaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'blogs',
                    'findBlogByName'
                ]),
                blogs() {
                    var blog = this.findBlogByName("Bramalea City Centre").posts;
                    var vm = this;
                    var temp_blog = [];
                    _.forEach(blog, function(value, key) {
                        today = moment().tz(vm.timezone);
                        webDate = moment(value.publish_date).tz(vm.timezone);
                        if (today >= webDate) {
                            if (_.includes(value.image_url, 'missing')) {
                                value.image_url = "//codecloud.cdn.speedyrails.net/sites/5c0581a36e6f643f53050000/image/jpeg/1527006352000/bccblogplaceholder.jpg";
                            }
                            value.body_short = _.truncate(value.body, { 'length': 99, 'separator': ' ' });
                            
                            temp_blog.push(value);
                        }
                    });
                    blog = _.reverse(_.sortBy(temp_blog, function (o) { return o.publish_date }));
                    return blog
                },
                firstPost() {
                    var first_post = _.slice(this.blogs, 0, 1);
                    return first_post
                },
                blogList() {
                    var blog_list = _.drop(this.blogs);
                    return blog_list
                }
            },
            methods: {
                loadData: async function () {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "blogs")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                handleButton: function () {
                    if(!this.morePostsFetched){
                        this.morePosts = this.blogList;
                        this.posts = this.morePosts.splice(0, 3);
                        this.morePostsFetched = true;
                    } else {
                        var nextPosts = this.morePosts.splice(0, 3);
                        // Add 3 more posts to posts array
                        var vm = this;
                        _.forEach(nextPosts, function(value, key) {
                            vm.posts.push(value);
                        });
                    }
                    if(this.blogList.length === 0){
                        this.noMorePosts = true
                        this.noPosts = true
                    } else {

                    }
                },
                shareURL(slug) {
                    var share_url = "https://www.bramaleacitycentre.com/posts/" + slug
                    return share_url
                }
            }
        });
    });
</script>