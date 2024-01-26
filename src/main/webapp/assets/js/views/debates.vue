<template>
    <base-layout id="debates">
        <template v-slot:title>
            <h1>{{ $root.user.is('USER') &&  categories.length > 1 ? $t('menu.my_debates') : categories.length === 1 ? "Débats de la catégorie : "+categories[0].name : $t('menu.public_debates') }}</h1>
        </template>

        <template v-slot:right>
            <router-link v-if="$root.user.is('MODO')"
                         class="waves-effect waves-light btn"
                         to="/createDebate">
                {{ $t('new_debate') }}
            </router-link>
        </template>
        <div v-if="categories.length > 1" style="display: flex; align-items: center; justify-content: space-between;">
            <div class="toolbar" style="display: flex; align-items: center;justify-content: center;">
                <input type="text" v-model="searchQuery" placeholder="Recherche par catégorie">
            </div>
            <div class="toolbar" style="display: flex; align-items: center;">
                <span :style="{ 'color': '#A1a5a6' }">Filtres : </span>
                <div style="display: flex; align-items: center; margin-left: 10px; margin-right: 10px;">
                <button :style="{ 'color': sortBy === 'lastCommentDate' ? '#333' : '#A1a5a6', 'border-radius': '5px', 'padding': '0', 'cursor': 'pointer', 'border': 'none' }" @click="setSortBy('lastCommentDate')" title="Trier par date du dernier commentaire">
                    <i class="material-icons" style="font-size: 18px;">history</i>
                </button>
                <button :style="{ 'color': sortBy === 'debatesCount' ? '#333' : '#A1a5a6', 'border-radius': '5px', 'padding': '0', 'cursor': 'pointer', 'border': 'none' }" @click="setSortBy('debatesCount')" title="Trier par nombre de débats">
                    <i class="material-icons" style="font-size: 18px;">filter_9_plus</i>
                </button>
                </div>

                <span :style="{ 'color': '#A1a5a6' }">Ordre : </span>
                <div style="display: flex; align-items: center;">
                <button :style="{ 'color': sortOrder === 'asc' ? '#333' : '#A1a5a6', 'border-radius': '5px', 'padding': '0', 'cursor': 'pointer', 'border': 'none' }" @click="setSortOrder('asc')" title="Ordre croissant">
                    <i class="material-icons" style="font-size: 18px;">arrow_upward</i>
                </button>
                <button :style="{ 'color': sortOrder === 'desc' ? '#333' : '#A1a5a6', 'border-radius': '5px', 'padding': '0', 'cursor': 'pointer', 'border': 'none' }" @click="setSortOrder('desc')" title="Ordre décroissant">
                    <i class="material-icons" style="font-size: 18px;">arrow_downward</i>
                </button>
                </div>
            </div>
        </div>
        <debates-grid
            v-bind:categories="filtredCategories"
            v-bind:extendable="true">

            <template v-if="$root.user.is('MODO')" v-slot:side.actions="{ debate }">
                <action-button
                    v-bind:tooltip="$t('helper.remove_debate')"
                    v-bind:tooltip-disabled="$t('helper.cannot_remove_debate')"
                    v-bind:disabled="canRemove(debate)"
                    @press="deleteDebate(debate)"
                    icon="delete">
                </action-button>
            </template>
            <template v-if="$root.user.is('MODO')" v-slot:add.contributor="{ debate }">
                <span class="waves-effect waves-light btn"
                      @click="edit( debate )">
                    {{ $t('invite_in_debate') }}
                </span>
            </template>

        </debates-grid>

        <template v-slot:addons v-if="$root.user.is('MODO')">
            <contributor-modal
                ref="contributorModal">
            </contributor-modal>
        </template>
    </base-layout>
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
                this.categories = [];
                let categoryId = this.$route.query.category;
                ArenService.Debates.getAll({
                    query: categoryId ? {category: categoryId, overview: true} : {overview: true},
                    onSuccess: debates => debates.forEach(d => {
                            if (!this.categories.includes(d.document.category)) {
                                this.categories.push(d.document.category);
                                this.sortCategories();
                            }
                        })
                });
            },
            edit(debate) {
                this.$refs.contributorModal.debate = debate;
                this.$refs.contributorModal.open( );
            },
            deleteDebate(debate) {
                this.$confirm({
                    title: this.$t("helper.delete_debate", {debateName: debate.document.name}),
                    message: this.$t('helper.delete_debate_warning') + '<br><br>' + this.$t('helper.not_cancelable') + '<br>' + this.$t('helper.do_continue'),
                    callback: (returnValue) => {
                        if (returnValue) {
                            ArenService.Debates.remove({
                                data: debate
                            });
                        }
                    }
                });
            },
            canRemove(debate) {
              return !ArenService.Configs['rules.remove.debatesWithComments'] && debate.commentsCount !== 0;
            }
        },
        watch: {
        searchQuery() {    
            this.sortCategories();
        }
        },
        components: {
            'contributor-modal': vueLoader('components/modals/contributorModal'),
            'debates-grid': vueLoader('components/grids/debatesGrid')
        }
    };
</script>