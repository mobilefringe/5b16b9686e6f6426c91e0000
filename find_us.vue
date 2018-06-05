<template>
	<div class="page_container" id="find_us_container">
		<div class="margin_25_across padding_top_40">
			<div class="row text-left">
			    <google-map :property="property"></google-map>
			</div>
			<div class="padding_top_40"></div>
		</div>
	</div>
</template>

<style>
    #find_us_container .row{
        margin-left:inherit;
    }
    #find_us_container {
        margin: 20px auto 0;
    }
    #find_us_container .directions {
        padding: 10px 0;   
    }
    .form-group .form-control-feedback{
        font-size: 12px;
        top:50px;
        color: #F44336;
    }
</style>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta", 'vee-validate'], function(Vue, Vuex, moment, tz, VueMoment, Meta, VeeValidate) {
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