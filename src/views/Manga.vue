<template>
<v-container>
    <!--  <ApolloQuery :query="require('../graphql/MangaById.gql')" :variables="{ id }">
    <template slot-scope="{ result: { loading, error, data }, isLoading }">-->
    <v-row v-if="$apollo.queries.manga.loading">
        <v-col>
            <v-progress-circular indeterminate color="primary"></v-progress-circular>
        </v-col>
    </v-row>
    <v-row v-else-if="manga == null">No Data</v-row>
    <template v-else-if="$apollo.queries.manga">
        <h2 class="light-blue--text darken-1">{{ manga.name }}</h2>
        <v-row>
            <v-col>
                <v-rating :value="Number(manga.score)" @input="rate" hover background-color="orange lighten-3" color="orange" large></v-rating>
            </v-col>
        </v-row>
        <v-divider></v-divider>
        <v-row>
            <v-col cols="4" class="text-center">
                <v-img height="600" width="500" :src="'data:image/png;base64, ' + manga.imageInfo"></v-img>
            </v-col>
            <v-col cols="4">
                <div class="manga-info">
                    <p class="text-justify">
                        <span>
                            <b>title:</b>
                        </span>
                        {{ manga.name }}
                    </p>
                    <v-spacer></v-spacer>
                    <p class="text-justify">
                        <span>
                            <b>alternative title:</b>
                        </span>
                        {{ manga.alternativeTitle }}
                    </p>
                    <v-spacer></v-spacer>
                    <p class="text-justify">
                        <span>
                            <b>author:</b>
                        </span>
                        {{ manga.author }}
                    </p>
                    <v-spacer></v-spacer>
                    <p class="text-justify">
                        <span>
                            <b>status:</b>
                        </span>
                        {{ manga.status }}
                    </p>
                    <v-spacer></v-spacer>
                    <p class="text-justify">
                        <span>
                            <b>type:</b>
                        </span>
                        {{ manga.type }}
                    </p>
                    <v-spacer></v-spacer>
                    <p class="text-justify">
                        <span>
                            <b>score:</b>
                        </span>
                        {{ manga.score }}
                    </p>

                    <v-spacer></v-spacer>
                    <p class="text-justify">
                        <span>
                            <b>genre:</b>
                        </span>
                        {{ manga.genre }}
                    </p>
                    <v-spacer></v-spacer>
                </div>
            </v-col>
        </v-row>
        <v-divider class="mt-4 mb-4"></v-divider>

        <v-row v-for="version in Object.keys(tables)" :key="version">
            <v-col>
                <h3 class="light-blue--text darken-1">{{ version }}</h3>
                <v-data-table dense v-model="selected[version]" :headers="headers" :items="tables[version]" :single-select="singleSelect[version]" item-key="title" show-select class="elevation-1"></v-data-table>
            </v-col>
        </v-row>
        <v-divider class="mt-4 mb-4"></v-divider>
        <v-row>
            <v-col>
                <h4 color>comments</h4>
                <!-- <ApolloQuery :query="require('../graphql/CommentsByResource.gql')" :variables="{ resourceID: id, resourceType: resourceType }">
                    <template slot-scope="{
                    result: { loading, error, data, paginatorInfo },
                    isLoading
          }">-->
                <v-col v-if="$apollo.queries.commentsByResource.loading" class="text-center">
                    <v-progress-circular indeterminate color="primary"></v-progress-circular>
                </v-col>
                <v-col v-else-if="$apollo.queries.commentsByResource == null">No DATA</v-col>
                <template v-else>
                    <v-row v-for="(comment, index) in resourceComments" :key="index">
                        <v-col>
                            <div class="d-flex flex-direction-row">
                                <p class="mr-5">{{ comment.id }}</p>
                                <h2>
                                    <b>{{ comment.user.name }}</b>
                                </h2>
                                <v-btn text :color="comment.userLiked ? 'success' : 'primary'" @click="likeComment(comment.id)">
                                    <v-icon small>mdi-thumb-up</v-icon>
                                </v-btn>
                                <v-btn text :color="
                              comment.userDisliked ? 'red darken-1' : 'primary'
                            " @click="dislikeComment(comment.id)">
                                    <v-icon small>mdi-thumb-down</v-icon>
                                </v-btn>
                                <p>likes: {{ parseLikes(comment.likes) }}</p>
                            </div>
                            <div v-html="comment.comment"></div>
                            <v-btn class="m-4 primary--text" text small @click="responseComment(comment)">comment</v-btn>
                            <v-btn class="m-4 primary--text" v-if="(comment.user.id = 1)" text small @click="deleteComment(comment.id)">delete</v-btn>
                            <v-btn class="m-4 primary--text" v-if="(comment.user.id = 1)" text small @click="updateComment(comment)">update</v-btn>
                            <v-expansion-panels v-if="comment.comments.length !== 0">
                                <v-expansion-panel>
                                    <v-expansion-panel-header>show comments({{ comment.comments.length }})</v-expansion-panel-header>
                                    <v-divider></v-divider>

                                    <v-expansion-panel-content v-for="(responseComment,
                              index) in comment.comments" :key="index" class="mt-5">
                                        <div class="d-flex flex-direction-row">
                                            <p class="mr-5">{{ responseComment.id }}</p>
                                            <h2>
                                                <b>{{ responseComment.user.name }}</b>
                                            </h2>
                                            <v-btn text :color="
                                    responseComment.userLiked
                                      ? 'success'
                                      : 'primary'
                                  " @click="likeComment(responseComment.id)">
                                                <v-icon small>mdi-thumb-up</v-icon>
                                            </v-btn>
                                            <v-btn text :color="
                                    responseComment.userDisliked
                                      ? 'red darken-1'
                                      : 'primary'
                                  " @click="dislikeComment(responseComment.id)">
                                                <v-icon small>mdi-thumb-down</v-icon>
                                            </v-btn>
                                            <p>likes: {{ parseLikes(responseComment.likes) }}</p>
                                        </div>
                                        <div v-html="responseComment.comment"></div>
                                        <v-btn class="m-4 primary--text" v-if="(responseComment.user.id = 1)" text small @click="deleteComment(responseComment.id)">delete</v-btn>
                                        <v-btn class="m-4 primary--text" v-if="(responseComment.user.id = 1)" text small @click="updateComment(responseComment)">update</v-btn>
                                    </v-expansion-panel-content>
                                </v-expansion-panel>
                            </v-expansion-panels>
                        </v-col>
                    </v-row>
                    <v-divider></v-divider>
                    <v-row>
                        <v-col>
                            <div v-if="commentAction == 'update'">modifies --> {{ selectedComment.id }}</div>
                            <div v-if="commentAction == 'response'">response to --> {{ selectedComment.id }}</div>
                            <v-btn class="mt-5 mr-5 mb-5" color="primary" :disabled="comment == '' || comment == null" @click="handleCommentButton">comment</v-btn>
                            <v-btn v-if="commentAction !== 'create'" class="mt-5 mr-5 mb-5" color="primary" @click="cancelComment">cancel</v-btn>
                            <ckeditor :editor="editor.editor" v-model="comment" :config="editor.editorConfig"></ckeditor>
                        </v-col>
                    </v-row>
                </template>
                <!--  </template>
          </ApolloQuery>-->
            </v-col>
        </v-row>
    </template>
    <!-- </template>
    </ApolloQuery>-->
</v-container>
</template>

<script>
import ClassicEditor from "@ckeditor/ckeditor5-build-classic";
import CKEditor from "@ckeditor/ckeditor5-vue";
import gql from "graphql-tag";

export default {
    components: {
        ckeditor: CKEditor.component
    },

    data() {
        return {
            manga: "",
            rating: 0,
            commentAction: "create",
            selectedComment: null,
            resourceComments: null,
            comment: "",
            id: "",
            tables: null,
            singleSelect: {},
            selected: {},
            headers: [{
                text: "chapter",
                align: "start",
                sortable: true,
                value: "title"
            }],
            resourceType: "manga",
            editor: {
                editor: ClassicEditor,
                editorConfig: {
                    height: "250px",
                    toolbar: {
                        items: [
                            "heading",
                            "|",
                            "bold",
                            "italic",
                            "|",
                            "bulletedList",
                            "numberedList",
                            "|",
                            "link",
                            "|",
                            "insertTable",
                            "|",
                            "undo",
                            "redo"
                        ]
                    },
                    table: {
                        contentToolbar: ["tableColumn", "tableRow", "mergeTableCells"]
                    },
                    language: "es"
                }
            }
        };
    },
    apollo: {
        manga: {
            query: gql `
        query($id: ID) {
          manga(id: $id) {
            id
            name
            imageInfo
            genre
            author
            status
            chapters
            score
            artist
            alternativeTitle
            type
            synopsis
          }
        }
      `,
            variables() {
                // Use vue reactive properties here
                return {
                    id: this.id
                };
            },
            result() {
                console.log("finished");
                this.parseChapters(this.manga.chapters);
            }
        },
        commentsByResource: {
            query: gql `
        query($resourceID: ID!, $resourceType: String!) {
          commentsByResource(
            resourceID: $resourceID
            resourceType: $resourceType
          ) {
            comment
            id
            user {
              id
              name
            }
            likes {
              like
              user_id
            }
            comments {
              comment
              id
              user {
                id
                name
              }
              likes {
                like
                user_id
              }
            }
          }
        }
      `,
            variables() {
                return {
                    resourceID: this.id,
                    resourceType: "manga"
                };
            },
            result() {
                this.resourceComments = this.addUserLikesInfoToComments(
                    this.commentsByResource
                );
            }
        }
    },
    created() {
        this.id = this.$route.params.id;
        this.rating = Number(this.manga.score);
    },
    methods: {
        rate(value) {
            this.$apollo
                .mutate({
                    // Query
                    mutation: gql `
            mutation(
              $resourceID: ID!
              $resourceType: String!
              $userID: ID!
              $rating: Int!
            ) {
              rateResourceMutation(
                resourceID: $resourceID
                resourceType: $resourceType
                userID: $userID
                rating: $rating
              ) {
                id
                name
                imageInfo
                genre
                author
                status
                chapters
                score
                artist
                alternativeTitle
                type
                synopsis
              }
            }
          `,
                    // Parameters
                    variables: {
                        resourceID: this.id,
                        resourceType: this.resourceType,
                        userID: 1,
                        rating: Number(value)
                    }
                })
                .then(data => {
                    this.manga = data.data.rateResourceMutation;
                    this.parseChapters(this.manga.chapters);
                })
                .catch(error => {
                    // Error
                    console.error(error);
                });
        },
        parseLikes(likes) {
            return likes.reduce((acc, like) => {
                acc += like.like;
                return acc;
            }, 0);
        },
        likeComment(commentID) {
            this.$apollo
                .mutate({
                    // Query
                    mutation: gql `
            mutation(
              $resourceID: ID!
              $resourceType: String!
              $userID: ID!
              $commentID: ID!
            ) {
              likeCommentMutation(
                resourceID: $resourceID
                resourceType: $resourceType
                userID: $userID
                commentID: $commentID
              ) {
                comment
                id
                user {
                  name
                  id
                }
                likes {
                  like
                  user_id
                }
                comments {
                  comment
                  id
                  user {
                    name
                    id
                  }
                  likes {
                    like
                    user_id
                  }
                }
              }
            }
          `,
                    // Parameters
                    variables: {
                        resourceID: this.id,
                        resourceType: this.resourceType,
                        userID: 1,
                        commentID: commentID
                    }
                })
                .then(
                    data =>
                    (this.resourceComments = this.addUserLikesInfoToComments(
                        data.data.likeCommentMutation
                    ))
                )
                .catch(error => {
                    // Error
                    console.error(error);
                });
        },
        dislikeComment(commentID) {
            this.$apollo
                .mutate({
                    // Query
                    mutation: gql `
            mutation(
              $resourceID: ID!
              $resourceType: String!
              $userID: ID!
              $commentID: ID!
            ) {
              dislikeCommentMutation(
                resourceID: $resourceID
                resourceType: $resourceType
                userID: $userID
                commentID: $commentID
              ) {
                comment
                id
                user {
                  name
                  id
                }
                likes {
                  like
                  user_id
                }
                comments {
                  comment
                  id
                  user {
                    name
                    id
                  }
                  likes {
                    like
                    user_id
                  }
                }
              }
            }
          `,
                    // Parameters
                    variables: {
                        resourceID: this.id,
                        resourceType: this.resourceType,
                        userID: 1,
                        commentID: commentID
                    }
                })
                .then(
                    data =>
                    (this.resourceComments = this.addUserLikesInfoToComments(
                        data.data.dislikeCommentMutation
                    ))
                )
                .catch(error => {
                    // Error
                    console.error(error);
                });
        },
        addUserLikesInfoToComments(comments) {
            let commentsParsed = comments.map(comment => {
                const userLike = comment.likes.find(like => like.user_id == 1);
                if (userLike) {
                    comment.userLiked = userLike.like == 1;
                    comment.userDisliked = userLike.like == -1;
                }
                comment.comments.map(responseComment => {
                    const userLike = responseComment.likes.find(
                        like => like.user_id == 1
                    );
                    if (userLike) {
                        responseComment.userLiked = userLike.like == 1;
                        responseComment.userDisliked = userLike.like == -1;
                    }
                    return responseComment;
                });
                return comment;
            });
            return commentsParsed;
        },
        cancelComment() {
            this.comment = "";
            this.selectedComment = null;
            this.commentAction = "create";
        },
        updateComment(comment) {
            this.selectedComment = comment;
            this.commentAction = "update";
            this.comment = comment.comment;
        },
        responseComment(comment) {
            this.selectedComment = comment;
            this.commentAction = "response";
        },
        handleResponseComment() {
            this.$apollo
                .mutate({
                    // Query
                    mutation: gql `
            mutation(
              $resourceID: ID!
              $resourceType: String!
              $comment: String!
              $userID: ID!
              $responseCommentID: ID!
            ) {
              createCommentMutation(
                resourceID: $resourceID
                resourceType: $resourceType
                comment: $comment
                userID: $userID
                responseCommentID: $responseCommentID
              ) {
                comment
                id
                user {
                  name
                  id
                }
                comments {
                  comment
                  id
                  user {
                    name
                    id
                  }
                }
              }
            }
          `,
                    // Parameters
                    variables: {
                        resourceID: this.id,
                        resourceType: this.resourceType,
                        comment: this.comment,
                        userID: 1,
                        responseCommentID: this.selectedComment.id
                    }
                })
                .then(data => (this.resourceComments = data.data.createCommentMutation))
                .catch(error => {
                    // Error
                    console.error(error);
                });
        },
        handleUpdateComment() {
            this.$apollo
                .mutate({
                    // Query
                    mutation: gql `
            mutation(
              $commentID: ID!
              $userID: ID!
              $resourceID: ID!
              $resourceType: String!
              $comment: String!
            ) {
              updateCommentMutation(
                commentID: $commentID
                userID: $userID
                resourceType: $resourceType
                resourceID: $resourceID
                comment: $comment
              ) {
                comment
                id
                user {
                  name
                  id
                }
                comments {
                  comment
                  id
                  user {
                    name
                    id
                  }
                }
              }
            }
          `,
                    // Parameters
                    variables: {
                        commentID: this.selectedComment.id,
                        userID: 1,
                        resourceType: this.resourceType,
                        resourceID: this.id,
                        comment: this.comment
                    }
                })
                .then(data => (this.resourceComments = data.data.updateCommentMutation))
                .catch(error => {
                    // Error
                    console.error(error);
                });
        },
        deleteComment(commentID) {
            this.$apollo
                .mutate({
                    // Query
                    mutation: gql `
            mutation(
              $commentID: ID!
              $userID: ID!
              $resourceID: ID!
              $resourceType: String!
            ) {
              deleteCommentMutation(
                commentID: $commentID
                userID: $userID
                resourceType: $resourceType
                resourceID: $resourceID
              ) {
                comment
                id
                user {
                  name
                  id
                }
                comments {
                  comment
                  id
                  user {
                    name
                    id
                  }
                }
              }
            }
          `,
                    // Parameters
                    variables: {
                        commentID: commentID,
                        userID: 1,
                        resourceType: this.resourceType,
                        resourceID: this.id
                    }
                })
                .then(data => (this.resourceComments = data.data.deleteCommentMutation))
                .catch(error => {
                    // Error
                    console.error(error);
                });
        },
        createComment() {
            this.$apollo
                .mutate({
                    // Query
                    mutation: gql `
            mutation(
              $resourceID: ID!
              $resourceType: String!
              $comment: String!
              $userID: ID!
            ) {
              createCommentMutation(
                resourceID: $resourceID
                resourceType: $resourceType
                comment: $comment
                userID: $userID
              ) {
                comment
                id
                user {
                  name
                  id
                }
                comments {
                  comment
                  id
                  user {
                    name
                    id
                  }
                }
              }
            }
          `,
                    // Parameters
                    variables: {
                        resourceID: this.id,
                        resourceType: this.resourceType,
                        comment: this.comment,
                        userID: 1
                    }
                })
                .then(data => (this.resourceComments = data.data.createCommentMutation))
                .catch(error => {
                    // Error
                    console.error(error);
                });
        },
        handleCommentButton() {
            const actions = {
                update: this.handleUpdateComment,
                create: this.createComment,
                response: this.handleResponseComment
            };
            actions[this.commentAction]();
            this.cancelComment();
        },
        parseChapters(chapters) {
            chapters = JSON.parse(chapters);
            this.tables = Object.keys(chapters).reduce((acc, key) => {
                this.selected[key] = [];
                this.singleSelect[key] = false;
                acc[key] = Object.keys(chapters[key]["chapters"]).reduce(
                    (acc, value) => {
                        acc.push({
                            title: value
                        });
                        return acc;
                    },
                    []
                );
                return acc;
            }, {});
        }
    }
};
</script>

<style scoped>
.manga-info p {
    font-size: 1.2rem;
}

.manga-info p:hover {
    background-color: rgb(66, 135, 245);
    border-radius: 2px;
    padding: 3px;
    font-size: 1.5rem;
}
</style>
