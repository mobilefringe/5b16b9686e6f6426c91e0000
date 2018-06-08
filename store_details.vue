<template>
	<div class="page_container store_dets_container" v-if="dataLoaded" id="store_dets_container">
		<div class="row">
			<div class="col-sm-4 store_logo_container">
				<div>
					<img :src="currentStore.store_front_url_abs" :alt="currentStore.name"/>
				</div>
			</div>
			<div class="col-sm-8 store_map_container">
				<div id="mapsvg_store_detail">
					<!--<png-map ref="pngmapref" v-bind:png-map-url="getPNGurl" v-bind:initial-position="'500 400'" @updateMap="updatePNGMap" :key="currentStore.id"></png-map>-->
					<mapplic-png-map ref="pngmap_ref" :height="300" :hovertip="true" :storelist="allStores" :floorlist="floorList" :svgWidth="property.map_image_width" :svgHeight="property.map_image_height" @updateMap="updatePNGMap"></mapplic-png-map>
				</div>
			</div>
		</div>
		<div class="row">
			<div class="col-sm-4 store_details_container">
				<div>
					<h1>{{currentStore.name}}</h1>
					<p>{{currentStore.category_name}}</p>
					<p>{{currentStore.phone}}</p>
					<div class="margin_20">
					    <a v-bind:href="'//'+currentStore.website" target="_blank">Visit Store Site</a>    
					</div>
					
					<div v-if="currentStore.store_hours.length > 0">
					    <p class="underline">Store Hours</p>
					    <ul class="store_details_hours">
                            <li v-if="!hour.is_closed" v-for="hour in hours">
                                <span class="day">{{hour.day_of_week | moment("dddd", timezone)}}: </span><span class="hour">{{hour.open_time | moment("h:mma", timezone)}} - {{hour.close_time | moment("h:mma", timezone)}}</span>
                            </li>
                            <li v-else>
                                <span class="day">{{hour.day_of_week | moment("dddd", timezone)}}: </span><span class="hour">CLOSED</span>
                            </li>
                        </ul>
					</div>
				</div>
			</div>
			<div class="col-sm-8 store_desc_container">
				<div class="text-left store_description">
					<p>{{currentStore.description}}</p>
				</div>
			</div>
		</div>
		<div class="store_promo_container" v-if="currentStore && currentStore.total_published_promos > 0">
		    <div class="promo_container_title text-left all_caps"> Sales & Promotions</div>
		    <div class="row store_promo_dets text-left" v-for="promo in promotions">
		        <div class="col-sm-7" >
		        <div class="promo_div_image">
		            <img :src="promo.image_url" alt=""/>
		        </div>
		        </div>
		        <div class="col-sm-5 promo_div_dets">
		            <p class="promo_div_name">{{promo.name}}</p>
		            <p class="promo_div_store_name">{{currentStore.name | uppercase}}</p>
		            <p class="promo_div_date">{{promo.start_date | moment("MMM D", timezone)}} - {{promo.end_date | moment("MMM D", timezone)}}</p>
					<p class="promo_div_description">{{promo.description_short}}</p>
					<span class="feature_read_more">
						<router-link :to="'/promotions/'+promo.slug" class="mobile_readmore" >
							<p class="feature-readmore">Read More <i class="fa fa-chevron-right pull-right" aria-hidden="true"></i></p>
						</router-link>
					</span>
		        </div>
		    </div>
		</div>
		<div class="store_promo_container" v-if="currentStore && currentStore.total_published_jobs > 0">
		    <div class="promo_container_title text-left all_caps"> Careers</div>
		    <div class="row store_promo_dets text-left" v-for="promo in jobs">
		        <div class="col-sm-7" >
		        <div class="promo_div_image">
		            <img :src="currentStore.store_front_url_abs" alt=""/>
		        </div>
		        </div>
		        <div class="col-sm-5 promo_div_dets">
		            <p class="promo_div_name">{{promo.name}}</p>
		            <p class="promo_div_store_name">{{currentStore.name | uppercase}}</p>
		            <p class="promo_div_date">{{promo.start_date | moment("MMM D", timezone)}} - {{promo.end_date | moment("MMM D", timezone)}}</p>
		            <p class="promo_div_description">{{promo.description_short}}</p>
					<span class="feature_read_more">
						<router-link :to="'/promotions/'+promo.slug" class="mobile_readmore" >
							<p class="feature-readmore">Read More <i class="fa fa-chevron-right pull-right" aria-hidden="true"></i></p>
						</router-link>
					</span>
		        </div>
		    </div>
		</div>
	</div>
</template>
<style>
    .store_logo_container,
    .store_map_container,
    .store_details_container,
    .store_desc_container{
        padding: 20px 10px;
    }
    
    .store_logo_container img{
        border: 1px solid #aea99e;
        width:300px;
        height:300px
    }
    .promo_container_title{
        border-top:1px solid #aea99e;
        border-bottom:1px solid #aea99e;
        height: 35px;
        line-height: 35px;
    }
    #store_dets_container #mapsvg_store_detail {
        border: 1px solid #aea99e;
    }
    #store_dets_container .map {
        height: 300px;
        overflow:hidden;
    }
	#store_dets_container .mapplic-popup-link {
	    display:none!important;
	}
	#store_dets_container .mapplic-popup-link, .mapplic-tooltip-close {
	    display:none!important;
	}
	#store_dets_container .mapplic-tooltip-content {
	    margin-right: auto;
	}
	#store_dets_container .mapplic-tooltip-title {
        text-align: center;
        margin: auto!important;
	}
</style>
<script>
    define(['Vue', 'vuex', 'moment', "vue!mapplic-png-map"], function(Vue, Vuex, moment, MapplicComponent) {
        return Vue.component("store-details-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    currentStore: null,
                    promotions : [],
                    jobs:[],
                    hours: [],
                    dataLoaded: false
                }
            },
            props:['id'],
            beforeRouteUpdate(to, from, next) {
                this.currentStore = this.findStoreBySlug(to.params.id);
                if (this.currentStore === null || this.currentStore === undefined){
                    this.$router.replace({ name: '404'});
                }
                next();
            },
            created (){
                this.loadData().then(response => {
                    this.dataLoaded = true;
                    this.updateCurrentStore(this.id);
                });
            },
            watch: {
                currentStore: function() {
                    this.currentStore.zoom = 2;
                    if ( _.includes(this.currentStore.store_front_url_abs, 'missing')) {
                        this.currentStore.store_front_url_abs = this.property.default_logo_url;
                    }
                    
                    var vm = this;
                    var storeHours = [];
                    _.forEach(this.currentStore.store_hours, function (value, key) {
                        storeHours.push(vm.findHourById(value));
                    });
                    this.hours = storeHours;
                    
                  
                    var temp_promo = [];
                    var temp_job = [];
                    _.forEach(this.currentStore.promotions, function(value, key) {
                        var current_promo = vm.findPromoById(value);
                        current_promo.description_short = _.truncate(current_promo.description, {
                            'length': 70
                        });
                        temp_promo.push(current_promo);
                    });
                    _.forEach(this.currentStore.jobs, function(value, key) {
                        var current_job = vm.findJobById(value);
                        current_job.description_short = _.truncate(current_job.description, {
                            'length': 70
                        });
                        temp_job.push(current_job);

                    })
                    this.promotions = temp_promo;
                    this.jobs = temp_job;
                    
                    // setTimeout(function() {
                    //     vm.addLandmark(vm.currentStore);
                    // }, 500);
                },
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedStores',
                    'findStoreBySlug',
                    'findHourById',
                    'findPromoById',
                    'findJobById',
                ]),
                getPNGurl () {
                    return "https://www.mallmaverick.com" + this.property.map_url;
                },
                pngMapRef() {
                    return this.$refs.pngmapref;
                },
                allStores() {
                    this.processedStores.map(function(store){
                        store.zoom = 4;
                    })
                    return this.processedStores;
                },
                floorList () {
                    var floor_list = [];
                    
                    var floor_1 = {};
                    floor_1.id = "first-floor";
                    floor_1.title = "Level One";
                    floor_1.map = this.getPNGurl;
                    floor_1.z_index = 1;
                    floor_1.show = true;
                    floor_list.push(floor_1);
                    return floor_list;
                }
            },
            methods: {
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch("getData","promotions"), this.$store.dispatch("getData", "jobs")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                updateCurrentStore (id) {
                    this.currentStore = this.findStoreBySlug(id);
                    if (this.currentStore === null || this.currentStore === undefined){
                        this.$router.replace({ path: '/'});
                    }
                },
                updatePNGMap(map) {
                    this.map = map;
                    this.dropPin(this.currentStore);
                },
                dropPin(store) {
                    this.$refs.pngmap_ref.showLocation(store.id);
                },
            }
        });
    });
</script>