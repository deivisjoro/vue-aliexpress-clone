<script setup>

    import MainLayout from '~/layouts/MainLayout.vue';
    import { useUserStore } from '~/stores/user';
    
    const userStore = useUserStore();
    const user = useSupabaseUser();
    const route = useRoute();
    userStore.isLoading = true;

    //definePageMeta({ middleware: "auth" })

    let stripe = null;
    let elements = null;
    let card = null;
    let form = null;
    let total = ref(0);
    let clientSecret = null;
    let currentAddress = ref(null);
    let isProcesing = ref(false);

    onBeforeMount(async ()=>{
        if(userStore.checkout.length<1){
            return navigateTo('shoppingcart');
        }

        total.value = 0;
        if(user.value){
            currentAddress.value = await useFetch(`/api/prisma/get-address-by-user/${user.value.id}`);
        }
    })

    watchEffect(()=>{
        if(route.fullPath == '/checkout' && !user.value){
            return navigateTo('/auth');
        }
    })

    onMounted(()=>{
        isProcesing.value = true;
        
        userStore.checkout.forEach(item => {
            total.value += item.price;
        })

        setTimeout(()=>{
            
            userStore.isLoading = false;
        }, 200)
    })

    watch(()=>total.value, ()=>{
        if(total.value>0){
            stripeInit();
        }
    })

    const stripeInit = async () => {
        const runtimeConfig = useRuntimeConfig();

        stripe = Stripe(runtimeConfig.public.stripePk);
        
        let res = await $fetch('/api/stripe/paymentintent', {
            method: 'POST',
            body: {
                amount: total.value
            }
        })

        clientSecret = res.client_secret;
        elements = stripe.elements();

        let style = {
            base: {
                fontSize: "18px"
            },
            invalid: {
                fontFamily: "Arial, sans-serif",
                color: "#ee4b2b",
                iconColor: "#ee4b2b"
            }
        }

        card = elements.create("card", {
            hidePostalCode: true,
            style
        })

        card.mount("#card-element");
        card.on('change', (e)=>{
            document.querySelector('button').disabled = e.empty;
            document.querySelector('#card-error').textContent = e.error ? e.error : '';
        })

        isProcesing.value = false;
    }

    const pay = async () => {
        if(currentAddress.value && currentAddress.value.data==''){
            showError('Agregue una direccion de envio');
            return;
        }
        isProcesing.value = true;

        let result = await stripe.confirmCardPayment(clientSecret, {
            payment_method: {
                card
            }
        })

        if(result.error){
            showError(result.error.message);
            isProcesing.value = false;
        }
        else{
            await createOrder(result.paymentIntent.id);
            userStore.cart = [];
            userStore.checkout = [];
            setTimeout(()=>{
                navigateTo('/success');
            }, 500)
        }
    }

    const createOrder = async (stripeId) => {
        await useFetch('/api/prisma/create-order', {
            method: 'POST',
            body: {
                userId: user.value.id,
                stripeId,
                name: currentAddress.value.data.name,
                address: currentAddress.value.data.address,
                zipcode: currentAddress.value.data.zipcode,
                city: currentAddress.value.data.city,
                country: currentAddress.value.data.country,
                products: userStore.checkout
            }
        })
    }

    const showError = (message)=>{
        let errorMsg = document.querySelector("#card-error");
        errorMsg.textContent = message;

        setTimeout(()=>{
            errorMsg.textContent = '';
        }, 4000)

    }
</script>

<template>
    <MainLayout>

        <div id="CheckoutPage" class="mt-4 max-w-[1200px] mx-auto px-2">
            <div class="md:flex gap-4 justify-between mx-auto w-full">
                <div class="md:w-[65%]">
                    <div class="bg-white rounded-lg p-4">
                        <div class="text-xl font-semibold mb-2">
                            Direccion de envio
                        </div>
                        <div v-if="currentAddress && currentAddress.data">
                            <NuxtLink to="/address" class="flex items-center pb-2 text-blue-500 hover:text-red-400">
                                <Icon name="mdi:plus" size="18" class="mr-2" /> 
                                Actualizar direccion
                            </NuxtLink>

                            <div class="pt-2 border-t">
                                <div class="underline pb-1">
                                    Direccion de entrega
                                </div>
                                <ul class="text-xs">
                                    <li class="flex items-center gap-2">
                                        <div>Nombre de contacto:</div>
                                        <div class="font-bold">{{ currentAddress.data.name }}</div>
                                    </li>
                                    <li class="flex items-center gap-2">
                                        <div>Direccion:</div>
                                        <div class="font-bold">{{ currentAddress.data.address }}</div>
                                    </li>
                                    <li class="flex items-center gap-2">
                                        <div>Codigo postal:</div>
                                        <div class="font-bold">{{ currentAddress.data.zipcode }}</div>
                                    </li>
                                    <li class="flex items-center gap-2">
                                        <div>Ciudad:</div>
                                        <div class="font-bold">{{ currentAddress.data.city }}</div>
                                    </li>
                                    <li class="flex items-center gap-2">
                                        <div>Pais:</div>
                                        <div class="font-bold">{{ currentAddress.data.country }}</div>
                                    </li>
                                </ul>
                            </div>
                        </div>
                        <div v-else>
                            <NuxtLink to="/address" class="flex items-center text-blue-500 hover:text-red-400">
                                <Icon name="mdi:plus" size="18" class="mr-2" /> 
                                Agregar nueva direccion
                            </NuxtLink>
                            <div class="border-t mt-2"></div>
                        </div>
                        <div id="Items" class="bg-white rounded-lg p-4 mt-4">
                            <div v-for="product in userStore.checkout" :key="product">
                                <CheckoutItem :product="product" />
                            </div>
                        </div>
                    </div>
                                       
                </div>
                <div class="md:hidden block my-4"></div> 
                <div class="md:w-[35%]">
                    <div id="PlaceOrder" class="bg-white rounded-lg p-4">
                        <div class="text-2xl font-extrabold mb-2">Resumen</div>
                        <div class="flex items-center justify-between my-4">
                            <div>Total compra:</div>
                            <div>Free</div>
                        </div>
                        <div class="border-t"></div>
                        <div class="flex items-center justify-between my-4">
                            <div class="font-semibold">Total</div>
                            <div class="text-2xl font-semibold">$ {{ total / 100 }}</div>
                        </div>
                        <form @submit.prevent="pay">
                            <div class="border border-gray-500 p-2 rounded-sm" id="card-element">

                            </div>
                            <p id="card-error" class="text-red-700 text-center font-semibold" role="alert"></p>

                            <button class="mt-4 bg-gradient-to-r from-[#fe630c] to-[#ff3200] w-full text-white text-[21px] font-semibold p-1.5 rounded-full" :disabled="isProcesing" type="submit" :class="isProcesing ? 'opacity-70' : 'opacity-100'">
                                <Icon name="eos-icons:loading" v-if="isProcesing" />
                                <div v-else>
                                    Realizar pedido
                                </div>
                            </button>
                        </form>
                    </div>
                    <div class="bg-white rounded-lg p-4 mt-4">
                        <div class="text-lg font-semibold mb-2 mt-2">
                            AliExpress
                        </div>
                        <p class="my-2">
                            AliExpress mantiene su informacion y pago seguros
                        </p>
                    </div>
                </div>
            </div>
        </div>

    </MainLayout>
    
</template>