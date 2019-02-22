<template>
    <div v-if="dataloaded">
        <div class="margin_60"></div>
		<div class="site_container inside_page_content page_content" v-if="currentPost">
            <div class="row">
                <div class="col-md-12">
                    <div class="post_details_image">
                        <img :src="currentPost.image_url" :alt="currentPost.title" />
                    </div>
                </div>
            </div>
            <div class="row">
                <div class="margin_60"></div>
                <div class="col-md-8">
                    <div class="post_details_container right_border">
                        <h2 class="post_heading caps">{{ currentPost.title }}</h2>
                        <p class="post_dates">{{ currentPost.publish_date | moment("MMM DD, YYYY", timezone) }}</p>
                        <social-sharing :url="shareURL(currentPost.slug)" :title="currentPost.title" :description="currentPost.body" :quote="_.truncate(currentPost.description, {'length': 99})" :twitter-user="$root.twitter_user" :media="currentPost.image_url" inline-template >
							<div class="blog-social-share">
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
                        <div class="margin_60 post_text" v-html="currentPost.html_body"></div>
                        <router-link to="/posts">
                            <button class="contact_btn">Back to Blog</button>
                        </router-link>
                    </div>
                </div>
                <div class="col-md-4">
                    <div class="previous_post_container">
                        <h3 class="previous_post_title">Previous Posts</h3>
                        <ul v-if="relatedPosts.length > 0">
                            <li v-for="post in relatedPosts">
                                <router-link :to="{ name: 'postDetails', params: { id: post.slug }}" class="" :aria="post.title">
					                <h4>{{ post.title }}</h4>
				                </router-link>
                            </li>
                        </ul>
                        <p v-else>No previous posts</p>
                    </div>   
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-social-sharing"], function (Vue, Vuex, moment, tz, VueMoment, SocialSharing) {
        return Vue.component("post-details-component", {
            template: template, // the variable template will be injected,
            props: ['id'],
            data: function () {
                return {
                    dataloaded: false,
                    currentPost: null
                }
            },
            created() {
                this.loadData().then(response => {
                    this.updateCurrentBlog(this.id);
                    this.dataloaded = true;
                });
            },
            watch: {
                $route: function () {
                    this.updateCurrentBlog(this.$route.params.id);
                },
                currentPost: function () {
                    if(this.currentPost != null){
                        if (_.includes(this.currentPost.image_url, 'missing')) {
                            this.currentPost.image_url = "//codecloud.cdn.speedyrails.net/sites/5c0581a36e6f643f53050000/image/jpeg/1527006352000/bccblogplaceholder.jpg";
                        }
                    }
                } 
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'blogs',
                    'findBlogByName',
                    'findBlogPostBySlug'
                ]),
                relatedPosts () {
                    var blog_posts = _.reverse(_.orderBy(this.findBlogByName("Bramalea City Centre").posts, function (o) { return o.publish_date }));

                    var current_post_date = this.currentPost.publish_date
                    var current_post_id = this.currentPost.id

                    var prev_posts = [];
                    _.forEach(blog_posts, function(value, key) {
                        if (value.id != current_post_id) {
                            if (value.publish_date <= current_post_date){
                                prev_posts.push(value); 
                            }
                        }
                    });
                    prev_posts = _.slice(prev_posts, 0, 3)
                    return prev_posts
                }
            },
            methods: {
                loadData: async function () {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "blogs")]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                updateCurrentBlog(id) {
                    var blogName = "Bramalea City Centre";
                    this.currentPost = this.findBlogPostBySlug(blogName, id);
                    if (this.currentPost === null || this.currentPost === undefined) {
                        this.$router.replace({ name: '404' });
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