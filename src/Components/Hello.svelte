<script>
    import {onMount} from "svelte";
    import { get } from "svelte/store";
    import count from "../stores/user.js";
    import {navigate} from "svelte-routing";
    import { query, where, getFirestore, collection, getDocs, limit, addDoc, doc } from "firebase/firestore";


    // const api_key = 'Am_VKTj9GfyGPlLwz5W1erJr-5NKJLUBNb6CNxMJBO8MQCNKjSTyzW37YzAgN41A'
    const api_key = import.meta.env.VITE_MILES_KEY
    let origin_zip = ''
    let destination_zip = ''
    let result_origin_city = ''
    let result_origin_zip = ''
    let result_origin_state = ''
    let result_dest_city = ''
    let result_dest_zip = ''
    let result_dest_state = ''
    let origin_x = ''
    let origin_y = ''
    let dest_x = ''
    let dest_y = ''
    let total_mileage = ''

    const firebaseConfig = {
        apiKey: import.meta.env.VITE_API_KEY,
        authDomain: "mileage-app-3d53c.firebaseapp.com",
        projectId: "mileage-app-3d53c",
        storageBucket: "mileage-app-3d53c.appspot.com",
        messagingSenderId: "253164655909",
        appId: "1:253164655909:web:60b8c3a3ad2550bf8ab6cc",
        measurementId: "G-0P6K23YY2C"
    };

    import axios from 'axios';
    import {initializeApp} from "firebase/app";


    let url_example = 'https://dev.virtualearth.net/REST/V1/Routes/Truck?o=xml&wp.0=60515&wp.1=60661&avoid=minimizeTolls&key=AnpZUGjRmAYAJAObOONxLq0AT0NThtsk4wngg76vNuNGdWd6JW87U57q1hc8otCy'

    let user = [];

    onMount (async() => {
        user = get(count)
        if(user.email === undefined)
        {
            navigate("/", {"replace": true })
        }

    });

    const mileageLookup = async () => {
        //console.log(`origin: ${origin_zip} \ndestination: ${destination_zip}`)
            let historyResult = await historyLookup();
            if(!historyResult) {
                await originLookup()
                await destinationLookup();
                await theMileagePiece();
                await addRoute();
                console.log("here")
            }
            origin_zip = '';
            destination_zip = '';
  }

    async function originLookup() {
        let first_origin_zip
        let origin_url = `http://dev.virtualearth.net/REST/v1/Locations/US/-/${origin_zip}/-/-?o=json&inclnb=1&key=${api_key}`
        await axios.get(origin_url).then(function (response){
            let coords = response.data

            //console.log(coords.resourceSets[0].resources[0].point.coordinates[0])
            // origin_coords = [coords.resourceSets[0].resources[0].point.coordinates[0], coords.resourceSets[0].resources[0].point.coordinates[1]]
            const obj = JSON.parse(JSON.stringify(response.data))
            origin_x = coords.resourceSets[0].resources[0].point.coordinates[0]
            origin_y = coords.resourceSets[0].resources[0].point.coordinates[1]
            //origin_coords = obj.resourceSets[0].resources[0].point.coordinates;
            result_origin_city = obj.resourceSets[0].resources[0].address.locality;
            result_origin_state = obj.resourceSets[0].resources[0].address.adminDistrict;
            result_origin_zip = obj.resourceSets[0].resources[0].address.postalCode;
           first_origin_zip = origin_x


        }).catch(function (error){
            console.log(error);
        })
        return first_origin_zip
    }

    async function destinationLookup() {
        let dest_url = `http://dev.virtualearth.net/REST/v1/Locations/US/-/${destination_zip}/-/-?o=json&inclnb=1&key=${api_key}`
        await axios.get(dest_url).then(function (response){
            let coords = response.data
            const obj = JSON.parse(JSON.stringify(response.data))
            // dest_coords = obj.resourceSets[0].resources[0].point.coordinates;
            dest_x = coords.resourceSets[0].resources[0].point.coordinates[0]
            dest_y = coords.resourceSets[0].resources[0].point.coordinates[1]
            result_dest_city = obj.resourceSets[0].resources[0].address.locality;
            result_dest_state = obj.resourceSets[0].resources[0].address.adminDistrict;
            result_dest_zip = obj.resourceSets[0].resources[0].address.postalCode;
        }).catch(function (error){
            console.log(error);
        })
    }
    async function theMileagePiece() {
        let distance_matrix_url = `https://dev.virtualearth.net/REST/v1/Routes/DistanceMatrix?origins=${origin_x},${origin_y}&destinations=${dest_x},${dest_y}&travelMode=driving&key=${api_key}`
        await axios.get(distance_matrix_url).then(function (response){
           total_mileage = Math.round(response.data.resourceSets[0].resources[0].results[0].travelDistance)

        })
        //console.log(distance_matrix_url)
    }

    const historyLookup = async () => {
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);

        const citiesRef = collection(db, "mileage_lookups");
        const q1 = query(citiesRef, where("origin_zip", "==", origin_zip), where("dest_zip", "==", destination_zip), limit(1));
        const docSnap = await getDocs(q1);
        if(docSnap.docs.length > 0)
        {
        docSnap.forEach((doc) => {
            result_origin_city = doc.data().origin_city
            result_origin_state = doc.data().origin_state
            result_origin_zip = doc.data().origin_zip
            result_dest_city = doc.data().dest_city
            result_dest_state = doc.data().dest_state
            result_dest_zip = doc.data().dest_zip
            total_mileage = doc.data().miles
        });
        return true;
        }
    return false
    }

    async function addRoute(){
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);

        await addDoc(collection(db, "mileage_lookups"), {
            origin_city: result_origin_city,
            origin_state: result_origin_state,
            origin_zip: result_origin_zip,
            dest_city: result_dest_city,
            dest_state: result_dest_state,
            dest_zip: result_dest_zip,
            miles: total_mileage
        });

    }

    const clearResults = async () => {
        origin_zip = ''
        destination_zip = ''
        result_origin_city = ''
        result_origin_zip = ''
        result_origin_state = ''
        result_dest_city = ''
        result_dest_zip = ''
        result_dest_state = ''
        origin_x = ''
        origin_y = ''
        dest_x = ''
        dest_y = ''
        total_mileage = ''
    }

</script>
<main>
    <div class="flex flex-row h-screen">

        <div class="basis-1/2 flex items-center justify-center">

            <form action="" class="border border-black rounded flex flex-col h-1/3 w-1/2 p-8 bg-gray-200">
                <label for="origin">Origin Zip</label>
                <input id="origin" bind:value={origin_zip} type="text">
                <label for="dest">Destination Zip</label>
                <input id="dest" bind:value={destination_zip} type="text">
                <br>
                <button on:click={mileageLookup} type="button" class="text-gray-900 bg-gradient-to-r from-teal-200 to-lime-200 hover:bg-gradient-to-l hover:from-teal-200 hover:to-lime-200 focus:ring-4 focus:outline-none focus:ring-lime-200 dark:focus:ring-teal-700 font-medium rounded-lg text-sm px-5 py-2.5 text-center me-2 mb-2">Lookup Mileage</button>
            </form>
        </div>
        <div class="bg-green-100 basis-1/2 flex items-center justify-center">
            <div class="flex flex-col border border-black h-4/5 w-1/2 space-y-16">
                <h1>Origin City: {result_origin_city}</h1>
                <h1>Origin State: {result_origin_state}</h1>
                <h1>Origin Zip: {result_origin_zip}</h1>
                <h1>Destination City: {result_dest_city}</h1>
                <h1>Destination State: {result_dest_state}</h1>
                <h1>Destination Zip: {result_dest_zip}</h1>
                <h1>Total Mileage: {total_mileage} </h1>
                <button on:click={clearResults} type="button" class="text-white bg-gradient-to-br from-pink-500 to-orange-400 hover:bg-gradient-to-bl focus:ring-4 focus:outline-none focus:ring-pink-200 dark:focus:ring-pink-800 font-medium rounded-lg text-sm px-5 py-2.5 text-center me-2 mb-2 w-3/4">Clear Results</button>

            </div>
        </div>
    </div>
</main>