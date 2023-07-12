<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";
import { useStore } from "vuex";
import { useRouter } from "vue-router";

const user = ref({
  username: "",
  password: "",
});
// 
const store = useStore();
const router = useRouter();

const submit = async () => {
  try {
    const response = await axios.get("/db.json");
    const users = response.data;

    const userInList = users.find(
      (u) =>
        u.username === user.value.username && u.password === user.value.password
    );
    if (userInList) {
      store.dispatch("login", userInList);
      router.push("/tasks");
    } else {
      alert("Invalid credentials");
    }

    user.value.username = "";
    user.value.password = "";
  } catch (error) {
    console.error("Error fetching users:", error);
    alert("Error fetching users. Please try again.");
  }
};
</script>

<template>
 <h4 v-if="!isLoggedIn" class="login-text">Please Login</h4>
  <div class="login-form-container">
    
    <div class="login-form">
      <form @submit.prevent="submit">
        <div class="mb-3">
          <label for="username" class="form-label">Username</label>
          <input
            type="text"
            class="form-control"
            id="username"
            v-model="user.username"
          />
        </div>
        <div class="mb-3">
          <label for="password" class="form-label">Password</label>
          <input
            type="password"
            class="form-control"
            id="password"
            v-model="user.password"
          />
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
      </form>
    </div>
  </div>
</template>

<style scoped>
.login-form-container {
  width: 300px;
  position: fixed;
  top: 35%;
  left: 40%;
  padding: 20px;
  border: 1px solid;
  border-radius: 30px;
  z-index: 9999;
}
</style>
