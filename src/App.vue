<script setup>
    import flagdict from './flagdict.js';

    const flags=flagdict();
    import { ref, onMounted } from 'vue';
    const list=ref(null);
    const text_input=ref('');
    const country=ref(null);
    const biengioi=ref('');
    var filter=ref('translation');``
    const BingMapsKey="Ai6L7qL1lWaAvtEmZe-NtqWKPQ8t6lYxyHLUmJTW_u2WiXNJSqv0-D3bkg4QH0FB";
    const mapboxkey="pk.eyJ1IjoicGhhbWJpbmhhbmN0YjIwMDQiLCJhIjoiY2xxOHVqbzJrMWhmZjJpcGt5OGUyYzRyYyJ9.nzjdgqPHD3Yi4d_Qr9-Ddg"
    let map=ref(null);

    async function getJSON(link){
        try {
            const response = await fetch(link);
            if (response.status !== 200) {
                alert("Error");
            } else {
                const x = await response.json();
                return x;
            }
        }
        catch (error) {
            throw new Error("Cannot connect to " + link);
            alert("Cannot connect to "+link);
        }
    }   

    async function getAllName(){
        const x=await getJSON("./countries-name.json");
        x.sort((a,b)=>a.name.common.localeCompare(b.name.common));
        list.value=x;
    }
    
    async function searchAPI(filter, input){
        const x = await getJSON("https://restcountries.com/v3.1/" + filter + "/" + input);
        x.sort((a,b)=>a.name.common.localeCompare(b.name.common));
        list.value = x || "Loading..."; // Update list.value directly
        return x;
    }
        
    async function updateOutput(name){
        const x=await getJSON("https://restcountries.com/v3.1/name/"+name+"?fullText=true");
        country.value=x[0];
        await getBorders(x[0].borders);
        console.log(biengioi.value);
        const y=await getJSON(`https://dev.virtualearth.net/REST/v1/Locations?countryRegion=${name}&maxResults=1&key=${BingMapsKey}`);
        let bbox=y.resourceSets[0].resources[0].bbox;
        let marker=country.value.capitalInfo.latlng;
        let link=`https://www.openstreetmap.org/export/embed.html?bbox=${bbox[1]},${bbox[0]},${bbox[3]},${bbox[2]}&layer=mapnik&marker=${marker[0]},${marker[1]}`;
        map.value=`<iframe style="height : 340px; width : 510px" src="${link}"></iframe><br><a href="${link}" target="_blank">View map in new tab</a>`;
    }

    async function getBorders(borders){
        let a=[],x;
        for (var code of borders){
            x=await getJSON("https://restcountries.com/v3.1/alpha/"+code);
            a.push(x[0].name.common);
        }
        biengioi.value=a.join(', ');
    }
    onMounted(() => {
        getAllName();
    });
</script>

<style>
img {
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 5px;
  height: 160px;
}
</style>

<template>
    
<div id="heading&input">
    <h1 style=
            "background-color:blue;
            color:white; 
            font-weight:bold;
            border-radius:16px/16px;
            text-align: center;
            width: 450px">
        Ｃ🌎ＵＮＴＲＩＥＳ ＩＮＦ🌏  
    </h1>
    <input style="width: 380px" v-model="text_input" placeholder="Type here to search" @keydown.enter="searchAPI(filter, text_input)">
    <button @click="searchAPI(filter, text_input)" style="padding: 1px 10px;">Search</button>
    <br>
    <label for='filter'> Search by </label>
    <select v-model="filter">
        <option value="translation">name</option>
        <option>capital</option>
        <option value="lang">language</option>
        <option>currency</option>
        <option>region</option>
        <option value="subregion">sub region</option>
    </select>
</div>
<br>
<div>
    <ul v-if="list">
        <a v-for="object of list" @click="updateOutput(object.name.common)"><br>{{ flags[object.name.common]+object.name.common }}</a>
    </ul>
</div>

    <div id="output" v-if="country!=null">
        <img v-bind:src="country.flags.png" alt=country.flag.alt>
        <img v-bind:src="country.coatOfArms.png" style="width=400 height=400" alt="Coat of arms">
      <div class="country-details">
        <h1>{{country.name.common}}</h1>
        <p>
            <span>This is </span><span v-if="!country.independent">not </span><span>an independent country and </span>
            <span v-if="!country.unMember">not </span><span>a member of the United Nations. It </span>
            <span v-if="!country.landlocked">has </span><span v-else>doesn't have </span><span>access to open sea. It shares borders with </span>
            <span v-if="!country.borders">no countries or territories</span>
            <span v-else>{{ biengioi }}</span>
        </p>
        <div class="listDiv">
          <ul>
            <li><span>Official name: </span>{{ country.name.official }}</li>
            <li>Name in native/official languages:
                <ul>
                <li v-for="nativename, language in country.name.nativeName">
                    {{country.languages[language]+" : "+nativename.official+" (" + nativename.common + ") "}}
                </li>
                </ul>
            </li>
            <li><span>Demonym:</span> {{country.demonyms.eng.m }}</li>
            <li><span>Population:</span> {{country.population | rformatNumbers }}</li>
            <li><span>Region:</span> {{country.region}}</li>
            <li><span>Sub Region:</span> {{country.subregion}}</li>
            <li><span>Capital: </span><span v-html='country.capital.join(", ")'></span></li>
            <li><span>Currencies:</span> {{Object.values(country.currencies)[0].name+" ("+Object.values(country.currencies)[0].symbol+")"}}</li>

            <li><span>Time zones: </span><span v-for="i in country.timezones">{{ i+', ' }} </span> </li>
            <li><span>Side of driving: </span>{{ country.car.side }}</li>
            <li><span>Dial-in code: </span>{{ country.idd.root+country.idd.suffixes[0] }}</li>
            <li><span>Top Level Domain:</span> <span>{{ country.tld.join(", ") }}</span></li>

            
          </ul>
        </div>
        <div v-html="map"></div>
      </div>
    </div>
    
</template>
