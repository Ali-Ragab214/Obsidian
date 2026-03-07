

this is parent file 


<script setup>

import Footer  from './Footer.vue'; //we import the Footer component here because we will use it in this file

  

let products = [

  {

    id: 1,

    name: "Product 1",

    description: "This is the description for Product 1.",

    price: "$19.99",

    image: "https://via.placeholder.com/150"

  },

  {

    id: 2,

    name: "Product 2",

    description: "This is the description for Product 2.",

    price: "$29.99",

    image: "https://via.placeholder.com/150"

  },

  {

    id: 3,

    name: "Product 3",

    description: "This is the description for Product 3.",

    price: "$39.99",

    image: "https://via.placeholder.com/150"

  }

];

</script>

  

<template>

<Footer    :products="products" /> <!--here we said that we will send the products prop to the Footer component-->

<!--this an example for props-->

</template>






child file

<script setup>

//this is the Footer(child) component, we will receive the products prop from the NavBar component

defineProps({

  products: {

    type: Array,

    required: true

  }

});

</script>

  

<template>

    <div v-for="product in products" :key="product.id">

    <h3>{{ product.name }}</h3>

    <p>{{ product.description }}</p>

    <span>{{ product.price }}</span>

  </div>

</template>

# يعني إيه `:key` ؟

لما تكتب:

`<div v-for="product in products" :key="product.id">`

أنت بتقول لـ Vue:

> كل عنصر هنا ليه هوية مميزة (ID)