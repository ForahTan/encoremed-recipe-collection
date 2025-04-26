<template>
    <!-- begin::Message Modal -->
    <div class="modal fade" id="messageModal" tabindex="-1" aria-labelledby="messageModalLabel" aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="messageModalLabel">{{ messageType === 'add' ? 'Added!' : 'Removed!' }}
                    </h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <p>{{ message }}</p>
                </div>
            </div>
        </div>
    </div>
    <!-- end::Message Modal -->

    <!-- begin::Recipe Popup -->
    <div v-if="showModal" class="modal fade show" style="display: block;" tabindex="-1"
        aria-labelledby="recipeModalLabel" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered modal-lg">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="recipeModalLabel">{{ selectedRecipe.name || 'Recipe Name' }}</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" @click="closeModal"
                        aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <div class="row">
                        <!-- Left Column: Image Carousel -->
                        <div class="col-md-5">
                            <div id="imageCarousel" class="carousel slide" data-bs-ride="carousel">
                                <div class="carousel-inner">
                                    <div v-for="(image, index) in selectedRecipe.image" :key="index"
                                        class="carousel-item" :class="{ active: index === 0 }">
                                        <img :src="image" class="d-block w-100" alt="Recipe Image">
                                    </div>
                                </div>
                                <!-- Custom Carousel Controls -->

                                <button class="carousel-control-prev" type="button" data-bs-target="#imageCarousel"
                                    data-bs-slide="prev">
                                    <span class="carousel-control-prev-icon"></span>
                                </button>
                                <button class="carousel-control-next" type="button" data-bs-target="#imageCarousel"
                                    data-bs-slide="next">
                                    <span class="carousel-control-next-icon"></span>
                                </button>
                            </div>
                        </div>
                        <!-- Right Column: Recipe Details -->
                        <div class="col-md-7">
                            <div class="recipe-detail-container">
                                <p><strong>Description:</strong>
                                    {{ selectedRecipe.description || "No description available" }}</p>
                                <p><strong>Ingredients:</strong></p>
                                <ul>
                                    <li v-for="(ingredient, index) in selectedRecipe.recipeIngredient || []"
                                        :key="index">{{ ingredient }}</li>
                                </ul>

                                <p><strong>Steps:</strong></p>
                                <ol>
                                    <li v-for="(step, index) in selectedRecipe.recipeInstructions || []" :key="index">{{
                                        step.text || 'No step description available' }}</li>
                                </ol>

                                <p><strong>Prep Time:</strong> {{ selectedRecipe.prepTime || 'N/A' }}</p>
                                <p><strong>Cook Time:</strong> {{ selectedRecipe.cookTime || 'N/A' }}</p>
                                <p><strong>Total Time:</strong> {{ selectedRecipe.totalTime || 'N/A' }}</p>

                                <h6>Nutrition:</h6>
                                <p><strong>Calories:</strong> {{ selectedRecipe.nutrition?.calories || 'N/A' }}</p>
                                <p><strong>Carbohydrates:</strong> {{ selectedRecipe.nutrition?.carbohydrateContent ||
                                    'N/A' }}g</p>

                                <p><strong>Publisher:</strong> {{ selectedRecipe.publisher?.name || 'Unknown Publisher'
                                }}</p>
                                <p><strong>Category:</strong> {{ selectedRecipe.recipeCategory || 'N/A' }}</p>
                                <p><strong>Cuisine:</strong> {{ selectedRecipe.recipeCuisine || 'N/A' }}</p>

                                <p><strong>Keywords:</strong> {{ selectedRecipe.keywords || 'N/A' }}</p>

                                <a :href="selectedRecipe.url || '#'">View Full Recipe</a>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <!-- end::Recipe Popup -->

    <!-- begin::My Collection -->
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

                                <p class="card-text text-truncate">{{ recipe.description || "~" }}</p>

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
    <!-- end::My Collection -->

    <!-- begin::Discover -->
    <div class="row justify-content-center mt-5 discover">
        <!-- begin::Title -->
        <div class="col-12">
            <h1>Discover</h1>
        </div>
        <!-- end::Title -->

        <!-- begin::Filter by Category | Search -->
        <div class="col-12 text-end">
            <!-- begin::Filter by Category -->
            <select v-model="selectedCategory" class="border p-2 rounded w-full mb-4 filter me-3 text-capitalize">
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

                <p class="card-text text-truncate">{{ recipe.description || "~" }}</p>
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
    <!-- end::Discover -->
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";

export default defineComponent({
    name: "Home",
    data() {
        return {
            showModal: false,
            recipes: {} as any,
            favoriteList: [] as any, // The list with name only
            favoriteRecipes: [] as any, //The list with details
            message: null, // Will store the message for showing the popup
            messageType: '', // To distinguish between add/remove messages,
            selectedRecipe: [] as any,
            searchQuery: "",
            selectedCategory: "",
            uniqueCategories: [] as any,
        }
    },
    watch: {
        searchQuery: function () {
            this.recipes = this.recipes.filter((recipe: any) =>
                recipe.name.toLowerCase().includes(this.searchQuery)
            );
            if (this.searchQuery == '') {
                this.getData();
            }
        },
        selectedCategory: function () {
            // this.getData();
            if (this.selectedCategory || this.selectedCategory != '') {
                this.recipes = this.recipes.filter((recipe: any) => {
                    // Ensure recipeCategory is a string or an array before attempting to split
                    let categories: string[] = [];

                    if (recipe.recipeCategory) {
                        if (Array.isArray(recipe.recipeCategory)) {
                            categories = recipe.recipeCategory;
                        } else if (typeof recipe.recipeCategory === 'string') {
                            categories = recipe.recipeCategory.split(',').map((cat: string) => cat.trim());
                        }
                    }
                    return categories.some((category: string) => category === this.selectedCategory);
                });
            }
        }
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
    mounted() {
        this.getData();
    },
    methods: {
        getData() {
            this.$axios.get('https://raw.githubusercontent.com/micahcochran/json-cookbook/refs/heads/main/cookbook-100.json').then((response) => {
                this.recipes = response.data;

                this.formatImages();
                this.formatCategories();

                this.recipes.forEach(element => {
                    console.log(element.recipeCategory);
                });

                this.updateFavoriteList();
                this.getCategoryList();
            })
        },
        objectCount(object) {
            if (typeof object === 'object') {
                return Object.keys(object).length;
            }
            return 0;
        },
        status(name) {
            if (this.favoriteList.includes(name)) {
                return true;
            }
            return false;
        },
        favorite(name) {
            if (!this.favoriteList.includes(name)) {
                this.favoriteList.unshift(name);
                localStorage.clear();
                localStorage.setItem("favoriteList", this.favoriteList);
            }
            this.updateFavoriteList();

            this.message = `${name} has been added!`;  // Popup message for add
            this.messageType = 'add'; // Set message type

            // Show the modal
            this.$nextTick(() => {
                const modal = new bootstrap.Modal(document.getElementById('messageModal'));
                modal.show(); // Show the modal
            });
        },
        unfavorite(name) {
            if (this.favoriteList.includes(name)) {
                this.favoriteList = this.favoriteList.filter(item => item !== name)
                localStorage.clear();
                localStorage.setItem("favoriteList", this.favoriteList);
            }
            this.updateFavoriteList();

            this.message = `${name} has been removed!`; // Popup message for remove
            this.messageType = 'remove'; // Set message type

            // Show the modal
            this.$nextTick(() => {
                const modal = new bootstrap.Modal(document.getElementById('messageModal'));
                modal.show(); // Show the modal
            });
        },
        updateFavoriteList() {
            this.favoriteRecipes = [];
            if (localStorage.getItem('favoriteList') && localStorage.getItem('favoriteList').length > 0) {
                this.favoriteList = localStorage.getItem('favoriteList').split(",");
                if (this.favoriteList.length > 0) {
                    this.favoriteRecipes = this.favoriteList.map(name =>
                        this.recipes.find(item => item.name === name)
                    );
                }
            }
        },
        view(recipe) {
            this.showModal = true;
            this.selectedRecipe = {};
            this.selectedRecipe = recipe;
        },
        closeModal() {
            this.showModal = false;
        },
        getCategoryList() {
            const allCategories: string[] = this.recipes.flatMap((recipe: any) => {
                const rawCategory = recipe.recipeCategory

                // If undefined/null, return empty array
                if (!rawCategory) return []

                // Normalize: if string, convert to array
                const categories = Array.isArray(rawCategory)
                    ? rawCategory
                    : [rawCategory]

                return categories.flatMap((cat: string) =>
                    typeof cat === 'string' ? cat.split(',') : []
                )
            })

            const cleaned = allCategories.map(c => c.trim()).filter(c => c.length > 0)
            this.uniqueCategories = Array.from(new Set(cleaned)).sort()
        },
        capitalizeFirstLetter(string: string): string {
            return string
                .split(' ')
                .map(word => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
                .join(' ');
        },
        formatImages() {
            this.recipes.forEach(element => {
                if (element.image === undefined) {
                    element.image = [];
                }

                if (typeof element.image === 'string') {
                    element.image = [element.image.trim()];
                }
            });
        },
        formatCategories() {
            this.recipes.forEach(element => {
                if (element.recipeCategory === undefined) {
                    element.recipeCategory = [];
                }

                if (typeof element.recipeCategory === 'string') {
                    element.recipeCategory = [element.recipeCategory.trim()];
                    element.recipeCategory = element.recipeCategory[0].split(",");
                }

                if (Array.isArray(element.recipeCategory)) {
                    // If it's an array, capitalize each category
                    element.recipeCategory = element.recipeCategory.map((cat: string) => this.capitalizeFirstLetter(cat));
                } else if (typeof element.recipeCategory === 'string') {
                    // If it's a string, split, capitalize, and join back
                    element.recipeCategory = element.recipeCategory.split(',').map((cat: string) => this.capitalizeFirstLetter(cat.trim())).join(',');
                }
            });
        }
    }
});
</script>