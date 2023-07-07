<script setup>
    import { useUserStore } from '~/stores/user';

    const user = useSupabaseUser();
    const client = useSupabaseClient();

    const userStore = useUserStore();

    const isAccountMenu = ref(false);
    const searchItem = ref('');
    const isSearching = ref(false);
    const isCartHover = ref(true);

    const items = ref(null);

    const searchByName = useDebounce(async ()=>{
        isSearching.value = true;
        items.value = await useFetch(`/api/prisma/search-by-name/${searchItem.value}`);
        console.log(items.value)
        isSearching.value = false;
    }, 200);

    watch(()=>searchItem.value, async ()=>{
        if(!searchItem.value){
            setTimeout(()=>{
                items.value = '';
                isSearching.value = false;
                return;
            }, 500)
        }
        else{
            searchByName();
        }
    })
</script>

<template>
    <div id="MainLayout" class="w-full fixed z-50">
        <div id="TopMenu" class="w-full bg-[#fafafa] border-b md:block hidden">
            <ul class="flex items-center justify-end text-xs text-[#333333] font-light px-2 h-10 bg-[#fafafa] max-w-[1200px]">
                <li class="border-r border-r-gray-400 px-3 hover:text-[#ff4646] cursor-pointer">Vender en AliExpress</li>
                <li class="border-r border-r-gray-400 px-3 hover:text-[#ff4646] cursor-pointer">Preferencias de Cookies</li>
                <li class="border-r border-r-gray-400 px-3 hover:text-[#ff4646] cursor-pointer">Ayuda</li>
                <li class="border-r border-r-gray-400 px-3 hover:text-[#ff4646] cursor-pointer">Proteccion al Comprador</li>
                <li class="px-3 hover:text-[#ff4646] cursor-pointer">
                    <Icon name="ic:sharp-install-mobile" size="17" />
                    App
                </li>
                <li
                    @mouseenter="isAccountMenu = true" 
                    @mouseleave="isAccountMenu = false"
                    class="relative flex items-center px-2.5 hover:text-[#ff4646] cursor-pointer h-full" 
                    :class="isAccountMenu ? 'bg-white border z-40 shadow-[0_15px_100px_40px_rgba(0,0,0,0.3)]' : 'border border-[#fafafa]'">
                    <Icon name="ph:user-thin" size="17" />
                    Cuenta
                    <Icon name="mdi:chevron-down" size="15" class="ml-5" />
                    <div id="AccountMenu" v-if="isAccountMenu" class="absolute bg-white w-[220px] text-[#333333] z-40 top-[38px] -left-[100px] border-x border-b">
                        <div v-if="!user">
                            <div class="text-semibold text-[15px] my-4 px-3">
                                Bienvenido a AliExpress!
                            </div>
                            <div class="flex items-center gap-1 px-3 mb-3">
                                <NuxtLink to="/auth" class="bg-[#ff4646] text-center w-full text-[16px] rounded-sm text-white font-semibold p-2">
                                    Entrar / Registrarse
                                </NuxtLink>
                            </div>
                        </div>
                        <div class="border-b"></div>
                        <ul class="bg-white">
                            <li v-if="user" @click="navigateTo('/orders')" class="text-[13px] py-2 px-4 w-full hover:bg-gray-200">
                                Mis Ordenes
                            </li>
                            <li v-if="user" @click="client.auth.signOut()" class="text-[13px] py-2 px-4 w-full hover:bg-gray-200">
                                Cerrar Sesion
                            </li>
                        </ul>
                    </div>
                </li>
            </ul>
        </div>
        <div id="MainHeader" class="flex items-center w-full bg-white">
            <div class="flex lg:justify-start justify-between gap-10 max-w-[1150px] w-full px-3 py-5 mx-auto items-center">
                <NuxtLink to="/" class="min-w-[170px]">
                    <img src="/aliexpress-logo.png" alt="logo aliexpress" width="170" />
                </NuxtLink>
                <div class="max-w-[700px] w-full md:block hidden">
                    <div class="relative">
                        <div class="flex items-center border-2 border-[#ff4646] rounded-md w-full">
                            <input type="text" class="w-full placeholder-gray-400 text-sm pl-3 focus:outline-none" placeholder="accesorios de portatiles" v-model="searchItem">
                            <Icon name="eos-icons:loading" size="25" class="mr-2" v-if="isSearching" />
                            <button class="flex items-center  h-[100%] p-1.5 px-2 bg-[#ff4646]">
                                <Icon name="ph:magnifying-glass" size="20" color="#ffffff" />
                            </button>
                        </div>
                        <div class="absolute bg-white max-w-[700px] h-auto w-full">
                            <div class="p-1" v-if="items && items.data">
                                <NuxtLink :to="`/item/${item.id}`" class="flex items-center justify-between w-full cursor-pointer hover:bg-gray-100" v-for="item in items.data" :key="item">
                                    <div class="flex items-center">
                                        <img class="rounded-md" width="40" :src="item.url" alt="imagen item">
                                        <div class="truncate ml-2">{{item.title}}</div>
                                    </div>
                                    <div class="truncate">$ {{item.price/100}}</div>

                                </NuxtLink>
                            </div>
                        </div>
                    </div>
                </div>
                <NuxtLink to="/shoppingcart" class="flex items-center">
                    <button class="relative md:block hidden" @mouseenter="isCartHover=true" @mouseleave="isCartHover=false">
                        <span class="absolute flex items-center justify-center -right-[3px] top-0 bg-[#ff4646] h-[17px] min-w-[17px] text-xs px-0.5 rounded-full text-white">
                            {{ userStore.cart.length }}
                        </span>
                        <div class="min-w-[40px]">
                            <Icon name="ph:shopping-cart-simple-light" size="33" :color="isCartHover ? '#ff4646' : ''" />
                        </div>
                    </button>
                </NuxtLink>
                <button @click="userStore.isMenuOverlay=true" class="md:hidden block rounded-full p-1.5 -mt-[4px] hover:bg-gray-200">
                    <Icon name="radix-icons:hamburger-menu" size="33" />
                </button>
            </div>
        </div>
    </div>

    <Loading v-if="userStore.isLoading"/>

    <div class="lg:pt-[150px] md:pt-[130px] pt-[80px]"></div>
    <slot />

    <Footer v-if="!userStore.isLoading"></Footer>
</template>