<script>
import {onMount} from "svelte";
import {get} from "svelte/store";
import count from "../stores/user.js";
import {navigate} from "svelte-routing";
import {collection, getDocs, getFirestore, onSnapshot} from "firebase/firestore";
import {initializeApp} from "firebase/app";

let user = [];
let todos = [];

const firebaseConfig = {
    apiKey: import.meta.env.VITE_API_KEY,
    authDomain: "mileage-app-3d53c.firebaseapp.com",
    projectId: "mileage-app-3d53c",
    storageBucket: "mileage-app-3d53c.appspot.com",
    messagingSenderId: "253164655909",
    appId: "1:253164655909:web:60b8c3a3ad2550bf8ab6cc",
    measurementId: "G-0P6K23YY2C"
};

onMount (async() => {
    user = get(count)
    if(user.email === undefined)
    {
        navigate("/", {"replace": true })
    }
    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);
    onSnapshot(collection(db, "mileage_lookups"), (snapshot) => {
        todos = snapshot.docs;
    });

    console.log(todos);

});

const history = async () => {
    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);
    const querySnapshot = await getDocs(collection(db, "mileage_lookups"));
    querySnapshot.forEach((doc) => {

        todos.push(doc)
    });
    console.log(todos);

}
</script>
<main class="bg-gray-200 h-screen">
    <center>
    <h1 class="text-3xl font-bold">Mileage Lookup History Table</h1>
    </center>
    <br>
    <br>
    <center>
    <table class="border border-slate-900 bg-white w-4/5 overflow-y-auto">
        <tr>
            <th>Origin City</th>
            <th>Origin State</th>
            <th>Origin Zip</th>
            <th>Destination City</th>
            <th>Destination State</th>
            <th>Destination Zip</th>
            <th>Miles</th>
        </tr>
        {#each todos as todo, index}
                <tr class="font-semibold">
                    <td class="border border-slate-900">
                        {todo.data().origin_city}
                    </td>
                    <td class="border border-slate-900">
                        {todo.data().origin_state}
                    </td>
                    <td class="border border-slate-900">
                        {todo.data().origin_zip}
                    </td>
                    <td class="border border-slate-900">
                        {todo.data().dest_city}
                    </td>
                    <td class="border border-slate-900">
                        {todo.data().dest_state}
                    </td>
                    <td class="border border-slate-900">
                        {todo.data().dest_zip}
                    </td>
                    <td class="border border-slate-900">
                        {todo.data().miles}
                    </td>
                </tr>
       {/each}
   </table>
    </center>
</main>