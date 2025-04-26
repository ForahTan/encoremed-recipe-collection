<template>
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
                                    <li v-for="(step, index) in selectedRecipe.recipeInstructions || []" :key="index">
                                        <span v-if="step['itemListElement']">
                                            {{ step.name }}

                                            <ul>
                                                <li v-if="step['itemListElement']"
                                                    v-for="(sub_step, index2) in step['itemListElement']">
                                                    {{ sub_step.text }}
                                                </li>
                                            </ul>
                                        </span>

                                        <span v-else>
                                            {{
                                                step.text || step || 'No step description available'
                                            }}
                                        </span>
                                    </li>
                                </ol>

                                <p><strong>Prep Time:</strong> {{ selectedRecipe.prepTime }}</p>
                                <p><strong>Cook Time:</strong> {{ selectedRecipe.cookTime }}</p>
                                <p><strong>Total Time:</strong> {{ selectedRecipe.totalTime }}</p>

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
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";

export default defineComponent({
    name: "RecipePopup",
    props: {
        showModal: Boolean,
        selectedRecipe: Object
    },
    methods: {
        closeModal() {
            this.$emit("action");
        }
    }
});
</script>