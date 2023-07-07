<script setup>

    import { useUserStore } from '~/stores/user';

    const userStore = useUserStore();

    const props = defineProps(['product', 'selectedArray'])
    const { product, selectedArray } = toRefs(props);
    
    const emit = defineEmits(['onSelectedRadio']);

    const isHover = ref(false);
    const isSelected = ref(false);

    const removeFromCart = () => {
        userStore.cart.forEach((prod, index) => {
            if(prod.id===product.value.id){
                userStore.cart.splice(index, 1);
            }
        })
    }

    watch(()=>isSelected.value, (val)=>{
        emit('onSelectedRadio', {id: product.value.id, val})
    })
</script>

<template>

    <div class="flex justify-start my-2">
        <div class="my-auto">
            <div class="flex items-center justify-start p-0.5">
                <div class="flex items-center justify-center h-[20px] w-[20px] rounded-full border mr-5 ml-2 cursor-pointer" :class="[isHover ? 'border-[#fd374f]' : 'border-gray-300', isSelected ? 'bg-[#fd374f]' : '']" @click="isSelected=!isSelected" @mouseenter="isHover=true" @mouseleave="isHover=false">
                    <div class="h-[8px] w-[8px] rounded-full bg-white"></div>

                </div>

            </div>
        </div>
        <img :src="product.url" :alt="product.url" class="rounded-md md:w-[150px] w-[90px]">
        <div class="overflow-hidden pl-2 w-full">
            <div class="flex items-center justify-between w-full">
                <div class="flex items-center justify-between truncate">
                    <span class="sm:block hidden bg-[#fd374f] text-white text-[9px] font-semibold px-1.5 rounded-sm  min-w-[80px]">
                        Oferta de bienvenida
                    </span>
                    <div class="truncate sm:pl-2">
                        {{ product.title }}
                    </div>
                </div>
                <button class="mx-3 sm:block hidden -mt-0.5 hover:text-red-500" @click="removeFromCart">
                    <Icon name="material-symbols:delete-outline" size="20" />
                </button>
            </div>
            <div class="text-xl font-semibold">
                $ <span class="font-bold">{{ product.price / 100 }}</span>
            </div>

            <p class="text-[#009a66] text-xs font-semibold pt-1">
                Entrega gratuita en 11 dias a partir de $ 8.20
            </p>
            <p class="text-[#009a66] text-xs font-semibold pt-1">
                Envio gratis
            </p>
            <div class="flex items-center justify-end">
                <button class="sm:hidden block -mt-0.5 hover:text-red-500" @click="removeFromCart">
                    <Icon name="material-symbols:delete-outline" size="20" />
                </button>
            </div>
        </div>
    </div>
    
</template>