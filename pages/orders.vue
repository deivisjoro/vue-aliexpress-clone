<script setup>

    import MainLayout from '~/layouts/MainLayout.vue';
    import { useUserStore } from '~/stores/user';

    const userStore = useUserStore();
    userStore.isLoading = true;

    const user = useSupabaseUser();

    const orders = ref(null);

    onBeforeMount(async ()=>{
        if(user.value){
            orders.value = await useFetch(`/api/prisma/get-all-orders-by-user/${user.value.id}`)
        }        
    })

    onMounted(()=>{
        setTimeout(()=>{
            userStore.isLoading = false;
        }, 200)

        if(!user.value){
            return navigateTo('/auth');
        }
    })

    watch(()=>user.value, ()=>{
        if(!user.value){
            return navigateTo('/auth');
        }
    })

</script>

<template>

    <MainLayout>
        <div id="OrdersPage" class="mt-4 max-w-[1200px] w-full px-2 min-h-[50vh]">
            <div class="bg-white w-full p-6 min-h-[150px]">
                <div class="flex items-center text-xl">
                    <Icon name="carbon:delivery" color="#5fcb04" size="35" /> 
                    <span class="pl-4">Pedidos</span>
                </div>
                <template v-if="orders && orders.data">
                    <div class="text-sm pl-[50px]" v-for="order in orders.data" :key="order">
                        <div class="border-b py-1 pt-3">
                            <p><span class="font-semibold">Stripe ID:</span> {{ order.stripeId }}</p>
                            <div class="pt-2"></div>
                            <div v-for="item in order.orderItem" :key="item">
                                <NuxtLink :to="`/item/${item.productId}`" class="flex items-center gap-3 p-1 hover:underline hover:text-blue-500">
                                    <img width="40" :src="item.product.url" alt="imagen producto"> 
                                    {{ item.product.title }}
                                </NuxtLink>
                            </div>
                            <div class="pt-2 pb-5">
                                <span class="font-semibold">Direccion de entrega:</span> {{ order.name }}, {{ order.address }}, {{ order.zipcode }}, {{ order.city }}, {{ order.country }} 
                            </div>
                        </div>
                    </div>                    
                </template>
                <div class="flex items-center justify-center" v-else>
                    No tienes historial de pedidos
                </div>
            </div>
        </div>
    </MainLayout>
    
</template>