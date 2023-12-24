<script>
  import {Router, Link, Route, navigate} from "svelte-routing";
  import Hello from "./Components/Hello.svelte";
  import Login from "./Components/Login.svelte"
  import {onMount} from "svelte";
  import {get} from "svelte/store";
  import count from "./stores/user.js";
  import {initializeApp} from "firebase/app";
  import {getAuth, signOut} from "firebase/auth";
  import Miles from "./Components/Miles.svelte";
  import dotenv from 'dotenv';
  export let url = "";

  $count


  const firebaseConfig = {
    apiKey: import.meta.env.VITE_API_KEY,
    authDomain: "mileage-app-3d53c.firebaseapp.com",
    projectId: "mileage-app-3d53c",
    storageBucket: "mileage-app-3d53c.appspot.com",
    messagingSenderId: "253164655909",
    appId: "1:253164655909:web:60b8c3a3ad2550bf8ab6cc",
    measurementId: "G-0P6K23YY2C"
  };
  // Initialize Firebase
  const app = initializeApp(firebaseConfig);
  const auth = getAuth(app);

  const logout = async () => {
    signOut(auth);
    count.set(0)
  }

</script>

<main>
  <Router {url}>
    <nav>
      <Link to="/">Home |</Link>
      <Link to="/about">Route Lookup |</Link>
      <Link to="/mileagehistory">Mileage History |</Link>
      {#if $count.email !== undefined}
        <Link on:click={logout}>Logout</Link>
      {/if}
    </nav>
    <div>
      <Route path="/" component={Login} />
      <Route path="/about" component={Hello} />
      <Route path="/mileagehistory" component={Miles} />
    </div>
  </Router>




</main>

<style>

</style>
