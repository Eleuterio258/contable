<template>
    <layout>
      <template v-slot:header>
        <simple-header title="Categorías" button-text="Crear categoría" button-link='/categories/create' />
      </template>
      <template v-slot:content>
        <simple-table :headings="headings" :data="categories" :loading="loading" @editRecord="editRecord" @deleteRecord="deleteRecord"></simple-table>
      </template>
    </layout>
</template>

<script>
    import CATEGORIES from '../../graphql/categories/categories.graphql';
    import DELETE_CATEGORY from '../../graphql/categories/delete-category.graphql';
    import Swal from 'sweetalert2/dist/sweetalert2.js'
    import 'sweetalert2/src/sweetalert2.scss'

    export default {
        data() {
            return {
                headings: [
                    'Nombre'
                ],
                categories: [],
                loading: true
            }
        },
        mounted() {
            this.getCategories();
        },
        methods: {
            async getCategories() {
                const response = await this.$apollo.query({
                    query: CATEGORIES
                });
                this.categories = response.data.categories.map(item => {
                    return {
                        id: item.id,
                        name: item.name
                    };
                });
                this.loading = this.$apollo.loading;
            },
            editRecord(record) {
                this.$router.push(`/categories/${record.id}/edit`);
            },
            deleteRecord(record) {
                Swal.fire({
                    title: 'Estas seguro?',
                    text: `Quieres eliminar la categoría ${record.name}?, esto no es reversible.`,
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Si, eliminar!'
                }).then((result) => {
                    if (result.value) {
                        this.loading = true;
                        return this.$apollo.mutate({
                            mutation: DELETE_CATEGORY,
                            variables: {
                                id: record.id
                            }
                        });
                    }
                }).then(response => {
                    this.$toasted.success('Categoría eliminada satisfactoriamente!', {
                        position: "top-center",
                        duration : 5000
                    });
                    this.getCategories();
                })
            }
        }
    }
</script>

<style scoped>

</style>
