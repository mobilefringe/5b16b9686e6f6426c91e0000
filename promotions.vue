<template>
	<div class="page_container promotions_container" v-if="dataloaded"><!-- for some reason if you do not put an outer container div this component template will not render -->
		<div class="date_bar">
		    <span v-on:click="beforeDate()"> <i class="fa fa-chevron-left"></i></span>
		    <span class="current_date">
		        {{selectedDate}}
		    </span>
		    <span v-on:click="afterDate()"> <i class="fa fa-chevron-right"></i></span>
		</div>
		<div class="all_dates text-center">
		    <span v-for="n in daysInMonth" :id="'date_'+n.date" v-on:click="newDate(n.date)" :class="{active: n.isActive}">{{n.date}}</span>
		</div>
		<div class="promo_container" v-if="filteredPromos.length > 0">
			<div class="row promo_dets is-table-row" v-for="promo in filteredPromos">
				<div class="col-sm-7" >
					<div class="promo_div_image" v-lazy-container="{ selector: 'img' }">
						<img :data-src="promo.store.image_url"  data-loading='//codecloud.cdn.speedyrails.net/sites/5b16b9686e6f6426c91e0000/image/png/1521035009104/Screen Shot 2018-03-14 at 9.43.24 AM.png' alt=""/>
					</div>
				</div>
				<div class="col-sm-5 promo_div_dets text-left">
					<p class="promo_div_name">{{promo.name}}</p>
					<p class="promo_div_store_name">{{promo.store.name | uppercase}}</p>
					<p class="promo_div_date">{{promo.start_date | moment("MMM D", timezone)}} - {{promo.end_date | moment("MMM D", timezone)}}</p>
					<p class="promo_div_description">{{promo.description_short}}</p>
					<div class="promo_feature_share row is-table-row">
    					<span class="feature_read_more col-sm-8">
    						<router-link :to="'/promotions/'+promo.slug" class="mobile_readmore" >
    							<p class="feature-readmore" :aria="promo.name">View Promotion Details <i class="fa fa-chevron-right pull-right" aria-hidden="true"></i></p>
    						</router-link>
    					</span>
    					<div class="text-right  col-sm-4" v-if="promo">
        					<social-sharing :url="shareURL(promo.slug)" :title="promo.title" :description="promo.body" :quote="_.truncate(promo.description, {'length': 99})" twitter-user="" :media="promo.image_url" inline-template >
                            <div class="blog-social-share">
                                <div class="social_share">
                                    <network network="facebook">
                                    <img src="//codecloud.cdn.speedyrails.net/sites/5b16b9686e6f6426c91e0000/image/png/1511797683914/facebook.png" class="social_icons" alt="">
                                </network>
                                <network network="twitter">
                                    <img src="//codecloud.cdn.speedyrails.net/sites/5b16b9686e6f6426c91e0000/image/png/1511797704155/twitter.png" class="social_icons" alt="">
                                </network>
                                </div>
                            </div>
                        </social-sharing>
    					</div>
    					
					</div>
				</div>
			</div>
		</div>
		<div class="promo_container" v-else>
		    <p style="padding:20px"> Sorry, No sales or promotions available for this day. </p>
		</div>
	</div>
</template>

<style>
    .date_bar{
        /* Today: */
        background: #D3D3D3;
        height: 40px;
        line-height: 40px;
        margin: auto;
        text-align: center;
    }
    .date_bar .fa{
        cursor: pointer;
    }
    .current_date{
        color: #4a4a4a;
        padding: 0 10px;
    }
    .all_dates {
        border-bottom: 1px solid #aea99e;
        display: flex;
        flex-wrap: wrap;
    }
    .all_dates span {
        font-size: 16px;
        color: #000000;
        letter-spacing: 1.5px;
        height: 40px;
        line-height: 40px;
        padding: 5px;
        cursor: pointer;
    }
     .all_dates [class*="date_"]:focus, [class*="date_"]:hover { 
        background-color: #D3D3D3;
    }
    .all_dates span.active { 
        background-color: #bababa;
    }
    .promo_dets {
        border-bottom: 1px solid #aea99e;
    }
    .row.is-table-row {
        margin: 0;
    }
    .row.is-table-row [class*="col-"] {
        padding:0;
    }
    .feature_read_more {
        width : auto;
    }
    .social_share span {
        /*display:inline-block;*/
        width: 24px;
        height: 24px;
        cursor: pointer;
    }
    .social_share .social_icons{
        width : 24px;
        height : 24px;
        display:inline;
        margin: 0 2px;
    }
</style>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta", "vue-lazy-load"], function(Vue, Vuex, moment, tz, VueMoment, Meta, VueLazyload) {
        Vue.use(Meta);
        Vue.use(VueLazyload);
        return Vue.component("promos-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    selectedDate: null,
                    filteredPromos:[],
                    dataloaded: false
                }
            },
            created() {
                this.$store.dispatch("getData", "promotions").then(response => {
                    this.dataloaded = true;
                    this.selectedDate = moment().tz(this.timezone).format('MMM D, YYYY');
                    var date = moment(this.selectedDate).date();
                    this.daysInMonth[date-1].isActive = true;
                }, error => {
                  console.error("Could not retrieve data from server. Please check internet connection and try again.");
                });
            },
            watch: {
                selectedDate: function() {
                    //sort promos
                    selected = moment(this.selectedDate).format('MM DD YYYY');
                    var vm = this;
                    vm.filteredPromos = _.filter(vm.promotions, function(val){
                        start_date = moment(val.start_date).tz(vm.timezone).format('MM DD YYYY');
                        end_date = moment(val.end_date).tz(vm.timezone).format('MM DD YYYY');
                        return (selected <= end_date && selected >= start_date || selected <= end_date);
                    });
                    //remove old active class, change active class 
                    old_date = moment(this.selectedDate).date();
                    this.daysInMonth.map(date => {
                        date.isActive = false;
                    });
                    this.daysInMonth[old_date-1].isActive = true;
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedPromos',
                ]),
                promotions() {
                    var vm = this;
                    var temp_promo = [];
                    var temp_job = [];
                    _.forEach(this.processedPromos, function(value, key) {
                        value.description_short = _.truncate(value.description, {
                            'length': 70
                        });
                        
                        if (value.store != null && value.store != undefined && _.includes(value.store.image_url, 'missing')) {
                            value.store.image_url = vm.property.default_logo_url;
                        }
                        else if ( value.store == null ||  value.store == undefined) {
                            value.store = {};
                            value.store.image_url =  vm.property.default_logo_url;
                        }
                        if (_.includes(value.image_url, 'missing')) {
                            value.image_url = vm.property.default_logo_url;
                        }
                        temp_promo.push(value);
                    });
                    _.sortBy(temp_promo, [function(o) { return o.start_date; }]);
                    return temp_promo;
                },
                daysInMonth() {
                    var days = moment(this.selectedDate).daysInMonth();
                    var all_days= [];
                    for(var i = 1; i <= days; i++) {
                        var temp_day = {};
                        temp_day.date = i;
                        temp_day.isActive = false;
                        
                        all_days.push(temp_day);
                    }
                    return all_days;
                }
            },
            methods: {
                beforeDate: function(ev) {
                    selectedDate = this.selectedDate;
                    selectedDate = moment(selectedDate).subtract(1, 'days');
                    this.selectedDate = moment(selectedDate).tz(this.timezone).format('MMM D, YYYY');
                },
                afterDate() {
                    selectedDate = this.selectedDate;
                    selectedDate = moment(selectedDate).add(1, 'days');
                    this.selectedDate = moment(selectedDate).tz(this.timezone).format('MMM D, YYYY');
                },
                newDate (val) {
                    old_date = moment(this.selectedDate).date();
                    month = moment(this.selectedDate).month();
                    year = moment(this.selectedDate).year();
                    this.selectedDate = moment([year, month, val]).format('MMM D, YYYY');
                },
                shareURL(slug){
                    var share_url = "http://holtrenfrewcentre.ca/promotions/" + slug;
                    return share_url;
                },
            }
        });
    });
</script>