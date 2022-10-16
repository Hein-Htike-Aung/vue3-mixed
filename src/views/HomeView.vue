<template>
  <div class="home">
    <h1>Recipes</h1>
    <button @click="togglePopup">Add new Recipe</button>

    <!-- Items -->
    <div class="recipes">
      <div
        class="card"
        v-for="(recipe, idx) in $store.state.recipes"
        :key="idx"
      >
        <h2>{{ recipe.title }}</h2>
        <p>{{ recipe.description }}</p>
        <router-link :to="{ name: 'recipe', params: { slug: recipe.slug } }">
          <button>View Recipe</button>
        </router-link>
        <button style="margin-left: 30px" @click="edit(recipe)">Edit</button>
        <button style="margin-left: 30px" @click="deleteRecipe(recipe)">
          Delete
        </button>
      </div>
    </div>

    <!-- Popup -->
    <div class="add-recipe-popup" v-if="popupOpen">
      <div class="popup-content">
        <h2>{{ onEdit ? "Edit" : "Add New" }} Recipe</h2>
        <form @submit.prevent="addNewRecipe">
          <div class="group">
            <label>Title</label>
            <input type="text" required v-model="newRecipe.title" />
          </div>

          <div class="group">
            <label>Description</label>
            <textarea v-model="newRecipe.description" required></textarea>
          </div>

          <div class="group">
            <label>Ingredients</label>
            <div
              class="ingredient"
              v-for="(item, idx) in newRecipe.ingredients"
              :key="idx"
            >
              <input
                required
                type="text"
                v-model="newRecipe.ingredients[idx]"
              />
              <span class="removeIcon" @click="removeIngredient(idx)">x</span>
            </div>
            <button type="button" @click="addNewIngredient">
              Add Ingredient
            </button>
          </div>

          <div class="group">
            <label>Method</label>
            <div
              class="ingredient"
              v-for="(item, idx) in newRecipe.method"
              :key="idx"
            >
              <textarea required v-model="newRecipe.method[idx]"></textarea>
              <span class="removeIcon" @click="removeStep(idx)">x</span>
            </div>
            <button type="button" @click="addNewStep">Add Step</button>
          </div>

          <button type="submit">
            {{ onEdit ? "Edit" : "Add New" }} Recipe
          </button>
          <button @click="togglePopup" type="button">Close</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
import { useStore } from "vuex";

export default {
  name: "HomeView",
  components: {},
  setup() {
    const newRecipe = ref({
      id: 0,
      title: "",
      description: "",
      ingredients: [""],
      method: [""],
    });
    const popupOpen = ref(false);
    const onEdit = ref(false);
    const store = useStore();

    const togglePopup = () => {
      popupOpen.value = !popupOpen.value;
    };

    const addNewIngredient = () => {
      newRecipe.value.ingredients.push("");
    };

    const removeIngredient = (idx) => {
      if (newRecipe.value.ingredients.length > 1) {
        newRecipe.value.ingredients.splice(idx, 1);
      }
    };

    const removeStep = (idx) => {
      if (newRecipe.value.method.length > 1) {
        newRecipe.value.method.splice(idx, 1);
      }
    };

    const addNewStep = () => {
      newRecipe.value.method.push("");
    };

    const edit = (recipe) => {
      newRecipe.value = {
        id: recipe.id,
        title: recipe.title,
        description: recipe.description,
        ingredients: recipe.ingredients,
        method: recipe.method,
      };
      onEdit.value = true;
      togglePopup();
    };

    const deleteRecipe = (recipe) => {
      store.commit("DELETE_RECIPE", recipe);
    };

    const addNewRecipe = () => {
      newRecipe.value.slug = newRecipe.value.title
        .toLowerCase()
        .replace(/\s/g, "-");

      if (newRecipe.value.slug === "") {
        alert("Please enter a title");
        return;
      }

      if (onEdit.value) {
        store.commit("UPDATE_RECIPE", { ...newRecipe.value });
        onEdit.value = false;
      } else {
        store.commit("ADD_RECIPE", { ...newRecipe.value });
      }

      newRecipe.value = {
        id: 0,
        title: "",
        description: "",
        ingredients: [""],
        method: [""],
      };

      togglePopup();
    };

    return {
      newRecipe,
      removeStep,
      popupOpen,
      edit,
      deleteRecipe,
      removeIngredient,
      togglePopup,
      addNewIngredient,
      addNewStep,
      addNewRecipe,
      onEdit,
    };
  },
};
</script>

<style scoped>
.home {
  padding: 1rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}
h1 {
  font-size: 3rem;
  margin-bottom: 32px;
}
.recipes {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
}
.recipes .card {
  padding: 1rem;
  border-radius: 5px;
  margin: 1rem;
  background-color: #081c33;
}
.recipes .card h2 {
  font-size: 2rem;
  margin-bottom: 1rem;
}
.recipes .card p {
  font-size: 1.125rem;
  line-height: 1.4;
  margin-bottom: 1rem;
}
.add-recipe-popup {
  position: fixed;
  top: 10px;
  left: 25%;
  background-color: transparent;
  width: 50%;
  margin: auto;
  height: 100%;
  overflow: scroll;
  scrollbar-width: none;
}
.add-recipe-popup .popup-content {
  background-color: #081c33;
  padding: 2rem;
  border-radius: 1rem;
  width: 100%;
  min-height: fit-content;
  max-width: 768px;
  box-shadow: 10px 4px 6px -1px rgba(0, 0, 0, 0.2),
    0 2px 4px -1px rgba(0, 0, 0, 0.06);
}
.popup-content h2 {
  font-size: 2rem;
  margin-bottom: 1rem;
}
.popup-content .group {
  margin-bottom: 1rem;
}
.popup-content .group label {
  display: block;
  margin-bottom: 0.5rem;
}
.ingredient {
  position: relative;
}
.removeIcon {
  position: absolute;
  top: 0;
  right: 10px;
  font-size: 30px;
  cursor: pointer;
  color: black;
}

.popup-content .group input,
.popup-content .group textarea {
  display: block;
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin-bottom: 1rem;
}
.popup-content .group textarea {
  height: 100px;
  resize: none;
}
.popup-content button[type="submit"] {
  margin-right: 1rem;
}
</style>
