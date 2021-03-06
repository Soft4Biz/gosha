
	<template>
    <WorkSpace>
        <template #header>
            <slot name="pageHeader">
                <VHead level="h1">EntityFilter</VHead>
            </slot>
        </template>

        <template #content>
            <slot name="data">
                <table>
                    <thead>
                        <tr>
                            <th v-for="(header, index) in fields" :key="index">{{ header }}</th>
                        </tr>
                    </thead>
            
                    <tbody>
                        <tr
                            v-for="entityFilterItem in entityFilterList"
                            :key="entityFilterItem.Id"
                            @click="selectEntityFilterItem(entityFilterItem)"
                            class="sw-table__row_can-select"
                            :class="{'sw-table__row_is-selected': entityFilterItem.Id === currentEntityFilterItem.item.Id}"
                        >
                            <td v-for="(value, key) in fields" :key="key + '-fields'">
                                <VCheckbox v-if="isCheckbox(entityFilterItem[key])" :checked="entityFilterItem[key]" disabled></VCheckbox>
                                <VText v-else>{{ entityFilterItem[key] }}</VText>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </slot>
            
            <slot name="panel">
                <VPanel
                    v-if="panel.show"
                    width="col3"
                    @close="closePanel"
                >
                    <template #header>
                        <VHead level="h3">{{ panelHeader }}</VHead>
                    </template>
        
                    <template #content>
                        <form @submit.prevent="saveChangesSubmit">
                            <VSet direction="vertical">
                                <VSet
                                    v-for="(filed, key) in editFields" :key="key + '-editFields'"
                                    vertical-align="center"
                                >
                                    <VLabel
                                        width="col4"
                                        :for="`currentEntityFilterItem${key}`"
                                    >{{ filed }}</VLabel>
                                    <VInput
										v-if="isInput(currentEntityFilterItem.item[key])"
                                        v-model="currentEntityFilterItem.item[key]"
                                        width="dyn"
                                        :id="`currentEntityFilterItem${key}`"
                                        @input="changeCurrentEntityFilterItem"
                                    />
									<VCheckbox
										v-if="isCheckbox(currentEntityFilterItem.item[key])"
                                        v-model="currentEntityFilterItem.item[key]"
                                        :id="`currentEntityFilterItem${key}`"
										@input="changeCurrentEntityFilterItem"
									/>
									
                                </VSet>
                            </VSet>
                            <button type="submit" :disabled="!currentEntityFilterItem.hasChange" hidden></button>
                        </form>
                    </template>

                    <template #footer>
                        <VSet>
                            <VButton
                                @click="saveChangesSubmit"
                                accent
                                :text="panelSubmitButtonText"
                                :disabled="!currentEntityFilterItem.hasChange"
                            />
                            <VButton
                                @click="cancelChanges"
                                text="Отменить"
                            />
                        </VSet>
                    </template>
                </VPanel>
            </slot>

            <slot name="confirmationPanel">
                <VPanel
                    v-if="currentEntityFilterItem.showDeleteConfirmation"
                    modal
                    @close="closeConfirmationPanel"
                >
                    <template #content>
                        <VHead level="h3">Удалить элемент?</VHead>
                    </template>

                    <template #footer>
                        <VSet>
                            <VButton
                                text="Подтвердить"
                                accent
                                @click="confirmDeleteHandler"
                            />
                            <VButton
                                text="Отмена"
                                @click="closeConfirmationPanel"
                            />
                        </VSet>
                    </template>
                </VPanel>
            </slot>
        </template>

        <template #footer>
            <slot name="pageFooter">
                <VSet>
                    <VButton
                        v-if="canCreate"
                        text="Добавить"
                        accent
                        @click="showPanel(panel.create)"
                    />
                    <VButton
                        v-if="canDelete"
                        text="Удалить"
                        :disabled="!currentEntityFilterItem.isSelected"
                        @click="deleteEntityFilterItemHandler"
                    />
                </VSet>
            </slot>
        </template>
    </WorkSpace>
</template>

<script>
    import entityFilterData from "../data/EntityFilterData";
    import { EntityFilter } from '../apiModel';
    import { mapGetters, mapMutations, mapActions } from 'vuex';
    import WorkSpace from "swui/src/components/WorkSpace";
    import VHead from "swui/src/components/VHead";
    import VSet from "swui/src/components/VSet";
    import VLabel from "swui/src/components/VLabel";
    import VInput from "swui/src/components/VInput";
    import VCheckbox from "swui/src/components/VCheckbox";
    import VText from "swui/src/components/VText";
    import VPanel from "swui/src/components/VPanel";
    import VButton from "swui/src/components/VButton";

    export default {
        name: 'EntityFilterGen',

        components: {VButton, VPanel, VText, VInput, VLabel, VSet, VHead, WorkSpace, VCheckbox},

        props: {
            fields: {
                type: Object,
                default() {
                    const entityFilterItem = new EntityFilter();
                    const fieldsObj = {};

                    for (let prop in entityFilterItem) {

                        if (entityFilterItem.hasOwnProperty(prop)) {
                            fieldsObj[prop] = prop;
                        }

                    }

                    return fieldsObj;
                }
            },
            editFields: {
                type: Object,
                default() {
                    const entityFilterItem = new EntityFilter();
                    const fieldsObj = {};

                    for (let prop in entityFilterItem) {

                        if (entityFilterItem.hasOwnProperty(prop)) {
                            fieldsObj[prop] = prop;
                        }

                    }

                    return fieldsObj;
                }
            },
            canDelete: {
                type: Boolean,
                default: true,
            },
            canCreate: {
                type: Boolean,
                default: true,
            },
        },

        data() {
            return entityFilterData;
        },

        created() {
			this.onCreated();
        },

        computed: {
            ...mapGetters({
                entityFilterList: 'getListEntityFilter'
            }),
            isPanelCreate() {
                return this.panel.type === this.panel.create;
            },
            isPanelEdit() {
                return this.panel.type === this.panel.edit;
            },
            panelHeader() {
                if (this.isPanelCreate) {
                    return this.panelHeaderCreate;
                }

                if (this.isPanelEdit) {
                    return this.panelHeaderEdit;
                }

                return  '';
            },
            panelSubmitButtonText() {
                if (this.isPanelCreate) {
                    return this.panelSubmitButtonTextCreate;
                }

                if (this.isPanelEdit) {
                    return this.panelSubmitButtonTextEdit;
                }

                return  '';
            },
            isCheckbox() {
                return data => {
                    return typeof data === "boolean";
                }
            },
            isInput() {
                return data => {
                    return ! this.isCheckbox(data);
                }
            },
        },

        methods: {
            ...mapActions([
                'findEntityFilter',
                'updateEntityFilter',
                'deleteEntityFilter',
                'createEntityFilter',
            ]),

            ...mapMutations([
                'addEntityFilterItemToList',
                'deleteEntityFilterFromList',
                'updateEntityFilterById',
            ]),

			onCreated() {
				this.fillEntityFilterFilter();
	            this.fetchEntityFilterData();
			},

            fillEntityFilterFilter() {
                this.entityFilterFilter.CurrentPage = 1;
                this.entityFilterFilter.PerPage = 1000;
            },

            fetchEntityFilterData() {
                return this.findEntityFilter({
                    filter: this.entityFilterFilter
                });
            },

            /**
             *
             * @param type
             */
            showPanel(type) {
                if (type === this.panel.create) {
                    this.panel.type = this.panel.create;
                    this.clearPanelEntityFilterItem();
                } else if (type === this.panel.edit) {
                    this.panel.type = this.panel.edit;
                    this.currentEntityFilterItem.isSelected = true;
                }

                this.panel.show = true;
            },

            closePanel() {
                this.panel.show = false;
                this.currentEntityFilterItem.isSelected = false;
                this.clearPanelEntityFilterItem();
            },

            selectEntityFilterItem(entityFilterItem) {
                this.showPanel(this.panel.edit);
                this.currentEntityFilterItem.isSelected = true;
                Object.assign(this.currentEntityFilterItem.item, entityFilterItem);
            },

            changeCurrentEntityFilterItem() {
                this.currentEntityFilterItem.hasChange = true;
            },

            cancelChanges() {
                this.clearPanelEntityFilterItem();
                this.closePanel();
            },

            clearPanelEntityFilterItem() {
                this.currentEntityFilterItem.item = new EntityFilter();
                this.currentEntityFilterItem.hasChange = false;
            },

            saveChangesSubmit() {
                if (this.isPanelCreate) {
                    this.createEntityFilterItemSubmit();
                    return;
                }

                if (this.isPanelEdit) {
                    this.editEntityFilterItemSubmit();
                }
            },

            createEntityFilterItemSubmit() {
                return this.createEntityFilter({
					data: this.currentEntityFilterItem.item,
                }).then((response) => {

                    if (response.Model) {
                        this.addEntityFilterItemToList(response.Model);
                        this.clearPanelEntityFilterItem();
                    } else {
                        console.error('Ошибка создания записи: ', response.Error);
                    }

                }).catch(error => {
                    console.error('Ошибка создания записи: ', error);
                });
            },

            editEntityFilterItemSubmit() {

                if (this.currentEntityFilterItem.hasChange) {
                    return this.updateEntityFilter({
                        id: this.currentEntityFilterItem.item.Id,
                        data: this.currentEntityFilterItem.item,
                    }).then((response) => {

                        if (response.Model) {
                            this.updateEntityFilterById(response.Model);
                            this.currentEntityFilterItem.hasChange = false;
                            this.clearPanelEntityFilterItem();
                            this.closePanel();
                        } else {
                            console.error('Ошибка изменения записи: ', response.Error);
                        }

                    }).catch(error => {
                        console.error('Ошибка изменения записи: ', error);
                    });

                } else {
					return new Promise(function(resolve, reject) {reject("Item has no changes. Nothing to save");})
				}
            },

            deleteEntityFilterItemHandler() {
                let deletedItemId = this.currentEntityFilterItem.item.Id;

                if (!this.currentEntityFilterItem.canDelete) {
                    this.currentEntityFilterItem.showDeleteConfirmation = true;
                    return;
                }

                this.deleteEntityFilter({
                    id: deletedItemId
                }).then(response => {

                    if (response.IsSuccess) {
                        this.deleteEntityFilterFromList(deletedItemId);
                        this.clearPanelEntityFilterItem();
                        this.currentEntityFilterItem.canDelete = false;
                        this.currentEntityFilterItem.isSelected = false;
                        this.panel.show = false;
                    } else {
                        console.error('Ошибка удаления элемента: ', response.Error);
                    }

                }).catch(error => {
                    console.error('Ошибка удаления элемента: ', error);
                });
            },

            confirmDeleteHandler() {
                this.currentEntityFilterItem.showDeleteConfirmation = false;
                this.currentEntityFilterItem.canDelete = true;
                this.deleteEntityFilterItemHandler();
            },

            closeConfirmationPanel() {
                this.currentEntityFilterItem.showDeleteConfirmation = false;
            },
        },
    }
</script>

<style lang="scss">

</style>
