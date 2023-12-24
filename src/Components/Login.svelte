<script>
    import { onMount } from 'svelte';
    import {navigate} from "svelte-routing";
    import {initializeApp } from 'firebase/app';
    import { getAuth, signInWithEmailAndPassword } from 'firebase/auth';
    import 'firebase/auth';
    import count from "../stores/user.js";
    import {get} from "svelte/store";




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
    let loggedinuser = ''

    let email = '';
    let password = '';

    const login = async () => {
        try {
            const userCredential = await signInWithEmailAndPassword(auth, email, password);
            const user = userCredential.user;
            count.set(user)
            console.log('Logged in as:', user.email);
            navigate("/about", {"replace": true })
            // You can redirect or perform other actions after successful login
        } catch (error) {
            console.error('Error during login:', error.message);
        }
    }

    const seeUser = async () => {
        loggedinuser = get(count);
    }
</script>

<main class="bg-gray-200">

<!--    <div class="bg-gray-200">-->
<!--    <form on:submit|preventDefault={login}>-->
<!--        <label for="email">Email:</label>-->
<!--        <input class="border-2 border-indigo-600" type="email" id="email" } />-->

<!--        <label for="password">Password:</label>-->
<!--        <input class="border-2 border-indigo-600" type="password" id="password" bind:value={password} />-->
<!--        <br>-->
<!--        <button type="submit">Login</button>-->
<!--    </form>-->

<!--    <h1>{loggedinuser.email}</h1>-->
<!--    <button on:click={seeUser}>See User</button>-->
<!--    </div>-->

    <section class="bg-gray-200 dark:bg-gray-900">
        <div class="flex flex-col items-center justify-center px-6 py-8 mx-auto md:h-screen lg:py-0">
            <div class="w-full bg-white rounded-lg shadow dark:border md:mt-0 sm:max-w-md xl:p-0 dark:bg-gray-800 dark:border-gray-700">
                <div class="p-6 space-y-4 md:space-y-6 sm:p-8">
                    <h1 class="text-xl font-bold leading-tight tracking-tight text-gray-900 md:text-2xl dark:text-white">
                        Sign in to your account
                    </h1>
                    <form on:submit|preventDefault={login} class="space-y-4 md:space-y-6" action="#">
                        <div>
                            <label for="email" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Your email</label>
                            <input bind:value={email} type="email" name="email" id="email" class="bg-gray-50 border border-gray-300 text-gray-900 sm:text-sm rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="name@company.com" required="">
                        </div>
                        <div>
                            <label for="password" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Password</label>
                            <input bind:value={password} type="password" name="password" id="password" placeholder="••••••••" class="bg-gray-50 border border-gray-300 text-gray-900 sm:text-sm rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" required="">
                        </div>

                        <button type="submit" class="w-full text-white bg-blue-600 hover:bg-primary-700 focus:ring-4 focus:outline-none focus:ring-primary-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-primary-600 dark:hover:bg-primary-700 dark:focus:ring-primary-800">Sign in</button>

                    </form>
                </div>
            </div>
        </div>
    </section>
</main>