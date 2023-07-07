<script setup>

    import MainLayout from '~/layouts/MainLayout.vue';
    import { useUserStore } from '~/stores/user';


    const userStore = useUserStore();
    userStore.isLoading = true;

    const user = useSupabaseUser();

    const contactName = ref(null);
    const address = ref(null);
    const zipCode = ref(null);
    const city = ref(null);
    const country = ref(null);

    const currentAddress = ref(null);
    const isUpdate = ref(false);
    const isWorking = ref(false);
    const error = ref(null);

    watchEffect(async ()=>{
        currentAddress.value = await useFetch(`/api/prisma/get-address-by-user/${user.value.id}`);
        if(currentAddress.value.data){
            contactName.value = currentAddress.value.data.name;
            address.value = currentAddress.value.data.address;
            zipCode.value = currentAddress.value.data.zipcode;
            city.value = currentAddress.value.data.city;
            country.value = currentAddress.value.data.country;

            isUpdate.value = true;
        }
        userStore.isLoading = false;
        
    })

    onMounted(()=>{
        //userStore.isLoading = false;    
    })

    const fnSubmit = async () => {
        isWorking.value = true;
        error.value = null;

        if(!contactName.value){
            error.value = {
                type: 'contactName',
                message: 'El nombre de contacto es requerido'
            }
        }
        else if(!address.value){
            error.value = {
                type: 'address',
                message: 'La direccion es requerida'
            }
        }
        else if(!zipCode.value){
            error.value = {
                type: 'zipCode',
                message: 'El codigo postal es requerido'
            }
        }
        else if(!city.value){
            error.value = {
                type: 'city',
                message: 'La ciudad es requerida'
            }
        }
        else if(!country.value){
            error.value = {
                type: 'country',
                message: 'El pais es requerido'
            }
        }

        if(error.value){
            isWorking.value = false;
            return; 
        }

        const body = {
            userId: user.value.id,
            name: contactName.value,
            address: address.value,
            zipCode: zipCode.value,
            city: city.value,
            country: country.value
        }

        let method = 'POST';
        let url = '/api/prisma/add-address';
        if(isUpdate.value){
            method = 'PATCH';
            url = `/api/prisma/update-address/${currentAddress.value.data.id}`;           
        }

        await useFetch(url, {
            method,
            body
        })
        isWorking.value = false;
        return navigateTo('/checkout');
        
    }
</script>

<template>

    <MainLayout>

        <div id="AddressPage" class="mt-4 max-w-[500px] mx-auto px-2">
            <div class="mx-auto bg-white rounded-lg p-3">
                <div class="text-xl text-bold mb-2">
                    Detalles de la direccion
                </div>
                <form @submit.prevent="fnSubmit">
                    <TextInput class="w-full" placeholder="Nombre de contacto" v-model:input="contactName" inputType="text" :error="error && error.type==='contactName' ? error.message : ''" />

                    <TextInput class="w-full mt-2" placeholder="Direccion" v-model:input="address" inputType="text" :error="error && error.type==='address' ? error.message : ''" />

                    <TextInput class="w-full mt-2" placeholder="Codigo postal" v-model:input="zipCode" inputType="text" :error="error && error.type==='zipCode' ? error.message : ''" />

                    <TextInput class="w-full mt-2" placeholder="Ciudad" v-model:input="city" inputType="text" :error="error && error.type==='city' ? error.message : ''" />

                    <TextInput class="w-full mt-2" placeholder="Pais" v-model:input="country" inputType="text" :error="error && error.type==='country' ? error.message : ''" />

                    <button 
                        :disabled="isWorking" 
                        type="submit" 
                        class="mt-6 bg-gradient-to-r from-[#fe630c] to-[#ff3200] w-full text-white text-[21px] font-semibold p-1.5 rounded-full"
                    >
                        <div v-if="!isWorking">Actualizar direccion</div>
                        <Icon v-else name="eos-icons:loading" size="25" class="mr-2" />
                    </button>
                </form>
            </div>
        </div>

    </MainLayout>
    
</template>