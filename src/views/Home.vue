<template>
<div class="home">
    <!-- Apollo watched Graphql query -->
    <ApolloQuery :query="require('../graphql/Genres.gql')">
        <template slot-scope="{ result: { loading, error, data }, isLoading }" @done="console.log(data)">
            <!-- Loading -->
            <v-progress-circular v-if="isLoading" indeterminate color="primary"></v-progress-circular>

            <!-- Error -->
            <div v-if="error" class="error apollo">An error occured</div>

            <!-- Result -->
            <v-container>
                <v-row>
                    <v-col>
                        <template v-if="data">
                            <h2 color="primary">Genres</h2>

                            <v-select :items="createItems(data.genres)" v-model="selectedGenre" label="choose a genre"></v-select>
                        </template>
                    </v-col>
                </v-row>
            </v-container>

            <!-- No result -->
        </template>
    </ApolloQuery>
    <h2>Name:</h2>
    <v-text-field label="name" v-model="selectedName"></v-text-field>
    <v-sheet>
        <v-container>
            <v-row class="d-flex flex-row">
                <ApolloQuery :query="require('../graphql/MangasByGenre.gql')" :variables="{ genre,name, page }">
                    <template slot-scope="{
                result: { loading, error, data, paginatorInfo },
                isLoading
              }">
                        <v-col v-if="isLoading" class="text-center">
                            <v-progress-circular indeterminate color="primary"></v-progress-circular>
                        </v-col>
                        <v-col v-else-if="data == null || error">No DATA</v-col>
                        <template v-else>
                            <v-pagination v-model="page" :length="setPage(data.mangasByGenre.paginatorInfo)" :total-visible="7"></v-pagination>

                            <v-row>
                                <v-col cols="2" class="pa-3 d-flex flex-column" v-for="(manga, index) in data.mangasByGenre.data" :key="index">
                                    <v-tooltip top color="primary darken-2">
                                        <template v-slot:activator="{ on, attrs }">
                                            <v-card v-bind="attrs" v-on="on" hover :to="{name:'manga',params:{id:manga.id}}" outlined color="primary lighten-4" class="elevation-5 mx-auto flex d-flex flex-column" max-height="100%" width="100%" max-width="100%">
                                                <v-img max-width="100%" max-height="400" height="400" :src="'data:image/png;base64, ' + manga.imageInfo"></v-img>

                                                <v-card-title class="d-inline-block text-truncate">{{ formatTitle(manga.name) }}</v-card-title>

                                                <v-card-text>
                                                    <v-expansion-panels focusable>
                                                        <v-expansion-panel v-on:click.prevent>
                                                            <v-expansion-panel-header color="primary lighten-4">Synopsis</v-expansion-panel-header>
                                                            <v-expansion-panel-content color="primary lighten-4" class="pt-5">{{ manga.synopsis }}</v-expansion-panel-content>
                                                        </v-expansion-panel>
                                                    </v-expansion-panels>
                                                </v-card-text>
                                            </v-card>
                                        </template>
                                        {{formatTitle(manga.name)}}
                                    </v-tooltip>
                                </v-col>
                            </v-row>
                            <v-pagination v-model="page" :length="setPage(data.mangasByGenre.paginatorInfo)" :total-visible="7"></v-pagination>
                        </template>
                    </template>
                </ApolloQuery>
            </v-row>
        </v-container>
    </v-sheet>
</div>
</template>

<script>
// @ is an alias to /src
export default {
    name: "Home",
    data() {
        return {
            selectedName: "",
            data: [],
            page: 1,
            mangas: null,
            selectedGenre: "Action",
            pages: 1
        };
    },
    computed: {
        genre() {
            return "%" + this.selectedGenre + "%";
        },
        name() {
            return (
                "%" +
                this.selectedName
                .split(" ")
                .map(value => value.charAt(0).toLowerCase() + value.substr(1))
                .join("-") +
                "%"
            );
        }
    },
    methods: {
        createItems(genres) {
            const array = genres.reduce((acc, genre) => {
                const genreFormatted = this.formatTitle(genre.genre);
                acc.push(genreFormatted);
                return acc;
            }, []);
            array.push("all");
            return array;
        },
        formatTitle(title) {
            return title
                .split("-")
                .map(value => value.charAt(0).toUpperCase() + value.substr(1))
                .join(" ");
        },
        setPage(paginatorInfo) {
            this.pages = paginatorInfo.lastPage;
            return this.pages;
        },
        spitData(data) {
            console.log(data.mangasByGenre.data);
        }
    }
};
</script>

<style scoped></style>
