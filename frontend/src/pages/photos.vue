<template>
    <div class="p-page p-page-photos" v-infinite-scroll="loadMore" infinite-scroll-disabled="loadMoreDisabled"
         infinite-scroll-distance="10" infinite-scroll-listen-for-event="infiniteScrollRefresh">
        <v-form ref="form" lazy-validation @submit="formChange" dense>
            <v-toolbar flat color="blue-grey lighten-4">
                <v-text-field class="pt-3 pr-3"
                              single-line
                              label="Search"
                              prepend-inner-icon="search"
                              clearable
                              color="blue-grey"
                              @click:clear="clearQuery"
                              v-model="filter.q"
                              @keyup.enter.native="formChange"
                              id="search"
                ></v-text-field>

                <v-spacer></v-spacer>

                <v-btn icon @click="advandedSearch = !advandedSearch" id="advancedMenu">
                    <v-icon>{{ advandedSearch ? 'keyboard_arrow_up' : 'keyboard_arrow_down' }}</v-icon>
                </v-btn>
            </v-toolbar>

            <v-card class="pt-1"
                    flat
                    color="blue-grey lighten-5"
                    v-show="advandedSearch">
                <v-card-text>
                    <v-layout row wrap>
                        <v-flex xs12 sm6 md3 pa-2 id="countriesFlex">
                            <v-select @change="formChange"
                                      label="Country"
                                      flat solo hide-details
                                      color="blue-grey"
                                      item-value="LocCountryCode"
                                      item-text="LocCountry"
                                      v-model="filter.country"
                                      :items="options.countries">
                            </v-select>
                        </v-flex>
                        <v-flex xs12 sm6 md3 pa-2 id="cameraFlex">
                            <v-select @change="formChange"
                                      label="Camera"
                                      flat solo hide-details
                                      color="blue-grey"
                                      item-value="ID"
                                      item-text="CameraModel"
                                      v-model="filter.camera"
                                      :items="options.cameras">
                            </v-select>
                        </v-flex>
                        <v-flex xs12 sm6 md3 pa-2 id="viewFlex">
                            <v-select @change="viewChange"
                                      label="View"
                                      flat solo hide-details
                                      color="blue-grey"
                                      v-model="view"
                                      :items="options.views"
                                      id="viewSelect">
                            </v-select>
                        </v-flex>
                        <v-flex xs12 sm6 md3 pa-2 id="timeFlex">
                            <v-select @change="formChange"
                                      label="Sort By"
                                      flat solo hide-details
                                      color="blue-grey"
                                      v-model="filter.order"
                                      :items="options.sorting">
                            </v-select>
                        </v-flex>
                    </v-layout>
                </v-card-text>
            </v-card>
        </v-form>
        <v-container fluid>
            <v-speed-dial
                    fixed
                    bottom
                    right
                    direction="top"
                    open-on-hover
                    transition="slide-y-reverse-transition"
                    style="right: 8px; bottom: 8px;"
            >
                <v-btn
                        slot="activator"
                        color="grey darken-2"
                        dark
                        fab
                >
                    <v-icon v-if="selected.length === 0">menu</v-icon>
                    <span v-else>{{ selected.length }}</span>
                </v-btn>
                <v-btn
                        fab
                        dark
                        small
                        color="deep-purple lighten-2"
                >
                    <v-icon>favorite</v-icon>
                </v-btn>
                <v-btn
                        fab
                        dark
                        small
                        @click.stop="dialog = true"
                        color="cyan accent-4"
                >
                    <v-icon>label</v-icon>
                </v-btn>
                <v-btn
                        fab
                        dark
                        small
                        color="teal accent-4"
                >
                    <v-icon>save</v-icon>
                </v-btn>
                <v-btn
                        fab
                        dark
                        small
                        @click.stop="dialog2 = true"
                        color="yellow accent-4"
                >
                    <v-icon>create_new_folder</v-icon>
                </v-btn>

                <v-btn
                        fab
                        dark
                        small
                        color="delete"
                >
                    <v-icon>delete</v-icon>
                </v-btn>
            </v-speed-dial>

            <p-photo-tiles v-if="view === 'tiles'" :photos="results" :selection="selected" :select="selectPhoto"
                           :open="openPhoto" :like="likePhoto"></p-photo-tiles>
            <p-photo-mosaic v-if="view === 'mosaic'" :photos="results" :selection="selected" :select="selectPhoto"
                            :open="openPhoto" :like="likePhoto"></p-photo-mosaic>
            <p-photo-details v-if="view === 'details'" :photos="results" :selection="selected"
                             :select="selectPhoto" :open="openPhoto" :like="likePhoto"></p-photo-details>
            <p-photo-list v-if="view === 'list'" :photos="results" :selection="selected" :select="selectPhoto"
                          :open="openPhoto" :like="likePhoto"></p-photo-list>
        </v-container>

        <v-dialog v-model="dialog" dark persistent max-width="600px">
            <v-card dark>
                <v-card-title>
                    <span class="headline">Edit tags</span>
                </v-card-title>
                <v-card-text>
                    <p>13 photos selected</p>
                    <h4>Add tags</h4>
                    <v-spacer></v-spacer>
                    <v-combobox
                            v-model="model"
                            :hide-no-data="!search"
                            :items="items"
                            :search-input.sync="search"
                            hide-selected
                            label="Add tags"
                            multiple
                            small-chips
                            solo
                    >
                        <template v-slot:no-data>
                            <v-list-tile>
                                <span class="subheading">Create</span>
                                <v-chip
                                        color="blue"
                                        label
                                        small
                                >
                                    search
                                </v-chip>
                            </v-list-tile>
                        </template>
                        <template v-slot:selection="{ item, parent, selected }">
                            <v-chip
                                    v-if="item === Object(item)"
                                    color="green"
                                    :selected="selected"
                                    label
                                    small
                            >
        <span class="pr-2">
          item text
        </span>
                                <v-icon
                                        small
                                        @click="parent.selectItem(item)"
                                >close
                                </v-icon>
                            </v-chip>
                        </template>
                        <template v-slot:item="{ index, item }">
                            <v-list-tile-content>
                                <v-text-field
                                        v-if="editing === item"
                                        v-model="editing.text"
                                        autofocus
                                        flat
                                        background-color="transparent"
                                        hide-details
                                        solo
                                        @keyup.enter="edit(index, item)"
                                ></v-text-field>
                                <v-chip
                                        v-else
                                        color="red"
                                        dark
                                        label
                                        small
                                >
                                    item text
                                </v-chip>
                            </v-list-tile-content>
                        </template>
                    </v-combobox>
                    <h4>Remove tags</h4>
                    <v-combobox
                            v-model="model"
                            :hide-no-data="!search"
                            :items="items"
                            :search-input.sync="search"
                            hide-selected
                            label="Remove tags"
                            multiple
                            small-chips
                            solo
                    >
                        <template v-slot:no-data>
                            <v-list-tile>
                                <span class="subheading">Create</span>
                                <v-chip
                                        color="blue"
                                        label
                                        small
                                >
                                    search
                                </v-chip>
                            </v-list-tile>
                        </template>
                        <template v-slot:selection="{ item, parent, selected }">
                            <v-chip
                                    v-if="item === Object(item)"
                                    color="green"
                                    :selected="selected"
                                    label
                                    small
                            >
        <span class="pr-2">
          item text
        </span>
                                <v-icon
                                        small
                                        @click="parent.selectItem(item)"
                                >close
                                </v-icon>
                            </v-chip>
                        </template>
                        <template v-slot:item="{ index, item }">
                            <v-list-tile-content>
                                <v-text-field
                                        v-if="editing === item"
                                        v-model="editing.text"
                                        autofocus
                                        flat
                                        background-color="transparent"
                                        hide-details
                                        solo
                                        @keyup.enter="edit(index, item)"
                                ></v-text-field>
                                <v-chip
                                        v-else
                                        color="red"
                                        dark
                                        label
                                        small
                                >
                                    item text
                                </v-chip>
                            </v-list-tile-content>
                        </template>
                    </v-combobox>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="success" flat @click="dialog = false">Cancel</v-btn>
                    <v-btn color="success" flat @click="dialog = false">Apply</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <v-dialog v-model="dialog2" dark persistent max-width="600px">
            <v-card dark>
                <v-card-title>
                    <span class="headline">Add to album</span>
                </v-card-title>
                <v-card-text>
                    <template>
                        <form>
                            13 photos selected
                            <v-select
                                    v-model="select"
                                    :items="items2"
                                    label="Album"
                            ></v-select>
                        </form>
                    </template>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="success" flat @click="dialog2 = false">Cancel</v-btn>
                    <v-btn color="success" flat @click="dialog2 = false">apply</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </div>
</template>

<script>
    import Photo from "model/photo";

    export default {
        name: 'photos',
        props: {},
        data() {
            const query = this.$route.query;
            const order = query['order'] ? query['order'] : 'newest';
            const camera = query['camera'] ? parseInt(query['camera']) : 0;
            const q = query['q'] ? query['q'] : '';
            const country = query['country'] ? query['country'] : '';
            const view = query['view'] ? query['view'] : 'tiles';
            const cameras = [{ID: 0, CameraModel: 'All Cameras'}].concat(this.$config.getValue('cameras'));
            const countries = [{
                LocCountryCode: '',
                LocCountry: 'All Countries'
            }].concat(this.$config.getValue('countries'));

            return {
                'advandedSearch': false,
                'results': [],
                'filter': {
                    country: country,
                    camera: camera,
                    order: order,
                    q: q,
                },
                'lastFilter': {},
                'options': {
                    'categories': [
                        {value: '', text: 'All Categories'},
                        {value: 'airport', text: 'Airport'},
                        {value: 'amenity', text: 'Amenity'},
                        {value: 'building', text: 'Building'},
                        {value: 'historic', text: 'Historic'},
                        {value: 'shop', text: 'Shop'},
                        {value: 'tourism', text: 'Tourism'},
                    ],
                    'views': [
                        {value: 'tiles', text: 'Tiles'},
                        {value: 'mosaic', text: 'Mosaic'},
                        {value: 'details', text: 'Details'},
                        {value: 'list', text: 'List'},
                    ],
                    'countries': countries,
                    'cameras': cameras,
                    'sorting': [
                        {value: 'newest', text: 'Newest first'},
                        {value: 'oldest', text: 'Oldest first'},
                        {value: 'imported', text: 'Recently imported'},
                    ],
                },
                'view': view,
                'pageSize': 60,
                'offset': 0,
                'loadMoreDisabled': true,
                'submitTimeout': false,
                'selected': [],
                'dialog': false,
                'search': null,
                'activator': null,
                'attach': null,
                'colors': ['green', 'purple', 'indigo', 'primary', 'success', 'orange'],
                'editing': null,
                'index': -1,
                'items': [
                    {header: 'Select a tag or create one'},
                    {text: 'Cat', color: 'primary'},
                    {text: 'Sun', color: 'red'},
                    {text: 'Dog', color: 'primary'},
                    {text: 'Holiday', color: 'primary'},
                    {text: 'Tiger', color: 'primary'},
                    {text: 'Soup', color: 'primary'},
                    {text: 'Night', color: 'primary'},
                    {text: 'Table', color: 'primary'},
                    {text: 'Apple', color: 'primary'},
                    {text: 'Frog', color: 'primary'},
                ],
                'nonce': 1,
                'menu': false,
                'model': [],
                'dialog2': false,
                'select': null,
                'items2': [
                    'South Africa',
                    'Cats & Dogs',
                    'Christmas 2018',
                ],
            };
        },
        methods: {
            clearSelection() {
                this.selected = [];
            },
            selectPhoto(photo) {
                const index = this.selected.indexOf(photo.ID);

                if (index === -1) {
                    this.selected.push(photo.ID);
                } else {
                    this.selected.splice(index, 1);
                }
            },
            likePhoto(photo) {
                photo.PhotoFavorite = !photo.PhotoFavorite;
                photo.like(photo.PhotoFavorite);
            },
            deletePhoto(photo) {
                this.$alert.success('Photo deleted');
            },
            openLocation(photo) {
                if (photo.PhotoLat && photo.PhotoLong) {
                    this.$router.push({name: 'Places', query: {lat: photo.PhotoLat, long: photo.PhotoLong}});
                } else if (photo.LocName) {
                    this.$router.push({name: 'Places', query: {q: photo.LocName}});
                } else if (photo.LocCity) {
                    this.$router.push({name: 'Places', query: {q: photo.LocCity}});
                } else if (photo.LocCountry) {
                    this.$router.push({name: 'Places', query: {q: photo.LocCountry}});
                } else {
                    this.$router.push({name: 'Places', query: {q: photo.CountryName}});
                }
            },
            viewChange() {
                this.updateQuery();
            },
            formChange() {
                this.refreshList();
            },
            clearQuery() {
                this.query.q = '';
                this.refreshList();
            },
            openPhoto(index) {
                this.$viewer.show(this.results, index)
            },
            loadMore() {
                if (this.loadMoreDisabled) return;

                this.loadMoreDisabled = true;

                this.offset += this.pageSize;

                const params = {
                    count: this.pageSize,
                    offset: this.offset,
                };

                Object.assign(params, this.lastFilter);

                Photo.search(params).then(response => {
                    this.results = this.results.concat(response.models);

                    this.loadMoreDisabled = (response.models.length < this.pageSize);

                    if (this.loadMoreDisabled) {
                        this.$alert.info('All ' + this.results.length + ' photos loaded');
                    }
                });
            },
            updateQuery() {
                const query = {
                    view: this.view
                };

                Object.assign(query, this.filter);

                this.$router.replace({query: query});

                this.$nextTick(() => this.$emit("infiniteScrollRefresh"));
            },
            refreshList() {
                this.loadMoreDisabled = true;

                // Don't query the same data more than once
                if (JSON.stringify(this.lastFilter) === JSON.stringify(this.filter)) return;

                Object.assign(this.lastFilter, this.filter);

                this.offset = 0;

                this.updateQuery();

                const params = {
                    count: this.pageSize,
                    offset: this.offset,
                };

                Object.assign(params, this.filter);

                Photo.search(params).then(response => {
                    this.results = response.models;

                    this.loadMoreDisabled = (response.models.length < this.pageSize);

                    if (this.loadMoreDisabled) {
                        this.$alert.info(this.results.length + ' photos found');
                    } else {
                        this.$alert.info('More than 50 photos found');
                    }
                });
            },
        },
        beforeRouteLeave(to, from, next) {
            next();
        },
        created() {
            this.refreshList();
        },
    };
</script>