<template>
	<div class="page_container events_container" v-if="dataLoaded"><!-- for some reason if you do not put an outer container div this component template will not render -->
		<div class="promo_container" v-if="events.length > 0">
			<div class="row promo_dets is-table-row" v-for="promo in events">
				<div class="col-sm-7" >
					<div class="promo_div_image" v-lazy-container="{ selector: 'img' }">
						<img :data-src="promo.image_url"  data-loading='//codecloud.cdn.speedyrails.net/sites/5b16b9686e6f6426c91e0000/image/png/1521035009104/Screen Shot 2018-03-14 at 9.43.24 AM.png' alt=""/>
					</div>
				</div>
				<div class="col-sm-5 promo_div_dets text-left">
					<p class="promo_div_name">{{promo.name}}</p>
					<p class="promo_div_date">{{promo.start_date | moment("MMM D", timezone)}} - {{promo.end_date | moment("MMM D", timezone)}}</p>
					<p class="promo_div_description">{{promo.description_short}}</p>
					<div class="promo_feature_share row is-table-row">
    					<span class="feature_read_more col-sm-8">
    						<router-link :to="'/events/'+promo.slug" class="mobile_readmore" >
    							<p class="feature-readmore" :aria="promo.name">View Event Details<i class="fa fa-chevron-right pull-right" aria-hidden="true"></i></p>
    						</router-link>
    					</span>
    					<div class="text-right  col-sm-4" v-if="promo">
        					<social-sharing :url="shareURL(promo.slug)" :title="promo.title" :description="promo.body" :quote="_.truncate(promo.description, {'length': 99})" twitter-user="EastgateSquare" :media="promo.image_url" inline-template >
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
		    <p style="padding:20px"> Sorry, no events are currently available.</p>
		</div>
	</div>
</template>

<style>
    .events_container .date_bar{
        /* Today: */
        background: #D3D3D3;
        height: 40px;
        line-height: 40px;
        margin: auto;
        text-align: center;
    }
    .events_container .date_bar .fa{
        cursor: pointer;
    }
    .events_container .current_date{
        color: #4a4a4a;
        padding: 0 10px;
    }
    .events_container .all_dates {
        border-bottom: 1px solid #aea99e;
    }
    .events_container .all_dates span {
        font-size: 16px;
        color: #000000;
        letter-spacing: 1.5px;
        height: 30px;
        line-height: 30px;
        padding: 0 5px;
        cursor: pointer;
    }
    .events_container .all_dates [class*="date_"]:focus, [class*="date_"]:hover { 
        background-color: #D3D3D3;
    }
    .events_container .all_dates span.active { 
        background-color: #bababa;
    }
    .events_container .promo_dets {
        border-bottom: 1px solid #aea99e;
    }
    .events_container .row.is-table-row {
        margin: 0;
    }
    .events_container .row.is-table-row [class*="col-"] {
        padding:0;
    }
    .events_container .feature_read_more {
        width : auto;
    }
    .events_container .social_share network {
        display:inline-block;
        width: 24px;
        height: 24px;
        cursor: pointer;
    }
    .events_container .social_share .social_icons{
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
        return Vue.component("events-component", {
            template: template, // the variable template will be injected
            data () {
              return {
                  dataLoaded: false
              }  
            },
            created () {
                this.$store.dispatch("getData", "events").then(response => {
                   this.dataLoaded = true;
                }, error => {
                    console.error("Could not retrieve data from server. Please check internet connection and try again.");
                    this.$router.replace({ path: '/'});
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedEvents'
                ]),
                events() {
                    var vm = this;
                    var temp_promo = [];
                    var temp_job = [];
                    _.forEach(this.processedEvents, function(value, key) {
                        value.description_short = _.truncate(value.description, {
                            'length': 70
                        });
                        if (value.store != null && value.store != undefined && _.includes(value.store.image_url, 'missing')) {
                            value.store.image_url = vm.property.default_logo_url;
                        }
                        if (_.includes(value.image_url, 'missing')) {
                            value.image_url = vm.property.default_logo_url;
                        }
                            
                        temp_promo.push(value);
                    });
                    
                    _.sortBy(temp_promo, [function(o) { return o.start_date; }]);
                    return temp_promo;
                }
            },
            methods: {
                shareURL(slug){
                    var share_url = "http://holtrenfrewcentre.ca/events/" + slug;
                    return share_url;
                }
            }
        });
    });
</script>