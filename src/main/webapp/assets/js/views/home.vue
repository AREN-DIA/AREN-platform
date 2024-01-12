<template>
    <div class="scroll-content" id="home">
        <div id="welcomeCallout" class="row light-color valign-wrapper">
            <img class="col" src="assets/img/imageleves.png">
                <p class="col" v-html="$t('helper.welcome')"></p>
        </div>
        
        <div class="container">
            <div style="display: flex; align-items: center;">
                <h1>Catégories</h1>
                <div class="toolbar" style="margin-left: auto; display: flex; align-items: center;">
                    <div style="margin-left: 10px;">
                        <input type="text" v-model="searchQuery" placeholder="Filtrer par nom de catégorie" style="padding: 5px;">
                    </div>
                    <div style="margin-right: 10px; display: flex; align-items: center;">
                        <button :style="{ 'color': sortBy === 'lastCommentDate' ? '#333' : '#A1a5a6', 'border-radius': '5px', 'padding': '0', 'cursor': 'pointer', 'border': 'none' }" @click="setSortBy('lastCommentDate')">
                            <i class="material-icons" style="font-size: 18px;">history</i>
                        </button>
                        <button :style="{ 'color': sortBy === 'debatesCount' ? '#333' : '#A1a5a6', 'border-radius': '5px', 'padding': '0', 'cursor': 'pointer', 'border': 'none' }" @click="setSortBy('debatesCount')">
                            <i class="material-icons" style="font-size: 18px;">filter_9_plus</i>
                        </button>
                    </div>
                    <div style="display: flex; align-items: center;">
                        <button :style="{ 'color': sortOrder === 'asc' ? '#333' : '#A1a5a6', 'border-radius': '5px', 'padding': '0', 'cursor': 'pointer', 'border': 'none' }" @click="setSortOrder('asc')">
                            <i class="material-icons" style="font-size: 18px;">arrow_upward</i>
                        </button>
                        <button :style="{ 'color': sortOrder === 'desc' ? '#333' : '#A1a5a6', 'border-radius': '5px', 'padding': '0', 'cursor': 'pointer', 'border': 'none' }" @click="setSortOrder('desc')">
                            <i class="material-icons" style="font-size: 18px;">arrow_downward</i>
                        </button>
                    </div>
                </div>
            </div>



            <div class="row center-align" id="categoriesList">
                <div v-for="category in filtredCategories" v-bind:key="category.id" class="col">
                    <router-link v-bind:to="'/debates?category='+category.id" class="card hoverable center-align">
                        <div class="card-image light-color valign-wrapper">
                            <img class="category-picture" v-bind:src="category.picture">
                        </div>
                        <div>
                            <h2 v-bind:title="category.name">{{ category.name }}</h2>
                            <p>{{ $tc("nb_debates", category.debatesCount) }}</p>
                        </div>
                        <div class="card-footer light-color">
                            <span>{{ typeof category.lastCommentDate !== 'undefined' ? $t("last_post_the") + " " + $d(category.lastCommentDate, 'short') : $t('no_post') }}</span>
                        </div>
                    </router-link>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    module.exports = {
        data( ) {
            return {
                categories: [],
                sortBy: 'lastCommentDate', // Par défaut, tri par date de modification
                sortOrder: 'desc', // Par défaut, tri décroissant
                searchQuery: '', // Terme de recherche 
                filtredCategories: []
            };
        },
        created( ) {
            this.fetchData( );
        },
        methods: {
            sortByDate(order) {
            this.sortBy = 'lastCommentDate';
            this.sortOrder = order;
            this.sortCategories();
            
        },
            sortByDebates(order) {
                this.sortBy = 'debatesCount';
                this.sortOrder = order;
                this.sortCategories();
            },
            setSortBy(sortBy) {
            this.sortBy = sortBy;
            this.sortCategories();
        },
            setSortOrder(order) {
                this.sortOrder = order;
                this.sortCategories();
            },
            sortCategories() {
                this.categories.sort((a, b) => {
                const aValue = this.sortBy === 'lastCommentDate' ? a.lastCommentDate : a.debatesCount;
                const bValue = this.sortBy === 'lastCommentDate' ? b.lastCommentDate : b.debatesCount;

                if (this.sortOrder === 'asc') {
                    return aValue - bValue;
                } else {
                    return bValue - aValue;
                }
                });
                this.filtredCategories = this.categories;
                // Filtrer en fonction du terme de recherche
                if (this.searchQuery === '') {
                    this.filtredCategories = this.categories;
                } else {
                    this.filtredCategories = this.categories.filter(category =>
                    category.name.toLowerCase().includes(this.searchQuery.toLowerCase())
                    );
                }
                    
                
            },
            fetchData( ) {
                ArenService.Categories.getAll({
                    query: {overview: true},
                    onSuccess: (categories) => { 
                        this.categories = categories; 
                        this.sortCategories();
                    },
                });
                
                    
            }
        },
        watch: {
        searchQuery() {    
            this.sortCategories();
        }
        },
    };
</script>