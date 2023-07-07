<script setup>
    const props = defineProps({
        product: {
            type: Object
        }
    })

    const { product } = toRefs(props);

    const priceComputed = computed(()=>{
        return product.value.price / 100;
    })

    const oldPriceComputed = computed(()=>{
        let res = (product.value.price + (product.value.price /20)) / 100;
        return res.toFixed(2);
    })    

</script>

<template>

    <div class="bg-white inline-block rounded hover:shadow-[0_0_10px_3px_rgba(0,0,0,0.15)] cursor-pointer" :id="`ProductComponent${product.id}`">
        <NuxtLink :to="`/item/${product.id}`">
            <img :src="product.url" :alt="`imagen producto ${product.title}`" class="rounded-t" />
            <div class="" id="ProductDetails">
                <span class="flex items-center justify-start gap-3 px-1 pt-1">
                    <span class="text-[#ff6674] font-semibold">${{ priceComputed }}</span>
                    <span class="text-gray-500 text-sm text-light line-through">${{ oldPriceComputed }}</span>
                </span>
                <span class="px-1 relative -top-1.5 text-[#ff6674] text-xs font-semibold">
                    Extra 5% descuento
                </span>
                <div class="flex items-center gap-1 px-1 relative -top-1">
                    <span class="bg-[#fd374f] text-white text-[9px] font-semibold px-1.5 rounded-sm">Dto. Bienvenida</span>
                    <span class="bg-[#f5f5f5] border text-[#c08562] text-[9px] font-semibold px-1.5 rounded-sm">Mas vendido</span>
                </div>
                <p class="flex items-center px-1 pt-0.5 text-xs text-[#252525]">
                    5,000+ ventas <Icon name="material-symbols:star-rate" color="#7575757" class="ml-1.5" /> 4.7
                </p>
                <p class="px-1 pt-0.5 text-xs text-[#252525]">
                    {{ product.title.substring(0,60) }}
                </p>
                <p class="px-1 pb-1">
                    <span class="text-[#009a66] text-xs font-semibold">Envio gratis</span>
                </p>
            </div>
        </NuxtLink>
    </div>
    
</template>