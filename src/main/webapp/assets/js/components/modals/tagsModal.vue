<template>
    <modal-layout>

        <template v-slot:title>
            {{ $t('tags_edition') }}
        </template>
        
        <!-- <div class="selection">
            <div class="reformulation-wrapper">
                <i class="material-icons">text_snippet</i>
                <div v-html="comment.selection" ref="selection"></div>
            </div>
            <hr>
            <div class="reformulation-wrapper">
                <i class="material-icons">hearing</i>
                <div v-html="comment.reformulation" ref="reformulation"></div>
            </div>
            <hr>
            <div class="argumentation-wrapper">
                <i class="material-icons">chat_bubble_outline</i>
                <div v-html="comment.argumentation" ref="argumentation"></div>
            </div>
        </div> -->
        <details :class="{ 'selected-collapsible': isCollapsibleSelected('sel'), 'open': collapsibles.sel.isOpen }" @click="toggleCollapsible('sel')">
        <summary><i class="material-icons small-icon">text_snippet</i> Selection </summary>
        <div class="selection">
            <div class="selection-tag-wrapper">
                
                <div v-html="comment.selection" ref="selection"></div>
            </div>
        </div>
        <hr>
        <div v-for="(tag, index) in sortedSelectionTags"
                v-bind:class="{ chip: true, negative: (comment.proposedTags.includes(tag) && tag.negative), positive: (comment.proposedTags.includes(tag) && !tag.negative) }"
                @click="toggleTag(tag)"
                @contextmenu.prevent.stop="tag.negative = !tag.negative">
                {{ getFormattedValue(tag) }} 
        </div>
        <div v-for="(tag, index) in filteredSelectionProposedTags"
             class="chip valid">
            {{ getFormattedValue(tag) }}
            <i class="close material-icons" @click.stop="comment.proposedTags.remove(tag)">close</i>
        </div>
        </details>
        <text-input v-if="isCollapsibleSelected('sel')"
            v-bind:helper="$t('helper.tag_input_explanations')"
            v-bind:label="$t('proposed_tags')"
            v-model="tagInput"
            @keyup.enter="addTag()">
        </text-input>
        
        <hr class="custom-hr">
        <details :class="{ 'selected-collapsible': isCollapsibleSelected('refo'), 'open': collapsibles.refo.isOpen }" @click="toggleCollapsible('refo')">
        <summary><i class="material-icons small-icon">hearing</i> {{ $t('comment.reformulation') }}</summary>
        <div class="selection">
            <div class="selection-tag-wrapper">
                
                <div v-html="comment.reformulation" ref="reformulation"></div>
            </div>
        </div>
        <hr>
        <div v-if ="sortedReformulationTags.length > 0" v-for="(tag, index) in sortedReformulationTags"
             v-bind:class="{ chip: true, negative: (comment.proposedTags.includes(tag) && tag.negative), positive: (comment.proposedTags.includes(tag) && !tag.negative) }"
             @click="toggleTag(tag)"
             @contextmenu.prevent.stop="tag.negative = !tag.negative">
            {{ getFormattedValue(tag) }}
        </div>
        <div v-for="(tag, index) in filteredReformulationProposedTags"
             class="chip valid">
            {{ getFormattedValue(tag) }}
            <i class="close material-icons" @click.stop="comment.proposedTags.remove(tag)">close</i>
        </div>
        </details>
        <text-input v-if="isCollapsibleSelected('refo')"
            v-bind:helper="$t('helper.tag_input_explanations')"
            v-bind:label="$t('proposed_tags')"
            v-model="tagInput"
            @keyup.enter="addTag()">
        </text-input>
        
        <hr class="custom-hr">
        <details :class="{ 'selected-collapsible': isCollapsibleSelected('arg'), 'open': collapsibles.arg.isOpen }" @click="toggleCollapsible('arg')">
        <summary><i class="material-icons small-icon">chat_bubble_outline</i> {{ $t('comment.argumentation') }}</summary>
        <div class="selection">
            <div class="selection-tag-wrapper">
                
                <div v-html="comment.argumentation" ref="argumentation"></div>
            </div>
        </div>
        
        <hr>
        <div class="tile-container">
                <div v-for="(tag, index) in sortedArgumentTags"
                v-bind:class="{ chip: true, negative: (comment.proposedTags.includes(tag) && tag.negative), positive: (comment.proposedTags.includes(tag) && !tag.negative) }"
                @click="toggleTag(tag)"
                @contextmenu.prevent.stop="tag.negative = !tag.negative">
                {{ getFormattedValue(tag) }}
                </div>
        </div>    
        <div v-for="(tag, index) in filteredArgumentProposedTags"
             class="chip valid">
            {{ getFormattedValue(tag) }}
            <i class="close material-icons" @click.stop="comment.proposedTags.remove(tag)">close</i>
        </div> 
        </details> 
        <text-input v-if="isCollapsibleSelected('arg')"
            v-bind:helper="$t('helper.tag_input_explanations')"
            v-bind:label="$t('proposed_tags')"
            v-model="tagInput"
            @keyup.enter="addTag()">
        </text-input> 
        
        


        <template v-slot:footer>
            <template v-if="readOnly">
                <button @click="close( );" class="waves-effect waves-green btn-flat">{{ $t('close') }}</button>
            </template>
            <template v-else>
                <button v-if="$root.user.is('ADMIN')" @click="update( );" class="waves-effect waves-green btn left">{{ $t('update_tags') }}</button>
                <button @click="close( );" class="waves-effect waves-green btn-flat">{{ $t('cancel') }}</button>
                <button @click="close( comment );" class="waves-effect waves-green btn-flat">{{ $t('validate') }}</button>
            </template>
        </template>

    </modal-layout>
</template>

<script>
    module.exports = {
        mixins: [VueModal],
        props: ['readOnly'],
        data() {
            return {
                tagInput: "",
                comment: false,
                selectedCollapsible: null,
                lastOpenedCollapsible: null, 
                openCollapsibles: [],
                collapsibles: {
                    sel : { isOpen: false },
                    refo : { isOpen: false },
                    arg : { isOpen: false },
                },
            }
        },
        computed: {
            isCollapsibleSelected() {
                return (collapsible) => this.selectedCollapsible === collapsible;
            },
            sortedTags() {
                return this.comment.tags ? this.comment.tags.sort((a, b) => b.power - a.power) : [];
            },
            sortedSelectionTags() {
                if (this.comment.tags){
                    const selectionTags = this.comment.tags.filter(tag => tag.value.startsWith('sel::') || !tag.value.includes("::"));
                
                    return selectionTags ? selectionTags.sort((a, b) => b.power - a.power) : [];
                }
            },
            sortedReformulationTags() {
                if (this.comment.tags){
                        const reformulationTags = this.comment.tags.filter(tag => tag.value.startsWith('refo::') || !tag.value.includes("::"));
                        return reformulationTags ? reformulationTags.sort((a, b) => b.power - a.power) : [];
                }
            },
            sortedArgumentTags() {
                if (this.comment.tags){
                        const argumentTags = this.comment.tags.filter(tag => tag.value.startsWith('arg::') || !tag.value.includes("::"));
                        return argumentTags ? argumentTags.sort((a, b) => b.power - a.power) : this.comment.tags.sort((a, b) => b.power - a.power);
                }
            },
            filteredProposedTags() {
                return this.comment.proposedTags ? this.comment.proposedTags.filter((t) => !this.comment.tags.includes(t)) : [];
            },
            filteredSelectionProposedTags() {
                if (this.comment.proposedTags){
                    const selectionTags = this.comment.proposedTags.filter(tag => !this.comment.tags.includes(tag) && tag.value.startsWith('sel::'));
                    return selectionTags ? selectionTags : [];
                }
            },
            filteredReformulationProposedTags() {
                if (this.comment.proposedTags){
                        const reformulationTags = this.comment.proposedTags.filter(tag => !this.comment.tags.includes(tag) && (tag.value.startsWith('refo::') || !tag.value.includes('::')));
                        return reformulationTags ? reformulationTags : [];
                }
            },
            filteredArgumentProposedTags() {
                if (this.comment.proposedTags){
                        const argumentTags = this.comment.proposedTags.filter(tag => !this.comment.tags.includes(tag) && tag.value.startsWith('arg::'));
                        return argumentTags ? argumentTags : [];
                }
            },
        },
        methods: {
            toggleCollapsible(collapsibleKey) {
                Object.keys(this.collapsibles).forEach((key) => {
                    if (key !== collapsibleKey) {
                        this.$set(this.collapsibles, key, { isOpen: false });
                    }
                });

            // Mettre à jour l'état d'ouverture du collapsible actuel
            this.$set(this.collapsibles, collapsibleKey, { isOpen: !this.collapsibles[collapsibleKey].isOpen });
           
            const index = this.openCollapsibles.indexOf(collapsibleKey);
            if (index === -1) {
                //this.lastOpenedCollapsible = this.selectedCollapsible;
                this.selectedCollapsible = collapsibleKey;
                this.openCollapsibles.push(collapsibleKey);
            } else {
                this.openCollapsibles.splice(collapsibleKey);
                const myOpen = this.openCollapsibles.pop();
                this.selectedCollapsible = myOpen;
                this.openCollapsibles.push(myOpen);
            }
            
        },
            toggleCollapsibles(collapsible) {
                if (collapsible !== this.selectedCollapsible){
                // Le collapsible actuel est sélectionné et ouvert
                this.selectedCollapsible = collapsible; // Définir le collapsible actuel comme le sélectionné
                this.lastOpenedCollapsible = collapsible; // Mettre à jour le dernier collapsible ouvert
                }
            },
            getFormattedValue(tag) {
                if (tag.value.includes("::")) {
                return tag.value.substring(tag.value.indexOf("::") + 2);
                } else { 
                return tag.value;
                }
            },
            toggleTag(tag) { 
                if (!this.comment.proposedTags.includes(tag)) {                    
                    tag.negative = false;
                    this.comment.proposedTags.push(tag);
                } else {
                    this.comment.proposedTags.remove(tag);
                }
            },
            addTag() {
                let tag = new Tag(this.tagInput);
                if (tag.value.length > 0) {
                    let lc = tag.value.toLowerCase();
                    let index = this.comment.proposedTags.findIndex((t) => t.value.toLowerCase() === lc);
                    let prefix = this.selectedCollapsible +"::";
                    tag.value = prefix + tag.value; // Ajouter le préfixe au tag

                    if (index !== -1) {
                        this.comment.proposedTags[index] = tag;
                    } else {
                        this.comment.proposedTags.push(tag);
                    }
                    this.tagInput = "";
                }
            },
            update() {
                this.$confirm({
                    title: this.$t('update_tags'),
                    message: this.$t('helper.update_tags'),
                    callback: (val) => {
                      if (val) {
                        ArenService.Comments.updateTags({
                            id: this.comment.id,
                        });
                        this.close();
                      }
                    }
                });
            },
            afterClose() {
                this.comment = false;
            }
        }
    };
</script>
