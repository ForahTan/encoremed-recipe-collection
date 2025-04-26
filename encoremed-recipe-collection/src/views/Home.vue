<template>
    <MessageModal :message="message" :message-type="messageType" />

    <RecipePopup :show-modal="showModal" :selected-recipe="selectedRecipe" @action="closeModal" />

    <Collection :recipes="recipes" :favorite-recipes="favoriteRecipes"
        :favorite-list="favoriteList" @favorite="favorite" @unfavorite="unfavorite" @view="view" />

    <Discover :favorite-list="favoriteList"
        :unique-categories="uniqueCategories" :all-recipes="allRecipes" :recipes="recipes" @getData="getData"
        @favorite="favorite" @unfavorite="unfavorite" @update="update" @search="search" @view="view" />
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import MessageModal from "../components/MessageModal.vue"
import RecipePopup from "../components/RecipePopup.vue";
import Collection from "../components/Collection.vue";
import Discover from "@/components/Discover.vue";
import { Modal } from 'bootstrap';

export default defineComponent({
    name: "Home",
    components: {
        MessageModal,
        RecipePopup,
        Collection,
        Discover
    },
    data() {
        return {
            showModal: false,
            allRecipes: {} as any,
            recipes: {} as any,
            favoriteRecipes: [] as any,
            favoriteList: [] as any, // The list with name only
            message: null, // Will store the message for showing the popup
            messageType: '', // To distinguish between add/remove messages,
            selectedRecipe: [] as any,
            uniqueCategories: [] as any,
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
                this.formatInstruction();
                this.formatTime();
                this.formatCuisine();
                this.formatKeywords();

                this.allRecipes = this.recipes;
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
        updateFavoriteList() {
            this.favoriteRecipes = [];
            if (localStorage.getItem('favoriteList') && localStorage.getItem('favoriteList').length > 0) {
                this.favoriteList = localStorage.getItem('favoriteList').split(",");
                if (this.favoriteList.length > 0) {
                    this.favoriteRecipes = this.favoriteList.map(name =>
                        this.allRecipes.find(item => item.name === name)
                    );
                }
            }
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
                const modal = new Modal(document.getElementById('messageModal')!);
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
                const modal = new Modal(document.getElementById('messageModal'));
                modal.show(); // Show the modal
            });
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
        },
        formatInstruction() {
            this.recipes.forEach(element => {
                if (typeof element.recipeInstructions === 'string') {
                    element.recipeInstructions = element.recipeInstructions
                        .split('\n')          // Split by line breaks
                        .map(line => line.trim()) // Remove extra spaces
                        .filter(line => line)     // Remove empty line
                }
            });
        },
        formatTime() {
            this.recipes.forEach(element => {
                element.prepTime = this.formatTimeText(element.prepTime);
                element.cookTime = this.formatTimeText(element.cookTime);
                element.totalTime = this.formatTimeText(element.totalTime);
            });
        },
        formatTimeText(time) {
            if (time === undefined) {
                return "N/A";
            }
            else {

                // Remove the "PT" part, leaving only the time information
                time = time.slice(2);

                // Regular expression to match the hours, minutes, and seconds
                const regex = /^(\d+H)?(\d+M)?(\d+S)?$/;

                const match = time.match(regex);
                let formattedTime = '';

                if (match) {
                    // If hours are found, add them to the formatted string
                    if (match[1]) {
                        formattedTime += match[1].replace('H', ' hour') + ' ';
                    }
                    // If minutes are found, add them to the formatted string
                    if (match[2]) {
                        formattedTime += match[2].replace('M', ' minute') + ' ';
                    }
                    // If seconds are found, add them to the formatted string
                    if (match[3]) {
                        formattedTime += match[3].replace('S', ' second') + ' ';
                    }
                }
                else {
                    formattedTime = 'N/A'
                }

                return formattedTime.trim();
            }
        },
        formatCuisine() {
            this.recipes.forEach(element => {
                if (element.recipeCuisine === undefined) {
                    element.recipeCuisine = [];
                }

                if (typeof element.recipeCuisine === 'string') {
                    element.recipeCuisine = [element.recipeCuisine.trim()];
                    element.recipeCuisine = element.recipeCuisine[0].split(",");
                }

            });
        },
        formatKeywords() {
            this.recipes.forEach(element => {
                if (element.keywords === undefined) {
                    element.keywords = "N/A";
                }
            });
        },
        update(value) {
            this.recipes = value;
        },
        search(value) {
            this.recipes = value;
        }

    }
});
</script>