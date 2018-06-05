<template>
	<div class="" id="find_us_container">
	    <div class="page_header all_caps double_border_bottom">
			<h2 class="page_container text_left page_header_title"> Find Us</h2>
		</div>
		<div class="page_container  padding_top_40">
			<div class="row text-left">
			    <div style="height: 300px;margin-bottom:-30px;">
			        <google-map :property="property"></google-map>
			    </div>
			</div>
			<div class="padding_top_40"></div>
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
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                ]),
            }
        });
    });
</script>