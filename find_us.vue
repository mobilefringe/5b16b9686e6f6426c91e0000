<template>
	<div class="" id="find_us_container">
	    <div class="page_header all_caps double_border_bottom">
			<h2 class="page_container text_left page_header_title"> Find Us</h2>
		</div>
		<div style="height: 400px;">
	        <google-map :property="property" :zoom="18"></google-map>
	    </div>
		<div class="page_container">
			<div style="margin-top:20px" class="row text-left" v-if="currentPage" v-html="currentPage.body">
			    
			</div>
		</div>
	</div>
</template>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta", 'vee-validate', "vue!google_map"], function(Vue, Vuex, moment, tz, VueMoment, Meta, VeeValidate, GoogleMapAPI) {
        Vue.use(Meta);
        Vue.use(VeeValidate);
        return Vue.component("find-us-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    success_subscribe: false,
                    currentPage: null
                }
            },
            
            created () {
                host_name = this.property.mm_host.replace("http:", "");
                this.$store.dispatch('LOAD_PAGE_DATA', {
                    url: host_name + "/pages/holtrenfrew-directions.json"
                }).then(response => {
                    this.currentPage = response.data;
                }, error => {
                    console.error("Could not retrieve data from server. Please check internet connection and try again.");
                    this.$router.replace({ path: '/'});
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                ]),
            }
        });
    });
</script>