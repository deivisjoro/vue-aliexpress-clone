<script setup>

    import MainLayout from '~/layouts/MainLayout.vue';
    import { useUserStore } from '~/stores/user';
    
    const userStore = useUserStore();
    const user = useSupabaseUser();

    const cards = ref([
        'visa.png',
        'mastercard.png',
        'paypal.png',
        'applepay.png',
    ])

    const totalPriceComputed = computed(()=>{
        let price = 0;
        userStore.cart.forEach(prod=>{
            price += prod.price;
        })

        return price / 100;
    })

    const fnSelectedRadio = (e) => {
        if(!selectedArray.value.length){
            selectedArray.value.push(e);
            return;
        }

        selectedArray.value.forEach((item, index) => {
            if(e.id != item.id){
                selectedArray.value.push(e);
            }
            else{
                selectedArray.value.splice(index, 1);
            }
        })
    }

    const goToCheckout = () => {
        let ids = [];
        userStore.checkout = [];
        selectedArray.value.forEach(item=>ids.push(item.id));
        let res = userStore.cart.filter((item)=>{
            return ids.indexOf(item.id) != -1;
        })

        res.forEach(item => userStore.checkout.push(toRaw(item)));

        return navigateTo('checkout');
    }

    let selectedArray = ref([]);

    onMounted(()=>{
        setTimeout(()=>{
            userStore.isLoading = false;
        }, 200)
    })

</script>

<template>

    <MainLayout>

        <div id="ShoppingCartPage" class="mt-4 max-w-[1200px] mx-auto px-2">
            <div v-if="!userStore.cart.length" class="flex items-center justify-center">
                <div class="pt-10">
                    <img src="/cart-empty.png" alt="carrito vacio" class="mx-auto" width="250">
                    <div class="text-xl text-center mt-4">
                        Todavia no hay articulos
                    </div>
                    <div v-if="!user" class="flex text-center">
                        <NuxtLink to="/auth" class="bg-[#fd374f] w-full text-white text-[21px] font-semibold p-1.5 rounded-full mt-4 hover:bg-[#ff074f]">
                            Inicia Sesion
                        </NuxtLink>
                    </div>
                </div>
            </div>
            <div v-else class="md:flex gap-4 justify-between mx-auto w-full">
                <div class="md:w-[65%]">
                    <div class="bg-white rounded-lg p-4">
                        <div class="text-2xl font-bold mb-2">
                            Carrito de compras ({{ userStore.cart.length }})
                        </div>
                    </div>
                    <div class="bg-[#feeeef] rounded-lg p-4 mt-4">
                        <div class="text-red-500 font-bold">
                            Oferta de bienvenida aplicable solo a 1 artículo
                        </div>
                    </div>
                    <div id="Items" class="bg-white rounded-lg p-4 mt-4">
                        <div v-for="product in userStore.cart" :key="product">
                            <CardItem :product="product" :selectedArray="selectedArray" @onSelectedRadio="fnSelectedRadio" />
                        </div>
                    </div>
                </div>
                <div class="md:hidden block my-4"></div>
                <div class="md:w-[35%]">
                    <div id="Summary" class="bg-white rounded-lg p-4">
                        <div class="text-2xl font-extrabold mb-2">
                            Resumen
                        </div>
                        <div class="flex items-center justify-between my-4">
                            <div class="font-semibold">Total: </div>
                            <div class="text-2xl font-semibold">
                                $ <span class="font-extrabold">{{ totalPriceComputed }}</span>
                            </div>
                        </div>
                        <button class="flex items-center justify-center bg-[#fd374f] w-full text-white text-[21px] font-semibold p-1.5 rounded-full mt-4 hover:bg-red-600" @click="goToCheckout">
                            Confirmar
                        </button>
                    </div>
                    <div id="PaymentProtection" class="bg-white rounded-lg p-4 mt-4">
                        <div class="text-lg font-semibold mb-2">
                            Metodos de pago
                        </div>
                        <div class="flex items-center justify-start gap-8 my-4">
                            <div v-for="card in cards" :key="card">
                                <img :src="card" alt="imagen tarjeta" class="h-6">
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

    </MainLayout>
    
</template>