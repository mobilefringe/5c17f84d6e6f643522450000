<template>
    <div v-if="pageBanner"> <!-- for some reason if you do not put an outer container div this component template will not render -->
        <div class="page_header" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
			<div class="site_container">
				<div class="header_content caps">
					<h1>{{$t("hours_page.hours")}}</h1>
					<h2 style="display:none;">Scroll to  view hours</h2>
				</div>
			</div>
		</div>  
        <div class="site_container">
            <div class="all_hours_container">
                <div class="row">
                    <div class="col-sm-6">
                        <h3 class="hours_heading text-left" >{{$t("hours_page.reg_hours")}}</h3>
                        <div id="hours_container" class="hours_container">
                            <div class="hours_div text-left" v-for="hour in hours">
                                <span>{{hour.day_of_week | moment("dddd", timezone)}}: </span>
                                <span>{{hour.open_time | moment("h:mm A", timezone)}} - {{hour.close_time | moment("h:mm A", timezone)}}</span>
                            </div>
                        </div>
                    </div>
                    <div class="col-sm-6 ">
                        <h3 class="hours_heading text-left">{{$t("hours_page.holiday_hours")}}</h3>
                        <div id="holidays_hours_container" class="hours_container">
                            <div class="hours_div text-left" v-for="hour in reducedHolidays">
                                <span v-if="hour.holiday_name">
                                    <span v-if="locale=='en-ca'">{{ hour.holiday_name }}</span>
                                    <span v-else>{{ hour.holiday_name_2 }}</span>
                                    ({{ hour.holiday_date | moment("MMM D YYYY", timezone) }}):
                                </span>
                                <span v-else>
                                    {{ hour.holiday_date | moment("MMM D YYYY", timezone) }}:
                                </span>
                                <span class="opening_hours">{{hour.open_time | moment("h:mm A", timezone)}} - {{hour.close_time | moment("h:mm A", timezone)}}</span>
                            </div>
                        </div>
                        <div class="padding_top_20"></div>
                        <h3 class="hours_heading text-left">{{$t("hours_page.holiday_closures")}}</h3>
                        <div id="closed_hours_container" class="hours_container">
                            <div class="hours_div text-left" v-for="hour in closeHolidays">
                                <span>
                                    <span v-if="locale=='en-ca'">{{hour.holiday_name}}</span>
                                    <span v-else>{{hour.holiday_name_2}}</span>
                                    ({{hour.holiday_date | moment("MMM D YYYY", timezone)}})
                                </span>
                            </div>
                        </div>
                        
                    </div>
                </div>
                <div class="padding_top_40"></div>
            </div>
        </div >
    </div>
</template>
<style>
    .hours_page_container .row{
        margin-left:inherit;
    }
</style>
<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta"], function(Vue, Vuex, moment, tz, VueMoment, Meta) {
        Vue.use(Meta);
        return Vue.component("hours-component", {
            template: template, // the variable template will be injected
            props:['locale'],
            data: function() {
                return {
                    pageBanner : null
                }
            },
            created() {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Hours Banner');
                    if (temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    } else {
                        this.pageBanner= {};
                        this.pageBanner.image_url = "";
                    }
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'getPropertyHours',
                    'getPropertyHolidayHours',
                    'getPropertyExtendedHours',
                    'findRepoByName'
                ]),
                hours () {
                   var hours = _.sortBy(this.getPropertyHours, ['day_of_week']);
                    // var ordered_hours = [];
                    // _.forEach(hours, function (val, key) {
                    // //   if(val.day_of_week !== 0) {
                    // //       ordered_hours.push(val);
                    // //   }
                    // });
                    // ordered_hours.push(hours[0]);
                    // return ordered_hours;
                    return hours
                },
                holidayHours () {
                    return _.sortBy(this.getPropertyHolidayHours,['holiday_date']);
                },
                reducedHolidays () {
                    var holidayHours = _.filter(this.holidayHours, function(o) { return !o.is_closed; });
                    var extendedHours = this.getPropertyExtendedHours;
                    var open_holidays = _.concat(holidayHours, extendedHours);
                    console.log("open_holidays", open_holidays)
                    var holidays = [];
                    _.forEach(open_holidays, function(val, key) {
                        var today = moment().format('X');
                        var holiday_date = moment(val.holiday_date).format('X');
                        if (today < holiday_date) {
                            holidays.push(val);
                        }
                    });
                    return _.sortBy(holidays, function(o) { return o.holiday_date; });
                    // return _.filter(holidayHours, function(o) { return !o.is_closed; });
                },
                closeHolidays () {
                    var holidayHours = this.holidayHours;
                    var closed_holidays = _.filter(holidayHours, function(o) { return o.is_closed; });
                    var holidays = [];
                    _.forEach(closed_holidays, function(val, key) {
                        var today = moment().format('X');
                        var holiday_date = moment(val.holiday_date).format('X');
                        if (today < holiday_date) {
                            holidays.push(val);
                        }
                    });
                    return _.sortBy(holidays, function(o) { return o.holiday_date; });
                },
                extendedHours () {
                    var extendedHours = this.getPropertyExtendedHours;
                    return _.sortBy(extendedHours, function(o) { return o.holiday_date; });
                }
            },
            methods : {
                loadData: async function() {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "repos")]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
            }
        });
    });
</script>
