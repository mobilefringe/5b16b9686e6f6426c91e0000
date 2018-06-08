<template>
	<div v-if="dataloaded" id="stores_container">
		<div class="page_header all_caps double_border_bottom">
			<h2 class="page_container text_left page_header_title"> Stores Directory & Map </h2>
		</div>
		<div class="page_container">
			<div class="store-list-left-block col-sm-4 padding_top_20">
			    <div style="position:relative; z-index:999999999;">
    			    <p class="text_left">Select Stores: </p>
    				<div class="alphabet-dd show_phone" >
    				    <v-select v-model="selectedStore" :options="filteredStores" :searchable="false" id="mobile_alpha_list" label="name" placeholder="Select a Store" ></v-select>
    			    </div>
			    </div>
		        <div id="mapsvg_store_detail_1" class="show_phone">
					<mapplic-png-map ref="pngmaprefmobile" :height="500" :minimap= "false" :deeplinking="false" :sidebar="false" :hovertip="true" :storelist="allStores" :floorlist="floorList" :bindLocationOpened="true" :svgWidth="1300" :svgHeight="787"  v-if="mobile_store"></mapplic-png-map>
				</div>
				<div class="store-list-container hidden_phone">
					<div class="dropdown_container hidden_phone">
						<h3 class="text_left">Select Category</h3>
						<div class="category-DD-div">
							<v-select v-model="selectedCat" :options="dropDownCats" :searchable="false" :on-change="filterByCategory"></v-select>
						</div>
						<h3 class="text_left" style=" margin-top: 10px;">Sort Alphabetically</h3>
						<div class="category-DD-div">
							<v-select v-model="selectedAlpha" :options="alphabet" :searchable="false" :on-change="filterStores" placeholder="All"></v-select>
						</div>
					</div>
					
					<ul class="store-listing text_left  padding_top_20">
						<li v-for="store in filteredStores" class="pointer">
							<p class="directory_store_name" v-on:click="dropPin(store)">{{store.name}}</p>
						</li>
					</ul>
				</div>
			</div>
			<div class="col-sm-8 padding_top_20">
				<div id="mapsvg_store_detail" class=" map">
					<mapplic-png-map ref="pngmap_ref" :height="500" :minimap= "false" :deeplinking="false" :sidebar="false" :hovertip="true" :storelist="allStores" :floorlist="floorList" :bindLocationOpened="true" :svgWidth="property.map_image_width" :svgHeight="property.map_image_height" :showPin="true" v-if="!mobile_store"></mapplic-png-map>
				</div>
			</div>
		</div>
	</div>
</template>

<style>
    #stores_container .v-select .open-indicator {
        right: 0 !important;
        top: 0 !important;
        bottom: initial !important;
        height: 100% !important;
        padding: 15px 25px 15px 20px !important;
        background-color: #9B9B9B !important;
        border-color: #fff !important;
        color: #fff !important;
    }

    #stores_container .v-select .open-indicator:before {
        border-color: #fff !important;
        margin-top: -5px;
    }

    #stores_container .v-select .dropdown-menu {
        font-family: arial;
        font-size: 14px;
        color: #000000;
        letter-spacing: 1.56px;
        display: block;
        transition: all 0.3s ease;
    }

    #stores_container .v-select .dropdown-toggle {
        text-align: left;
        border-radius: initial;
    }
    #stores_container .v-select .selected-tag{
        position:absolute;
    }
    #stores_container .v-select li.active.hightlight {
        cursor: none!important;
    }

	#stores_container .text {
	    color:white;
	    font-size:16px;
	    padding-top:2px;
	}

</style>

<script>
    define(["Vue", "vuex", "vue-select", "vue!mapplic-png-map"], function(Vue, Vuex, VueSelect, MapplicComponent) {
        return Vue.component("stores-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    dataloaded: false,
                    selectedCat: "All",
                    selectedAlpha: null,
                    alphabet: ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z"],
                    filteredStores: null,
                    selectedStore: null,
                    mobile_store: false,
                    windowWidth: 0
                }
            },
            created (){
                this.loadData().then(response => {
                    this.dataloaded = true;
                    this.filteredStores = this.allStores;
                    
                    this.$on('updateMap', this.updatePNGMap);
                });
            },
            watch: {
                windowWidth: function() {
                    if (this.windowWidth <= 768) {
                        this.mobile_store = true;
                    } else {
                        this.mobile_store = false;
                    }
                },
                selectedStore () {
                    this.dropPin(this.selectedStore);
                }
            },
            mounted() {
                this.$nextTick(function() {
                    window.addEventListener('resize', this.getWindowWidth);
                    //Init
                    this.getWindowWidth();
                });
            },
            methods: {
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch("getData", "categories")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                filterByCategory(category_id) {
                    if (category_id == "All" || category_id == null || category_id == undefined) {
                        category_id = "All";
                    } else {
                        category_id = this.findCategoryByName(category_id).id;
                    }

                    this.breakIntoCol = false;
                    if (category_id == "All") {
                        this.filteredStores = this.allStores; //this.storesByAlphaIndex;
                        // this.breakIntoCol = true;
                    } else {

                        var find = this.findCategoryById;
                        var filtered = _.filter(this.allStores, function(o) {
                            return _.indexOf(o.categories, _.toNumber(category_id)) > -1;
                        });
                        this.filteredStores = filtered;
                    }
                    // this.filteredStores = 
                },
                updatePNGMap(map) {
                    this.map = map;
                },
                getWindowWidth(event) {
                    this.windowWidth = window.innerWidth;
                },
                dropPin(store) {
                     if(this.windowWidth <= 768 && this.selectedStore) {
                        this.pngMapRef.showLocation(store.id);
                    }
                    else if(this.windowWidth > 768) {
                        this.pngMapRef.showLocation(store.id);
                    }
                },
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedStores',
                    'processedCategories',
                    'storesByAlphaIndex',
                    'storesByCategoryName',
                    'findCategoryById',
                    'findCategoryByName',
                    'findRepoByName'

                ]),
                allStores() {
                    this.processedStores.map(function(store){
                        store.zoom = 2;
                    })
                    return this.processedStores;
                },
                allCatergories() {
                    return this.processedCategories;
                },
                dropDownCats() {
                    var cats = _.map(this.$store.getters.processedCategories, 'name');
                    cats.unshift('All');
                    return cats;
                },
                findCategoryById() {
                    return this.$store.getters.findCategoryById;
                },
                findCategoryByName() {
                    return this.$store.getters.findCategoryByName;
                },
                getPNGurl() {
                    return "https://www.mallmaverick.com" + this.property.map_url;
                },
                storeNames () {
                    return _.map(this.filteredStores, 'name');
                },
                pngMapRef() {
                    var reference = null; 
                    if(this.windowWidth <= 768) {
                        reference = this.$refs.pngmaprefmobile;
                    }
                    else {
                        reference = this.$refs.pngmap_ref;
                    }
                    return reference;
                },
                filterStores() {
                    letter = this.selectedAlpha;
                    if (letter == "All") {
                        this.filteredStores = this.allStores;
                    } else {
                        var filtered = _.filter(this.allStores, function(o, i) {
                            return _.lowerCase(o.name)[0] == _.lowerCase(letter);
                        });
                        this.filteredStores = filtered;
                    }
                },
                floorList () {
                    var floor_list = [];
                    
                    var floor_1 = {};
                    floor_1.id = "first-floor";
                    floor_1.title = "Floor 1";
                    floor_1.map = this.getPNGurl;
                    floor_1.z_index = 1;
                    floor_1.show = true;
                    
                    floor_list.push(floor_1);
                    return floor_list;
                }
                
            },
            beforeDestroy: function() {
                window.removeEventListener('resize', this.getWindowWidth);
            }
        });
    });
</script>