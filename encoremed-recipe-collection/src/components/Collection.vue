<template>
    <div class="row justify-content-center mt-5 collection">
        <div class="col-12">
            <h1>My Collection</h1>
        </div>

        <div v-if="productChunks.length > 0" id="multiCarousel" class="carousel slide" data-bs-ride="carousel"
            data-bs-wrap="false">
            <div class="carousel-inner">
                <div v-for="(item, index) in productChunks" :key="index"
                    :class="['carousel-item', { active: index === 0 }]">
                    <div class="row">
                        <div v-for="(recipe, idx) in item" :key="idx" class="col-12 col-md-4 ">
                            <div class="card p-3">
                                <div class="d-flex justify-content-between">
                                    <h5 class="card-title text-truncate">{{ recipe.name }}</h5>
                                    <a @click="unfavorite(recipe.name)" v-if="status(recipe.name)"><i
                                            class="bi bi-heart-fill"></i></a>
                                    <a @click="favorite(recipe.name)" v-else><i class="bi bi-heart"></i></a>
                                </div>

                                <p class="card-text text-truncate">
                                    {{
                                        recipe.description || "No description, but lots of love"
                                    }}
                                </p>

                                <button type="button" class="btn btn-primary custom-btn" @click="view(recipe)">
                                    View Recipe
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <button class="carousel-control-prev" type="button" data-bs-target="#multiCarousel" data-bs-slide="prev">
                <span class="carousel-control-prev-icon"></span>
            </button>
            <button class="carousel-control-next" type="button" data-bs-target="#multiCarousel" data-bs-slide="next">
                <span class="carousel-control-next-icon"></span>
            </button>
        </div>

        <div v-else class="text-center">
            <p>Oops! No recipes saved yet — your kitchen is still asleep 😴</p>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";

export default defineComponent({
    name: "Collection",
    props: {
        recipes: Object,
        favoriteRecipes: Object,
        favoriteList: Object
    },
    computed: {
        productChunks() {
            const chunkSize = window.innerWidth <= 768 ? 1 : 3; // 1 on mobile, 3 on desktop
            const chunks = [];

            for (let i = 0; i < this.favoriteRecipes.length; i += chunkSize) {
                chunks.push(this.favoriteRecipes.slice(i, i + chunkSize));
            }

            return chunks;
        }
    },
    methods: {
        status(name) {
            if (this.favoriteList.includes(name)) {
                return true;
            }
            return false;
        },
        unfavorite(name) {
            this.$emit("unfavorite", name)
        },
        favorite(name) {
            this.$emit("favorite", name)
        },
        view(name) {
            this.$emit("view", name)
        },
        
    }
});
</script>