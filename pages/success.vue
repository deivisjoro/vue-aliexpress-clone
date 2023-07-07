<script setup>
    import MainLayout from '~/layouts/MainLayout.vue';
    import { useUserStore } from '~/stores/user';

    const userStore = useUserStore();
    userStore.isLoading = true;

    onMounted(()=>{
        setTimeout(()=>{
            userStore.isLoading = false;
        }, 200)
    })
</script>

<template>
    
    <MainLayout>
        <div id="SuccessPage" class="mt-4 max-w-[1200px] mx-auto px-2 min-h-[50vh]">
            <div class="bg-white w-full p-6 min-h-[150px]">
                <div class="flex items-center text-xl">
                    <Icon name="clarity:success-standard-line" color="#5fcb04" size="35" /> 
                    <span class="pl-4">Pago exitoso</span>
                </div>
                <p class="text-sm pl-[50px]">
                    Gracias. Hemos recibido su pago.
                </p>
            </div>
        </div>
    </MainLayout>

</template>