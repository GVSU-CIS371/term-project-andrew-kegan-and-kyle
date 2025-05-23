<template>
<!--if user isn't signed in-->
<div v-if="store.currentUser===null">
    <ul class = "sign-in">
        <li>
            <input type="text" v-model="enteredUser" placeholder="Username">
        </li>
        <li>
            <div class="password-container">
            <input :type="passwordVisible ? 'text' : 'password'" v-model="enteredPwd" placeholder="Password">
            <button type="button" class="toggle-password" @click="togglePasswordVisibility">
                {{ passwordVisible ? 'Hide' : 'Show' }}
            </button>
    </div>
        </li>
        <li>
            <button @click="signIn">Sign In</button>
            <button @click="signUp">Sign Up</button>
        </li>
        <li class = "error">
            {{ errorMsg }}
        </li>
    </ul>
    <h2>Sign in to view your posts!</h2>
</div>
<!--otherwise-->
<div v-else>

    <li>
        <button @click="logOut">Log out</button>
        <!--admin does not get to delete account-->
        <button v-if="store.currentUser.username!=='admin'" @click="deleteAccount">Delete Account</button>
    </li>
    <li v-if="deleting">
        <h3>Are you sure?</h3>
        <button @click="confirmDelete">Confirm Deletion</button>
        <button @click="cancelDelete">Cancel</button>
    </li>
    
    <span id="title">{{ store.currentUser.username }}</span>

    <body class="favcolor" v-if="store.currentUser.favcolor==='Gray'">
        <!--site bullies you into choosing your favorite color-->
        You haven't chosen a favorite color yet! <br>
        <label for="dropdown">Select an option:</label><br>
            <select id="dropdown" v-model="selected" @change="updateFavColor">
                <option disabled value="">Please select one</option>
                <option v-for="color in colors" :key="color" :value="color">
                {{ color }}
                </option>
            </select>
    </body>

    <!--display posts by currently logged in user-->
    <template v-for="post in store.posts">
        <ul>
            <!--if post is by currently logged in user and collapsed is true, display title + timestamp with "expand" option-->
            <li v-if="post.author.username===store.currentUser.username">
                <UserPosts :post=post :comments=post.comments />
            </li>
        </ul>
    </template>
</div>


</template>

<script setup lang="ts">
import UserPosts from "../components/UserPosts.vue"
import { useStore } from "../stores/store"
const store = useStore();
import { onMounted, ref } from 'vue';
const enteredUser = ref('');
const enteredPwd = ref('');
const errorMsg = ref('');
const deleting = ref(false);
const passwordVisible = ref(false);

// When user signs in, set the selected color to their saved favorite
onMounted(() => {
  if (store.currentUser && store.currentUser.favcolor) {
    selected.value = store.currentUser.favcolor;
  }
});

// Sign in function
const signIn = async () => {
    const loginSuccess = await store.signIn(enteredUser.value, enteredPwd.value);
    enteredUser.value = '';
    enteredPwd.value = '';
    if (loginSuccess===true){
        errorMsg.value = '';
        // Set selected color to user's favorite if it exists
        if (store.currentUser && store.currentUser.favcolor) {
            selected.value = store.currentUser.favcolor;
        }
    }
    else {
        errorMsg.value = "Username or password was incorrect";
    }
}

const signUp = async () => {
    const username = enteredUser.value;
    const password = enteredPwd.value;
    //check if username and password are the same
    if(username === password){
        errorMsg.value = "Username and password cannot match."
        return;
    }
    const usernamePattern = new RegExp("^[a-z0-9-]{1,25}$");
    const passwordPattern = new RegExp("^[a-zA-Z0-9]{1,25}$");
    
    if (!usernamePattern.test(username)) {
        errorMsg.value = "Username can only contain lowercase letters, numbers, and hyphens (-). Must be 1-25 characters.";
        return;
    }
    if (!passwordPattern.test(password)){
        errorMsg.value = "Password can only contain alphanumeric characters. Must be 1-25 characters.";
        return;
    }
    const signupSuccess = await store.createUser(enteredUser.value, enteredPwd.value);
    enteredUser.value = '';
    enteredPwd.value = '';
    if(signupSuccess===true){
        errorMsg.value = '';
    }
    else {
        errorMsg.value = "Couldn't create new user -- password was empty or username already exists";
    }
}

// Log out function
const logOut = () => {
  store.logOut();
};

// Function to update favorite color
const updateFavColor = async () => {
  if (store.currentUser && selected.value) {
    await store.updateUserFavColor(selected.value);
  }
}

const togglePasswordVisibility = () => {
    passwordVisible.value = !passwordVisible.value;
};

const deleteAccount = () => {
    deleting.value = !deleting.value;
}
const cancelDelete = () => {
    deleting.value = !deleting.value;
}

const confirmDelete = () => {
    //delete all posts by the current user
    for(const post of store.posts){
        //if a post is by the user being deleted
        if(store.currentUser!==null && post.author.username === store.currentUser.username){
            //delete it
            store.confirmDelete(post);
        }
    }
    if(store.currentUser!=null){
        //delete the user
        store.deleteUser(store.currentUser);
    }
    deleting.value = !deleting.value;
    
}

const selected = ref('');
const colors = ['Red', 'Orange', 'Yellow', 'Green', 'Blue', 'Purple', 'Pink'];

</script>

<style lang="scss">

html {
    margin: 10px;
    padding: 10px;
    background-color: whitesmoke
}


button {
        padding: 5px 10px;
        margin: 10px;
        background-color: #4b0082;
        color: white;
        border: none;
        border-radius: 3px;
        cursor: pointer;

        &:hover {
            background-color: #6800b2
        }
    }

ul {
    list-style: none;
}
li {
    list-style: none;
}
.sign-in {
    margin: 10px;
    padding: 10px;

    input[type="text"] {
        padding: 5px;
        margin-right: 10px;
        margin-bottom: 10px;
    }
}

.error {
    color: red;
    margin: 10px;
}

.favcolor {
    background: whitesmoke;
    margin: 20px;
}

.password-container {
    position: relative;
    //display: flex;
    align-items: center;
}

.toggle-password {
    margin-left: 5px;
    padding: 3px 8px;
    font-size: 0.8em;
    background-color: #e0e0e0;
    color: #333;
}

.toggle-password:hover {
    background-color: #d0d0d0;
}

</style>