<template>
    <div class="row justify-content-center mt-5 discover">
        <!-- begin::Title -->
        <div class="col-12">
            <h1>Discover</h1>
        </div>
        <!-- end::Title -->

        <!-- begin::Filter by Category | Search -->
        <div class="col-12 text-end">
            <!-- begin::Filter by Category -->
            <select v-model="selectedCategory" class="border p-2 rounded w-full mb-4 filter me-md-3 text-capitalize">
                <option value="">All Categories</option>
                <option v-for="cat in uniqueCategories" :key="cat" :value="cat">
                    {{ cat }}
                </option>
            </select>
            <!-- end::Filter by Category -->

            <!-- begin::Search -->
            <input v-model="searchQuery" type="text" placeholder="Search for a recipe..." class="search-input">
            <!-- end::Search -->
        </div>
        <!-- end::Filter by Category | Search -->

        <!-- begin::Public Recipe List -->
        <div v-if="recipes.length > 0" class="col-md-4 mt-card" v-for="recipe in recipes">
            <div class="card p-3">
                <div class="d-flex justify-content-between">
                    <h5 class="card-title text-truncate">{{ recipe.name }}</h5>
                    <a @click="unfavorite(recipe.name)" v-if="status(recipe.name)"><i class="bi bi-heart-fill"></i></a>
                    <a @click="favorite(recipe.name)" v-else><i class="bi bi-heart"></i></a>
                </div>

                <p class="card-text text-truncate">{{ recipe.description || "No description, but lots of love." }}</p>
                <img v-if="recipe.image.length > 0" :src="recipe.image[0]"
                    alt="The image is broken... Looks like it's on a coffee break!">
                <div v-else class="no-image d-flex justify-content-center align-items-center">
                    <p>Image got hungry. Ate itself</p>
                </div>

                <button type="button" class="btn btn-primary custom-btn" @click="view(recipe)">
                    View Recipe
                </button>
            </div>
        </div>

        <div v-else class="text-center">
            <p>Oops! No recipes found. Maybe the recipe fairy is on vacation 🧚‍♂️</p>
        </div>
        <!-- end::Public Recipe List -->
    </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";

export default defineComponent({
    name: "Discover",
    props: {
        uniqueCategories: Object,
        recipes: Object,
        favoriteList: Object,
        allRecipes: Object
    },
    data() {
        return {
            selectedCategory: "",
            searchQuery: "",
        }
    },
    watch: {
        selectedCategory: function () {
            if (this.selectedCategory || this.selectedCategory != '') {
                this.$emit('update', this.allRecipes.filter(r =>
                    r.recipeCategory.includes(this.selectedCategory)
                ));
            }
            else {
                this.$emit("getData")

            }
        },
        searchQuery: function () {
            const result = this.allRecipes.filter((recipe: any) =>
                recipe.name.toLowerCase().includes(this.searchQuery)
            );
            this.$emit('search', result);
            if (this.searchQuery == '') {
                this.$emit("getData")
            }
        },
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
        }
    }
});
</script>