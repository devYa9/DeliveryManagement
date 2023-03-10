<template>
    <div>
        <!-- Datatable -->
        <div class="tw-w-full tw-h-[475px]" :class="[{'tw-bg-gray-900': dark}]">
            <v-grid
                :can-focus="false"
                :theme="dark ? 'darkMaterial' : 'material'"
                :source="rows"
                :columns="columns"
                :filter="filterConfig"
                :resize="true"
                :range="3"
                :sortable="false"
                @beforefiltertrimmed="aftertrimmed"
                @beforesorting="beforesorting"
                frame-size="20"
                class="tw-border"
                :auto-size-column="true"
            >
            </v-grid>
        </div>
        <!-- /Datatable -->

        <!-- Pagination -->
        <div class="mt-5 tw-flex tw-justify-between">
            <div class="d-flex align-center">
                <div class="text-body-2 tw-h-fit mr-2 tw-text-zinc-700">Show per page: </div>
                <v-select :hide-details="true" v-model="paginationLimit" :items="allowedLimit" variant="outlined" density="compact" color="primary-color"></v-select>
            </div>
            <div class="d-flex align-center">
                <div class="text-caption tw-h-fit mr-2 font-weight-bold tw-text-zinc-700">{{ prevRange + 1 }} - {{ (currentPage == pageCount ? isFiltered ? filtredItems.length : rows.length : nextRange) }} of {{ isFiltered ? filtredItems.length : rows.length }} items </div>
                <div>
                <v-btn @click="currentPage = n" :ripple="false" variant="flat" class="mr-1" icon rounded="lg" :color="n == currentPage ? 'primary-color' : 'grey'" density="comfortable"  v-for="n in pageCount" :key="n">
                    <span class="tw-text-white">{{ n }}</span>
                </v-btn>
                </div>
            </div>
        </div>
        <!-- /Pagination -->
    </div>
</template>

<script>
import VGrid from '@revolist/vue3-datagrid'
import filterConfig from '@/config/dataTableFilters'



export default {
    components: { VGrid },
    props: {
        rows: {
            required: true
        },
        columns: {
            required: true
        },
        filterConfig: {
            required: false,
            default: filterConfig
        }
    },

    data() {
        return {
            dark: false,
            allowedLimit: [5, 10, 20, 50, 100],
            isFiltered: false,
            currentPage: 1,
            paginationLimit: 10,
            filtredOut: {}
        }
    },

    computed: {
        pageCount() {
            return this.isFiltered ? this.pageCountWithFilter : this.pageCountNoFilter
        },

        pageCountWithFilter() {
            return Math.ceil(this.filtredItems.length / this.paginationLimit)
        },

        pageCountNoFilter() {
            return Math.ceil(this.rows.length / this.paginationLimit)
        },

        prevRange() {
            return (this.currentPage - 1) * this.paginationLimit
        },

        nextRange() {
            return (this.currentPage) * this.paginationLimit
        },

        // returns array with result from the filter
        filtredItems() {
            const indexes = Object.keys(this.filtredOut);
            return this.rows.filter((e, i) => {
                return !indexes.includes(`${i}`);
            })
        },


        // returns object Items to be filtred out by index if filter is not active
        trimmedNoFilter() {
            const trimmed = {}

            this.rows.forEach((e, i) => {
                if(i < this.prevRange || i > this.nextRange - 1) {
                    trimmed[i] = true
                }
            })

            return trimmed
        },

        // returns object Items to be filtred out by index if filter is active
        trimmedWithFilter() {
            let toBeTrimmed = {}

            this.rows.forEach((item, index) =>{
                toBeTrimmed[index] = true
            });

            const currentVisibleItemsKeys = this.visibleItemsIndexes.slice(this.prevRange, this.nextRange);

            currentVisibleItemsKeys.forEach(item => {
                toBeTrimmed[item] = false
            }) 

            return toBeTrimmed
        },


        visibleItemsIndexes() {
            
            const visibleItemsIndexes = [];
            this.filtredItems.forEach((item) => {
                visibleItemsIndexes.push(this.rows.indexOf(item));
            })
            return visibleItemsIndexes
        }
    },

    watch: {
        currentPage() {
            this.updateDatatable()
        },
        filtredOut() {
            this.currentPage = 1;
            this.updateDatatable()
        },
        paginationLimit() {
            this.currentPage = 1;
            this.updateDatatable()
        }
    },

    methods: {
        aftertrimmed(source) {
            const filtersCount = Object.keys(source.detail.collection).length
            if (filtersCount > 0) {
                this.filtredOut = source.detail.itemsToFilter
                this.isFiltered =  true;
                
            } else {
                this.filtredOut = {}
                this.isFiltered = false
            }
            // console.log(source);
        },
        clearFilters() {
            document.querySelector('revo-grid')
            .addTrimmed({})
        },

        beforesorting(source) {
            console.log(source);
        },

        updateDatatable() {
            document.querySelector('revo-grid')
            .addTrimmed(this.isFiltered ? this.trimmedWithFilter : this.trimmedNoFilter)
        }
    },

    mounted() {
            this.updateDatatable()
        

    }
}
</script>

<style>

</style>