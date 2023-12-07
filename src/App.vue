<script setup>
    import { ref } from 'vue'
    const name_input=ref('');
    var output=ref('');
    async function get_object(input){
        var response, x;
        await fetch("https://restcountries.com/v3.1/name/"+input).then(async response => {
            console.log(response.status);
            if (response.status != 200) {
                alert("Error");
            }
            else{
                await response.json().then(x => {
                    while (x===undefined){
                        output="Loading...";
                        console.log("Loading");
                    }
                    output=x;
                })
            }
        });
        console.log(output);
    }
</script>

<template>
    <h1>Countries Info</h1>
    <p></p>
    <input v-model="name_input" placeholder="Input name of country">
    <button @click="get_object(name_input)">Search</button>
    <p>{{ output }}</p>
</template>